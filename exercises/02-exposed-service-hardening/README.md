# Exercise 2 — Exposed Service Hardening

**Difficulty:** Beginner  
**Focus:** Service discovery, port scanning, listening addresses, and SSH socket hardening  
**Platform:** OPNsense 26.7, Kali Linux, Ubuntu Server, Debian, VirtualBox

## Overview

Identify services running on an internal Linux host, compare their accessibility from different network segments, and reduce unnecessary SSH listening exposure. Validate the change and restore the original configuration.

For commands, brief output explanations, verified results, and screenshots, see the [step-by-step walkthrough](02-walkthrough.md).

## Scenario

You are reviewing a segmented network's Ubuntu server. SSH and HTTP are available to internal systems, while an external workstation is separated by a firewall. Your task is to assess which services are reachable, restrict SSH to the server's internal IPv4 address, and verify that the service still works before restoring the baseline.

## Learning Objectives

- Identify listening TCP services and the addresses to which they bind.
- Interpret `open`, `closed`, and `filtered` port-scan results.
- Compare service accessibility from internal and external network segments.
- Recognize SSH socket activation and the difference between a service's status and its listening socket.
- Restrict a service to a specific local IP address using a systemd socket override.
- Validate the changed listening address without confusing service binding with firewall filtering.
- Restore the original socket configuration and verify the baseline.

## Prerequisites

Complete the [Lab Environment Setup](../../Lab%20Environment%20Setup.md) and restore the default WAN policy from Exercise 1.

| Component | Address | Role |
|---|---|---|
| Kali (`Lab-External`) | `192.168.50.10/24` | External scanning host |
| OPNsense WAN | `192.168.50.1/24` | External firewall interface |
| OPNsense LAN | `172.16.77.1/24` | Internal firewall interface |
| Ubuntu (`Lab-Internal`) | `172.16.77.10/24` | Apache and SSH target |
| Debian (`Lab-Internal`) | `172.16.77.11/24` | Internal scanning host |

**Before starting:** Confirm Apache and SSH are installed on Ubuntu, OPNsense is running, and Kali uses only its `Lab-External` adapter during external scans. Take an Ubuntu snapshot before modifying SSH configuration.

> **Isolation requirement:** Keep all VMs on VirtualBox Internal Networks. If Kali's temporary management adapter is enabled to administer OPNsense, disable it before external scans.

## Assessment Tasks

1. **Identify listening services.** On Ubuntu, inspect listening TCP sockets and the status of Apache and SSH.
2. **Scan from the external segment.** From Kali, scan Ubuntu's TCP ports 22, 80, and 443 and record their states.
3. **Compare internal exposure.** Scan the same ports from Debian and compare the results with Kali's scan.
4. **Restrict SSH listening.** Inspect `ssh.socket`, then create a reversible systemd override binding SSH only to `172.16.77.10:22`.
5. **Validate the change.** Compare Ubuntu's listening sockets before and after the change and confirm SSH remains accessible from Debian.
6. **Compare security controls.** Explain the difference between restricting a service's local listening address and filtering traffic at a network firewall.
7. **Restore the baseline.** Remove the SSH socket override and verify the original IPv4 and IPv6 listeners return.

## Success Criteria

| Check | Required result |
|---|---|
| Initial Ubuntu SSH and HTTP | Listening on TCP 22 and 80 |
| Kali → Ubuntu ports 22, 80, 443 | Filtered |
| Debian → Ubuntu ports 22, 80 | Open |
| Debian → Ubuntu port 443 | Closed |
| Hardened SSH listener | `172.16.77.10:22` only |
| Internal SSH accessibility | Still available from Debian |
| Restored SSH listeners | `0.0.0.0:22` and `[::]:22` |
| Cleanup | SSH override removed; Kali management adapter disabled |

Capture evidence of the baseline, configuration change, and restoration. The walkthrough includes suggested screenshot filenames.

## Knowledge Check

1. What is the difference between an `open`, `closed`, and `filtered` TCP port?
2. What is the difference between a service listening on `0.0.0.0` and one listening on a specific IPv4 address?
3. How does a loopback address differ from a network-facing interface address?
4. What is an attack surface, and how can unnecessary network services increase it?
5. What is the principle of least privilege, and how can it apply to network service exposure?
6. Why is it beneficial to use multiple layers of security controls rather than relying on a firewall alone?

## Walkthrough

Compare your results with the [Exercise 2 Walkthrough](02-walkthrough.md) for commands, brief output explanations, screenshots, and verified results.

## Cleanup and Safety

Remove the exercise-specific SSH socket override, confirm the original listeners are restored, and disable Kali's temporary management adapter. Keep the lab isolated from production, home, and public networks.
