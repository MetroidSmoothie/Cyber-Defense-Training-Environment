# Exercise 6 — SSH Brute-Force Detection & Response

**Difficulty:** Intermediate  
**Focus:** Authentication security, log analysis, automated detection, containment, and recovery  
**Platform:** Ubuntu Server, Debian, OpenSSH, Fail2ban, nftables, VirtualBox

## Overview

Simulate repeated SSH authentication failures, investigate Linux authentication logs, and configure Fail2ban to automatically restrict access from an offending source IP. Verify firewall enforcement, recover legitimate access, and restore the lab baseline.

For commands, output explanations, and screenshots, see the [step-by-step walkthrough](06-walkthrough.md).

## Scenario

An internal host is repeatedly attempting to authenticate to an Ubuntu server over SSH. Your task is to identify the suspicious authentication activity and implement automated containment while preserving access to unrelated services.

## Learning Objectives

- Inspect SSH authentication settings and establish baseline connectivity.
- Identify repeated authentication failures and their source in Linux logs.
- Configure threshold-based detection and automated IP blocking.
- Validate the effectiveness and scope of an automated response.
- Inspect the host firewall rules enforcing an IP ban.
- Manually remove a ban and verify legitimate access is restored.
- Restore the original lab security posture.

## Prerequisites

Complete the [Lab Environment Setup](../../Lab%20Environment%20Setup.md). Confirm OpenSSH, Apache, Fail2ban, and nftables are available on Ubuntu, and the SSH client and Netcat are available on Debian.

| Component | Address | Role |
|---|---|---|
| Ubuntu (`Lab-Internal`) | `172.16.77.10/24` | Protected SSH and HTTP server |
| Debian (`Lab-Internal`) | `172.16.77.11/24` | Authentication testing host |
| OPNsense LAN | `172.16.77.1/24` | Internal firewall interface |

**Before starting:** Confirm password authentication is enabled for the lab account, Fail2ban is stopped, and UFW is inactive. Use the Ubuntu VirtualBox console for administration. Perform all authentication testing within the isolated lab.

## Assessment Tasks

1. **Establish authentication baseline.** Inspect SSH settings and verify connectivity from Debian.
2. **Simulate failed authentication.** Generate three controlled SSH password failures.
3. **Investigate authentication logs.** Identify the source IP, target account, and repeated failures.
4. **Configure automated protection.** Create an SSH jail and verify the active detection and response parameters.
5. **Validate detection and response.** Trigger an automated ban and verify SSH is restricted while HTTP remains accessible.
6. **Investigate and recover.** Inspect nftables enforcement, manually remove the ban, and verify legitimate SSH authentication.
7. **Restore baseline.** Remove the exercise-specific configuration, stop and disable Fail2ban, and verify service accessibility.

## Success Criteria

| Check | Required result |
|---|---|
| Initial SSH connectivity | TCP 22 accessible from Debian |
| Authentication logs | Three failed password events identify Debian and the target account |
| Active SSH jail | Three failures within 60 seconds trigger a 300-second ban |
| Automated detection | Fail2ban logs matching failures and a ban event |
| Access restriction | SSH blocked; HTTP remains accessible |
| Firewall enforcement | nftables address set contains the banned IP during an active ban |
| Recovery | Ban removed; address set empty; legitimate SSH login succeeds |
| Restored baseline | Fail2ban inactive and disabled; exercise configuration removed; TCP 22 and 80 accessible |

Capture evidence for each major validation step. The walkthrough provides suggested screenshot filenames and observed results where available.

## Knowledge Check

1. How do brute-force attacks, password spraying, and credential stuffing differ?
2. Why does public-key authentication reduce exposure to password-guessing attacks when password authentication is disabled?
3. What authentication log fields help distinguish repeated failures against one account from a distributed authentication attack?
4. How do detection thresholds and time windows affect false positives and missed detections?
5. What are the limitations of source-IP-based blocking when multiple legitimate users share an IP address or an attacker uses multiple source addresses?
6. Why is blocking an offending IP insufficient to establish that an account has not already been compromised, and what additional evidence should an investigator review?

## Cleanup and Safety

Remove only the exercise-specific jail configuration, stop and disable Fail2ban, and confirm its firewall restrictions are removed. Preserve authentication logs and leave Fail2ban installed. Do not change OPNsense policies or conduct authentication testing outside the isolated lab.
