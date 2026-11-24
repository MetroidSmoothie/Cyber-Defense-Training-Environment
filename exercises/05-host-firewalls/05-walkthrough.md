# Exercise 5 — Host-Based Firewall Defense

This walkthrough demonstrates how a host-based firewall restricts communication between systems on the same internal subnet, independently of OPNsense.

Complete the [Lab Environment Setup](../../Lab%20Environment%20Setup.md) before beginning. Use the Ubuntu VirtualBox console for firewall administration to avoid losing remote access.

## Task 1 — Establish Baseline Connectivity

On Debian, verify the route to Ubuntu and test SSH and HTTP connectivity:

```bash
ip route get 172.16.77.10
nc -z -v -w 3 172.16.77.10 22
nc -z -v -w 3 172.16.77.10 80
```

The route uses `enp0s3` directly, without a gateway. Both TCP connections succeed, confirming that Debian can access Ubuntu's services without traversing OPNsense.

![Debian baseline connectivity](../../images/05-host-based-firewall-defense/debian-host-firewall-baseline.png)

## Task 2 — Inspect the Host Firewall

On Ubuntu, run:

```bash
sudo ufw status verbose
sudo ufw show added
```

- `ufw` — Uncomplicated Firewall, a utility for managing host firewall rules.
- `status verbose` — Displays firewall status, policies and active rules when enabled.
- `show added` — Lists explicitly added UFW rules, including when UFW is inactive.

The baseline shows `Status: inactive` and no added rules. If your system has existing rules, preserve its configuration before proceeding.

![Initial UFW status](../../images/05-host-based-firewall-defense/ubuntu-ufw-initial-status.png)

## Task 3 — Implement Host-Based Firewall Protection

On Ubuntu, configure the default policies:

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

- `default deny incoming` — Denies incoming traffic unless permitted by firewall rules.
- `default allow outgoing` — Allows outbound traffic by default.

Create an HTTP exception restricted to Debian:

```bash
sudo ufw allow from 172.16.77.11 to 172.16.77.10 port 80 proto tcp
```

- `from` / `to` — Specify the source and destination addresses.
- `port 80` — Specifies the destination port.
- `proto tcp` — Restricts the rule to TCP.

Enable logging and activate UFW:

```bash
sudo ufw logging on
sudo ufw enable
sudo ufw status verbose
```

- `logging on` — Enables firewall logging.
- `enable` — Activates UFW and enables it at system startup.

If prompted about disrupting SSH connections, confirm using the local VM console.

Verify that UFW is active, incoming traffic is denied by default, outgoing traffic is allowed, logging is on, and the only explicitly added rule permits TCP 80 from Debian.

![Active UFW policy and HTTP exception](../../images/05-host-based-firewall-defense/ubuntu-ufw-policy-configuration.png)

## Task 4 — Validate Internal Access Restrictions

On Debian, repeat the TCP connectivity tests and verify Apache responds:

```bash
nc -z -v -w 3 172.16.77.10 22
nc -z -v -w 3 172.16.77.10 80
wget --spider -S -T 5 -t 1 http://172.16.77.10
```

The SSH connection times out, the HTTP connection succeeds, and Apache returns `HTTP/1.1 200 OK`.

![SSH blocked and HTTP permitted](../../images/05-host-based-firewall-defense/debian-host-firewall-validation.png)

## Task 5 — Verify Host Firewall Enforcement

On Debian, generate another SSH connection attempt:

```bash
nc -z -v -w 3 172.16.77.10 22
```

On Ubuntu, examine recent kernel logs:

```bash
sudo journalctl -k --since "5 minutes ago" --no-pager | grep 'UFW BLOCK'
```

- `journalctl -k` — Displays kernel log messages.
- `--since` — Restricts results to the specified time period.
- `grep 'UFW BLOCK'` — Filters output to UFW block events.

For a shorter output, display only the last three matching entries:

```bash
sudo journalctl -k --since "10 minutes ago" --no-pager | grep 'UFW BLOCK' | tail -n 3
```

- `tail -n 3` — Displays the last three lines of output.

Identify the following fields in a matching log entry:

| Field | Observed value |
|---|---|
| Action | `[UFW BLOCK]` |
| `IN` | `enp0s3` |
| `SRC` | `172.16.77.11` |
| `DST` | `172.16.77.10` |
| `PROTO` | `TCP` |
| `DPT` | `22` |
| TCP flag | `SYN` |

`DPT` identifies the destination port; `SPT` identifies the source port. The block entry confirms that Ubuntu's host firewall denied the incoming SSH connection attempt.

![UFW blocked SSH connection logs](../../images/05-host-based-firewall-defense/ubuntu-ufw-blocked-ssh-log.png)

## Task 6 — Validate Defense in Depth

On Ubuntu, inspect the configured rules:

```bash
sudo ufw status numbered
```

- `numbered` — Displays rules with their corresponding rule numbers.

Confirm that the HTTP exception is the only explicitly added allow rule.

On Debian, scan Ubuntu:

```bash
nmap -sT -p 22,80,443 172.16.77.10
```

Compare the results with the internal scan from Exercise 4 and the connectivity baseline from Task 1:

| Port | Before UFW | After UFW |
|---|---|---|
| 22/tcp (SSH) | Open | Filtered |
| 80/tcp (HTTP) | Open | Open |
| 443/tcp (HTTPS) | Closed | Filtered |

A filtered port does not reveal whether a service is listening. TCP 443 changes from closed to filtered because the firewall blocks the probes before the scanner receives a definitive response.

![Internal port scan with UFW enabled](../../images/05-host-based-firewall-defense/debian-host-firewall-port-comparison.png)

## Task 7 — Restore the Baseline

On Ubuntu, disable UFW and remove the exercise-specific configuration:

```bash
sudo ufw disable
sudo ufw reset
```

- `disable` — Stops UFW enforcement and disables it at system startup.
- `reset` — Removes UFW rules and restores its default configuration.

Confirm the reset when prompted. Reset is appropriate for this lab because UFW was initially inactive with no added rules. Do not reset an existing firewall configuration without preserving it.

Verify the restored state:

```bash
sudo ufw status verbose
sudo ufw show added
```

Confirm that UFW is inactive and no added rules remain.

On Debian, repeat the original connectivity tests:

```bash
nc -z -v -w 3 172.16.77.10 22
nc -z -v -w 3 172.16.77.10 80
```

Both connections should succeed, matching the Task 1 baseline.

![Restored Debian connectivity](../../images/05-host-based-firewall-defense/debian-host-firewall-baseline-restored.png)

## Final Validation

- [ ] UFW is inactive and no exercise-specific rules remain.
- [ ] Debian can access Ubuntu on TCP 22 and 80.
- [ ] OPNsense rules remain unchanged.
- [ ] The lab remains isolated on VirtualBox Internal Networks.
