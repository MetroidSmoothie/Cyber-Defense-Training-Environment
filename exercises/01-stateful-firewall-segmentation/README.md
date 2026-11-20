# Exercise 1 — Stateful Firewall & Network Segmentation

**Difficulty:** Beginner  
**Focus:** Firewall policy, network segmentation, stateful inspection, and log analysis  
**Platform:** OPNsense 26.7, Kali Linux, Ubuntu Server, VirtualBox

## Overview

Practice implementing and validating a least-privilege firewall exception. You'll first establish that the default WAN policy blocks access to a protected web server, then allow a single service, examine how OPNsense tracks and logs the connection, and restore the original policy.

For configuration details, command explanations, verified results, and screenshots, see the [step-by-step walkthrough](01-walkthrough.md).

## Scenario

You are responsible for a small segmented network. An external testing workstation needs temporary HTTP access to an internal Ubuntu web server. Your task is to grant only the required access, prove that other services remain protected, and remove the exception when testing is complete.

## Learning Objectives

- Identify listening services and verify a protected host's network configuration.
- Establish a default-deny baseline using connectivity tests and firewall logs.
- Create a narrowly scoped OPNsense rule using specific source, destination, protocol, and port values.
- Validate permitted HTTP traffic while confirming that SSH and ICMP remain blocked.
- Examine state-table entries and explain stateful versus stateless filtering.
- Correlate permitted and denied traffic with firewall log entries.
- Remove temporary access and verify that the original policy is restored.

## Prerequisites

Complete the [Lab Environment Setup](../../Lab%20Environment%20Setup.md) guide and confirm that the four-VM environment is operational.

| Component | Address | Role |
|---|---|---|
| Kali (`Lab-External`) | `192.168.50.10/24` | External testing host |
| OPNsense WAN | `192.168.50.1/24` | External firewall interface |
| OPNsense LAN | `172.16.77.1/24` | Internal firewall interface |
| Ubuntu (`Lab-Internal`) | `172.16.77.10/24` | Protected Apache HTTP and SSH host |
| Debian (`Lab-Internal`) | `172.16.77.11/24` | Additional protected host; not directly tested in this exercise |

**Before starting:** Verify that Apache and SSH are available on Ubuntu, OPNsense is running, and Kali's normal configuration uses only the `Lab-External` adapter. Take a snapshot of OPNsense before modifying firewall rules.

> **Isolation requirement:** Keep the lab on VirtualBox Internal Networks. OPNsense administration may require temporarily enabling Kali's second adapter on `PFSense Network`; disable it before segmentation tests. The [walkthrough](01-walkthrough.md) explains this procedure and the routing checks needed to avoid bypassing the WAN firewall.

## Assessment Tasks

1. **Identify an internal service.** Confirm Ubuntu's IP address and identify its listening HTTP and SSH services. Verify Apache responds locally.
2. **Establish baseline connectivity.** From Kali, test HTTP and ICMP access to Ubuntu. Locate the corresponding default-deny events in OPNsense.
3. **Configure least-privilege access.** Create a logged, stateful WAN rule allowing only Kali (`192.168.50.10`) to reach Ubuntu (`172.16.77.10`) on TCP port 80. Preserve the existing Kali-to-OPNsense WAN ICMP exception.
4. **Validate access restrictions.** Confirm that HTTP succeeds while SSH (TCP 22) and ICMP to Ubuntu remain blocked.
5. **Inspect connection states.** Generate HTTP traffic and identify the matching TCP connection in OPNsense's state table. Explain the role of connection tracking.
6. **Review firewall logs.** Find the WAN inbound pass entry for the HTTP exception and compare it with the earlier denied traffic.
7. **Restore the baseline.** Remove the temporary HTTP rule, apply the change, and verify that Kali can no longer reach Ubuntu over HTTP or ICMP while the original WAN ICMP exception still works.

## Success Criteria

| Check | Required result |
|---|---|
| Initial Kali → Ubuntu HTTP and ICMP | Blocked |
| Default-deny log evidence | WAN inbound deny events identified |
| Temporary Kali → Ubuntu HTTP | Allowed on TCP 80 only |
| Kali → Ubuntu SSH and ICMP during exception | Still blocked |
| Stateful inspection | Matching HTTP TCP states observed |
| HTTP allow-rule logging | WAN inbound pass event identified |
| Final Kali → Ubuntu HTTP and ICMP | Blocked again |
| Final Kali → OPNsense WAN ICMP | Allowed by original rule |
| Cleanup | Temporary HTTP rule removed; Kali's management adapter disabled |

Capture clear evidence for each major validation step. The walkthrough provides suggested screenshot filenames and examples from a completed run.

## Knowledge Check

1. Why is a rule restricted to a `/32` source and destination more specific than a subnet-wide rule?
2. Why can an HTTP connection succeed without a separate firewall rule for its return traffic?
3. What distinguishes a stateful firewall from a stateless firewall?
4. Why might two TCP connections to the same destination port use different source ports?
5. Why should you check existing connection states when removing a temporary firewall exception?
6. **Match each service to its default port:** HTTP, HTTPS, SSH, DNS, and RDP — **22, 53, 80, 443, 3389**. Which of these services commonly uses both TCP and UDP?

## Walkthrough

**Ready to compare your work with a verified implementation?** Open the [Exercise 1 Walkthrough](01-walkthrough.md) for step-by-step instructions, command and flag explanations, screenshots, troubleshooting notes, and observed results.

## Cleanup and Safety

Remove the exercise-specific HTTP exception, confirm the original WAN ICMP rule remains enabled, and disable Kali's temporary LAN management adapter. Do not connect the lab to production, home, or public networks. Use the walkthrough's final validation steps to confirm the restored baseline before beginning Exercise 2.
