# Exercise 5 — Host-Based Firewall Defense

**Difficulty:** Beginner–Intermediate  
**Focus:** Host-based firewalls, defense in depth, internal access control, and lateral movement prevention  
**Platform:** Ubuntu Server, Debian, UFW, VirtualBox

## Overview

Configure Ubuntu's host-based firewall to restrict access from another system on the same internal subnet. Preserve required HTTP access, block SSH connection attempts, and verify the restrictions through connectivity tests, port scanning, and firewall logs.

For commands, output explanations, and screenshots, see the [step-by-step walkthrough](05-walkthrough.md).

## Scenario

An internal workstation has been compromised and can communicate directly with other systems on the local network. Your task is to protect an Ubuntu web server by restricting unnecessary inbound access while preserving its required HTTP service, independently of the OPNsense perimeter firewall.

## Learning Objectives

- Identify traffic that does not traverse the network firewall.
- Inspect and configure a host-based firewall using UFW.
- Implement a default-deny inbound policy with a narrowly scoped service exception.
- Validate permitted and denied connections between hosts on the same subnet.
- Identify blocked connection attempts in host firewall logs.
- Explain how endpoint-level access controls help limit lateral movement.
- Restore the original host firewall configuration.

## Prerequisites

Complete the [Lab Environment Setup](../../Lab%20Environment%20Setup.md). Confirm that Ubuntu and Debian are operational, UFW is available on Ubuntu, and Netcat, Wget, and Nmap are available on Debian.

| Component | Address | Role |
|---|---|---|
| Ubuntu (`Lab-Internal`) | `172.16.77.10/24` | Protected HTTP and SSH server |
| Debian (`Lab-Internal`) | `172.16.77.11/24` | Internal testing host |
| OPNsense LAN | `172.16.77.1/24` | Internal firewall interface |

**Before starting:** Confirm that Apache and SSH are available on Ubuntu. Use the Ubuntu VirtualBox console for firewall administration to avoid losing remote access.

## Assessment Tasks

1. **Establish baseline connectivity.** Verify that Debian communicates directly with Ubuntu and can access TCP ports 22 and 80.
2. **Inspect the host firewall.** Identify UFW's initial status and any existing rules.
3. **Implement host-based firewall protection.** Configure a default-deny inbound policy, permit outbound traffic, allow HTTP from Debian, and enable firewall logging.
4. **Validate internal access restrictions.** Confirm SSH connections time out while HTTP remains accessible and Apache returns a successful response.
5. **Verify host firewall enforcement.** Identify UFW block events corresponding to Debian's SSH connection attempts.
6. **Validate defense in depth.** Scan Ubuntu from Debian and compare the resulting port states with the earlier internal reconnaissance results.
7. **Restore the baseline.** Disable UFW, remove the exercise-specific configuration, and verify that SSH and HTTP access is restored.

## Success Criteria

| Check | Required result |
|---|---|
| Initial Debian → Ubuntu connectivity | TCP 22 and 80 accessible via the internal network |
| Initial UFW configuration | Inactive; no added rules |
| Host firewall policy | Active; default deny incoming, allow outgoing; logging enabled |
| HTTP exception | TCP 80 allowed from Debian only by the explicit rule |
| SSH access | Connection times out |
| HTTP functionality | `HTTP/1.1 200 OK` |
| Host firewall logs | UFW block entries identify Debian's TCP 22 connection attempts |
| Internal port scan | TCP 22 and 443 filtered; TCP 80 open |
| Restored baseline | UFW inactive with no added rules; TCP 22 and 80 accessible again |

Capture evidence for each major validation step. The walkthrough includes observed results and suggested screenshot filenames.

## Knowledge Check

1. How does a host-based firewall differ from a network-based firewall?
2. What is lateral movement, and why might a compromised internal workstation attempt it?
3. Why might a perimeter firewall be unable to prevent communication between two hosts on the same subnet?
4. What is the difference between north-south and east-west network traffic?
5. Why should a system's presence on a trusted internal network not automatically grant it access to other systems?
6. How can host firewall logs help identify potential lateral movement, and why is a blocked connection alone insufficient to establish malicious intent?

## Walkthrough

Compare your results with the [Exercise 5 Walkthrough](05-walkthrough.md) for step-by-step instructions, command explanations, screenshots, and verified results.

## Cleanup and Safety

Restore UFW to its initial inactive state with no added rules. Only reset UFW if its original configuration was verified to contain no added rules, as in this lab. Confirm SSH and HTTP connectivity is restored, preserve the OPNsense configuration, and keep the lab isolated on VirtualBox Internal Networks.
