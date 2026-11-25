# Exercise 06 — Prerequisite Knowledge

## SSH Brute-Force Detection & Response

This document introduces the concepts behind SSH authentication, credential attacks, log-based detection, and automated defensive responses.

## 1. Secure Shell (SSH)

Secure Shell (SSH) is a protocol used to securely access and administer remote systems over a network. It provides encrypted communication and server authentication, and typically uses TCP port 22.

An SSH connection involves a client initiating a connection to a server, establishing an encrypted channel, verifying the server's identity, and authenticating the user. Encryption protects credentials in transit, but does not prevent an attacker from attempting to authenticate with guessed credentials.

## 2. Authentication

Authentication is the process of verifying an identity. Authorization determines what an authenticated identity is permitted to access.

| Authentication method | Description |
| --- | --- |
| Password | The user demonstrates knowledge of a secret password. |
| Public key | The client proves possession of a private key corresponding to an authorized public key. The private key is not transmitted to the server. |
| Multi-factor authentication (MFA) | Requires evidence from two or more distinct authentication factor categories. |

SSH host keys identify servers, whereas user authentication keys identify users. Verifying a server's host key helps protect against impersonation.

## 3. Credential Attacks

Credential attacks attempt to gain unauthorized access by exploiting authentication mechanisms.

| Attack | Description |
| --- | --- |
| Brute force | Repeatedly attempts possible credentials to find a valid combination. |
| Dictionary attack | Attempts passwords drawn from a list of likely candidates. |
| Password spraying | Attempts one or a few passwords against multiple accounts. |
| Credential stuffing | Attempts previously compromised username and password combinations against other services. |

Repeated authentication failures may indicate a credential attack, but can also result from legitimate users entering incorrect passwords or applications using outdated credentials.

## 4. Authentication Hardening

| Control | Purpose |
| --- | --- |
| Strong, unique passwords | Reduce susceptibility to guessing and credential reuse. |
| Public key authentication | Reduces exposure to remote password guessing when password authentication is disabled. |
| MFA | Adds an independent authentication requirement. |
| Rate limiting | Restricts the frequency of authentication attempts. |
| Account lockout | Temporarily restricts authentication to a targeted account after defined conditions are met. |
| Source IP blocking | Restricts connections originating from a particular IP address. |

Account lockouts and source IP bans are different controls. An account lockout affects an identity, whereas an IP ban affects traffic from a network address and may impact multiple legitimate users sharing that address.

## 5. Linux Authentication Logging

Linux services generate logs that provide information about system activity, errors, and security events. On systems using systemd, the systemd journal can collect logs from services such as OpenSSH.

An illustrative SSH authentication failure might appear as follows:

```text
Nov 25 15:51:09 ubuntu sshd-session[1539]: Failed password for ubuntu from 172.16.77.11 port 55428 ssh2
```

| Field | Interpretation |
| --- | --- |
| Nov 25 15:51:09 | Event timestamp |
| ubuntu (before process name) | Host that recorded the event |
| sshd-session[1539] | Process name and process ID |
| Failed password | Authentication outcome |
| ubuntu (after `for`) | Account used in the attempt |
| 172.16.77.11 | Source IP address |
| 55428 | Client source port, not the SSH server's listening port |
| ssh2 | SSH protocol version |

A failed authentication event does not establish that an account was compromised. Successful authentication events and other supporting evidence should also be examined during an investigation.

## 6. Detection Logic

Detection rules identify activity that meets predefined conditions. A threshold-based detection evaluates the number of matching events within a specified period.

For example, a rule might identify three failed SSH authentication attempts from the same source IP within 60 seconds.

| Component | Purpose |
| --- | --- |
| Event pattern | Identifies relevant log entries, such as failed passwords. |
| Grouping criterion | Associates related events, such as those sharing a source IP. |
| Threshold | Number of matching events required to trigger a response. |
| Time window | Period during which the matching events must occur. |
| Response | Action taken when the detection criteria are satisfied. |

A regular expression (regex) is a pattern used to match text. Log-based detection tools can use regular expressions to identify authentication failures and extract relevant fields such as source IP addresses.

## 7. Fail2ban

Fail2ban is a log-monitoring and automated response tool. It identifies matching events and can temporarily block source IP addresses using the host's firewall.

| Component | Function |
| --- | --- |
| Filter | Defines patterns used to identify relevant log events. |
| Jail | Associates a filter with a log source, detection settings, and actions. |
| Backend | Determines how Fail2ban obtains or monitors log events. |
| Action | Defines what happens when an IP is banned or unbanned. |
| `maxretry` | Number of matching failures required to trigger a ban. |
| `findtime` | Time window in which failures are counted. |
| `bantime` | Duration of the ban. |

Fail2ban does not replace SSH authentication or a firewall. It uses evidence from authentication logs to initiate a defensive action through an enforcement mechanism.

## 8. Automated Response and Firewall Enforcement

In this exercise, Fail2ban monitors SSH authentication events on Ubuntu and uses nftables to enforce temporary bans.

```text
Failed SSH authentication
          |
          v
Authentication log event
          |
          v
Fail2ban filter and jail
          |
          v
Detection threshold reached
          |
          v
Temporary firewall ban
          |
          v
Ban expires or is manually removed
```

A firewall can use an address set to maintain a collection of blocked IP addresses. A rule referencing that set can reject matching traffic without requiring a separate rule for every address.

A ban scoped to SSH traffic can prevent a source from establishing new SSH connections while leaving unrelated services, such as HTTP, accessible. The exact behavior depends on the configured firewall action.

Detection, enforcement, and recovery should be validated separately. A ban recorded in a log does not independently prove that the firewall successfully enforced it.

## 9. Detection Limitations

| Limitation | Description |
| --- | --- |
| False positive | Legitimate activity incorrectly identified as suspicious. |
| False negative | Malicious activity that does not trigger a detection. |
| Distributed attacks | Attempts spread across multiple sources may remain below per-IP thresholds. |
| Low-frequency attacks | Attempts spread over time may fall outside the detection window. |
| Shared IP addresses | Blocking one address may affect multiple legitimate users. |
| Missing or incomplete logs | Detection may fail when relevant events are unavailable or not matched by the filter. |

Detection thresholds should balance security effectiveness against operational impact. Automated blocking is one layer of defense, not a substitute for strong authentication and ongoing monitoring.

---

Return to the [Exercise 06 README](README.md) or continue to the [walkthrough](06-walkthrough.md).
