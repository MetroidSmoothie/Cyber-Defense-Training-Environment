# Exercise 4 — Network Reconnaissance & Firewall Testing

**Difficulty:** Beginner–Intermediate  
**Focus:** Host discovery, service enumeration, network segmentation, and firewall scan validation  
**Platform:** OPNsense 26.7, Kali Linux, Debian, Ubuntu Server, VirtualBox

## Overview

Use Nmap to discover hosts, enumerate services, and compare what internal and external systems can identify. Temporarily permit access to one protected service, observe how reconnaissance results change, and restore the original firewall policy.

For commands, concise output explanations, verified results, and screenshots, see the [step-by-step walkthrough](04-walkthrough.md).

## Scenario

You are assessing a segmented network to determine which systems and services are discoverable from different network locations. An external testing host must be granted temporary HTTP access to a protected server. Your task is to compare scan results before and after the exception, verify firewall decisions, and remove the temporary access.

## Learning Objectives

- Discover active hosts on a directly connected subnet.
- Explain how firewall filtering affects host discovery and Nmap results.
- Identify open ports and service versions from an internal network.
- Compare internal and external reconnaissance of the same host.
- Validate how a narrowly scoped firewall exception changes service visibility.
- Correlate Nmap results with OPNsense pass and block logs.
- Restore the original firewall policy and verify its behavior.

## Prerequisites

Complete the [Lab Environment Setup](../../Lab%20Environment%20Setup.md). Confirm that all four VMs are operational, Nmap is available on Kali and Debian, and Ubuntu is running Apache and SSH.

| Component | Address | Role |
|---|---|---|
| Kali (`Lab-External`) | `192.168.50.10/24` | External scanning host |
| OPNsense WAN | `192.168.50.1/24` | External firewall interface |
| OPNsense LAN | `172.16.77.1/24` | Internal firewall interface |
| Ubuntu (`Lab-Internal`) | `172.16.77.10/24` | Protected HTTP and SSH server |
| Debian (`Lab-Internal`) | `172.16.77.11/24` | Internal scanning host |

**Before starting:** Confirm that the original Kali-to-OPNsense WAN ICMP exception is enabled and no temporary HTTP exception remains. Take an OPNsense snapshot before changing firewall rules.

> **Isolation requirement:** Keep all VMs on VirtualBox Internal Networks. Kali's temporary management adapter on `PFSense Network` must not bypass OPNsense during external scans. Verify the route to Ubuntu uses Kali's `eth0` interface via `192.168.50.1`, and disable the management adapter after use.

## Assessment Tasks

1. **Establish the network baseline.** Verify Kali's interface configuration, route to Ubuntu, and ICMP connectivity to OPNsense WAN.
2. **Perform host discovery.** Discover hosts on Kali's external subnet, attempt discovery of Ubuntu across the firewall, and compare with a scan that skips host discovery.
3. **Enumerate internal services.** From Debian, identify Ubuntu's TCP 22, 80, and 443 states and detect available service versions.
4. **Test external reconnaissance.** Scan the same ports from Kali and compare service visibility with Debian's results.
5. **Validate a temporary firewall exception.** Create a logged WAN rule allowing Kali to access Ubuntu on TCP 80 only, then repeat service enumeration.
6. **Analyze scan results and firewall logs.** Correlate permitted HTTP traffic and blocked SSH/HTTPS probes with OPNsense log entries.
7. **Restore and verify the baseline.** Remove the temporary HTTP rule, confirm all three ports are filtered from Kali, and verify the original WAN ICMP exception remains functional.

## Success Criteria

| Check | Required result |
|---|---|
| External subnet discovery | Kali and OPNsense WAN identified |
| Cross-firewall host discovery | Ubuntu not discovered by standard probes |
| Forced external port scan | TCP 22 and 80 reported as filtered |
| Debian → Ubuntu service enumeration | SSH and HTTP open; HTTPS closed; versions identified where available |
| Initial Kali → Ubuntu service enumeration | TCP 22, 80, and 443 filtered; versions not identified |
| Temporary HTTP exception | TCP 80 open with Apache identified; TCP 22 and 443 remain filtered |
| Firewall log correlation | HTTP pass and SSH/HTTPS block entries identified |
| Restored baseline | All three ports filtered from Kali; OPNsense WAN ping succeeds |
| Cleanup | Temporary HTTP rule removed; Kali management adapter disabled |

Capture evidence for each major validation step. The walkthrough includes the observed results and suggested screenshot filenames.

## Knowledge Check

1. What is the difference between host discovery and port scanning?
2. Why might a host appear offline during discovery even though it is running?
3. What do Nmap's `open`, `closed`, and `filtered` port states indicate?
4. What information can service-version enumeration reveal to a network administrator or an attacker?
5. How does network segmentation reduce the information available to a system on another network?
6. Why does limiting access to a single destination port reduce exposure compared with allowing unrestricted access to a host?

## Walkthrough

Compare your results with the [Exercise 4 Walkthrough](04-walkthrough.md) for step-by-step instructions, command explanations, screenshots, and verified results.

## Cleanup and Safety

Remove `EX04 - Allow Kali HTTP Recon`, apply the firewall change, preserve the original WAN ICMP exception, and disable Kali's temporary LAN management adapter. Keep the lab isolated from production, home, and public networks.
