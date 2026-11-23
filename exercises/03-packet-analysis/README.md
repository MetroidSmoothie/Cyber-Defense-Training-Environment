# Exercise 3 — Packet Capture & Protocol Analysis

**Difficulty:** Beginner  
**Focus:** Packet capture, ICMP, TCP, UDP, HTTP, and firewall traffic analysis  
**Platform:** OPNsense 26.7, Kali Linux, Ubuntu Server, Debian, VirtualBox

## Overview

Capture and interpret traffic between lab hosts using `tcpdump`, then correlate packets observed on OPNsense's WAN interface with firewall log entries. Compare ICMP, TCP, UDP, and HTTP traffic to understand how different protocols appear in packet captures.

For commands, brief output explanations, verified results, and screenshots, see the [step-by-step walkthrough](03-walkthrough.md).

## Scenario

You are investigating network communications in a segmented lab. Internal hosts exchange diagnostic and web traffic, while an external workstation attempts to access a protected web server. Your task is to capture these exchanges, identify key protocol fields, and determine what happened to traffic blocked at the firewall.

## Learning Objectives

- Identify a network interface and capture traffic with protocol filters.
- Recognize ICMP Echo Requests and Echo Replies.
- Identify the SYN, SYN-ACK, and ACK packets of a TCP three-way handshake.
- Read HTTP request and response headers in an unencrypted capture.
- Correlate inbound TCP SYN packets with OPNsense firewall block logs.
- Compare TCP connection establishment with UDP datagrams and ICMP error messages.
- Stop captures and return the lab to its normal isolated configuration.

## Prerequisites

Complete the [Lab Environment Setup](../../Lab%20Environment%20Setup.md). Confirm that `tcpdump` is available on Ubuntu, Wireshark is available on Kali, and Apache is running on Ubuntu. Keep the temporary HTTP allow rule from Exercise 1 removed.

| Component | Address | Role |
|---|---|---|
| Kali (`Lab-External`) | `192.168.50.10/24` | External testing host |
| OPNsense WAN | `192.168.50.1/24` | External firewall interface |
| OPNsense LAN | `172.16.77.1/24` | Internal firewall interface |
| Ubuntu (`Lab-Internal`) | `172.16.77.10/24` | Packet capture host and Apache server |
| Debian (`Lab-Internal`) | `172.16.77.11/24` | Internal traffic source |

**Before starting:** Verify the four VMs are operational and Kali uses only its `Lab-External` adapter for external testing.

> **Isolation requirement:** Keep all VMs on VirtualBox Internal Networks. Kali may temporarily use a second adapter on `PFSense Network` to administer OPNsense; verify that test traffic still routes through OPNsense WAN, and disable the adapter when finished.

## Assessment Tasks

1. **Prepare capture tools.** Verify `tcpdump` on Ubuntu, Wireshark on Kali, and Ubuntu's capture interface.
2. **Capture ICMP traffic.** Ping Ubuntu from Debian and identify matching Echo Requests and Echo Replies.
3. **Analyze a TCP handshake.** Capture an HTTP connection and identify SYN, SYN-ACK, and ACK packets.
4. **Inspect HTTP traffic.** Examine a readable HTTP `HEAD` request and successful server response.
5. **Investigate blocked traffic.** Capture Kali's TCP connection attempts on OPNsense WAN and correlate them with firewall block logs.
6. **Compare protocol behavior.** Send a UDP datagram to an unused port and identify the resulting ICMP Port Unreachable message.
7. **Clean up and validate.** Stop packet captures, close capture jobs, and restore Kali's normal adapter configuration without changing the baseline firewall policy.

## Success Criteria

| Check | Required result |
|---|---|
| Capture tools and interface | Available; Ubuntu interface identified |
| Debian → Ubuntu ICMP | Echo Requests and Echo Replies observed |
| TCP connection establishment | SYN → SYN-ACK → ACK identified |
| HTTP inspection | `HEAD` request and `200 OK` response visible |
| Kali → Ubuntu HTTP attempt | TCP SYN packets observed on OPNsense WAN |
| Firewall correlation | Matching WAN inbound default-deny entries identified |
| UDP to unused port | UDP datagram and ICMP Port Unreachable observed |
| Cleanup | Captures stopped; temporary Kali management adapter disabled |

Capture clear evidence of each major protocol exchange. The walkthrough includes suggested screenshot filenames and a TCP handshake diagram.

## Knowledge Check

1. What information can a packet capture reveal about network communication?
2. What is the purpose of the TCP three-way handshake?
3. How do TCP and UDP differ in how they establish communication?
4. What is the difference between an ICMP Echo Reply and an ICMP Destination Unreachable message?
5. Why can HTTP request headers be readable in a packet capture, and how does HTTPS change this?
6. Why does observing a packet arrive at a firewall not necessarily mean it reached its intended destination?

## Walkthrough

Compare your results with the [Exercise 3 Walkthrough](03-walkthrough.md) for commands, brief output explanations, screenshots, and verified results.

## Cleanup and Safety

Stop active packet captures, close OPNsense capture jobs, and disable Kali's temporary LAN management adapter. Preserve the original firewall rules and keep the lab isolated from production, home, and public networks.
