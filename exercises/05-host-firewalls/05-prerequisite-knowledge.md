# Prerequisite Knowledge — Exercise 05

## Host-Based Firewall Defense

This document introduces the concepts relevant to protecting individual systems, restricting internal network traffic, and validating host-based firewall controls.

## 1. Host-Based vs. Network-Based Firewalls

A host-based firewall filters network traffic on an individual system. A network-based firewall filters traffic that passes through it, typically between networks or security zones.

| Firewall Type | Scope | Example |
| --- | --- | --- |
| Network-based | Traffic traversing the firewall between networks | OPNsense |
| Host-based | Traffic entering, leaving, or being forwarded by an individual host | UFW on Ubuntu |

In this lab, Ubuntu (`172.16.77.10`) and Debian (`172.16.77.11`) share the same subnet. Their direct communication does not normally traverse OPNsense. A host-based firewall can restrict this communication even when the perimeter firewall never sees it.

## 2. Traffic Direction

Firewall policies can distinguish traffic based on its direction relative to the system enforcing the policy.

| Direction | Description |
| --- | --- |
| Inbound | Traffic destined for the local system |
| Outbound | Traffic originating from the local system |
| Forwarded | Traffic passing through the system toward another destination |

Direction is relative to the system observing the traffic. An HTTP request sent by Debian is outbound from Debian and inbound to Ubuntu.

A stateful firewall can permit response traffic associated with an allowed connection without requiring a separate rule for every response.

## 3. North-South vs. East-West Traffic

These terms describe traffic patterns relative to a network or security boundary.

| Traffic Pattern | Description | Lab Example |
| --- | --- | --- |
| North-south | Traffic entering or leaving a defined network boundary | Kali communicating with Ubuntu through OPNsense |
| East-west | Traffic between systems within an environment | Debian communicating directly with Ubuntu |

East-west traffic can also cross internal security boundaries in larger environments. It is not necessarily confined to a single subnet.

## 4. Lateral Movement

Lateral movement occurs when an attacker moves from an initially compromised system to other systems within an environment. This may involve abusing remote services, stolen credentials, or existing trust relationships.

A system being located on an internal network does not establish that it is trustworthy. Restricting unnecessary communication between internal systems reduces opportunities for lateral movement.

For example, a host may need access to a web server without needing SSH access to that same server. Allowing HTTP while restricting SSH reduces unnecessary exposure without preventing the required functionality.

## 5. Defense in Depth

Defense in depth is the use of multiple complementary security controls so that security does not depend entirely on a single mechanism.

| Security Control | Purpose |
| --- | --- |
| Network segmentation | Restricts communication between network segments |
| Host-based firewall | Restricts communication at the individual system |
| Service hardening | Reduces unnecessary service exposure and insecure configurations |
| Authentication and authorization | Restricts access to approved identities and permissions |
| Security logging | Provides visibility into activity and policy enforcement |

These controls address different aspects of security. A firewall allowing access to SSH, for example, does not establish that the connecting user is authorized to log in.

## 6. Firewall Policies and Least Privilege

A default policy determines how traffic is handled when no applicable rule permits or otherwise handles it. Explicit rules define exceptions based on criteria such as source address, destination address, protocol, and port.

| Policy | Description |
| --- | --- |
| Default deny | Blocks traffic unless explicitly permitted |
| Default allow | Permits traffic unless explicitly restricted |

The principle of least privilege applies to network communication as well as user permissions. Systems should have only the network access required to perform their intended functions.

In this exercise, Ubuntu will deny unsolicited inbound traffic by default while permitting HTTP from a specific internal host. Outbound traffic will remain allowed. This is an example policy for the lab, not a universal recommendation for every environment.

## 7. Linux Firewall Architecture

Linux provides packet filtering capabilities through Netfilter, a framework within the Linux kernel. Administrators use tools to configure the rules that govern this filtering.

| Component | Description |
| --- | --- |
| Netfilter | Kernel framework for packet filtering, connection tracking, and related network operations |
| iptables | Userspace tool for managing firewall rules through the traditional iptables interface |
| nftables | Modern Linux packet filtering framework, commonly managed using the `nft` tool |
| UFW (Uncomplicated Firewall) | Higher-level interface that simplifies firewall configuration |

On modern Linux distributions, iptables commands may use an nftables compatibility backend. UFW simplifies policy management rather than replacing the kernel's packet filtering functionality. The underlying implementation depends on the distribution and configuration.

## 8. Host-Based Firewall Logging

Firewall logs provide evidence of traffic observed and handled by the firewall. Linux host-based firewall logs may contain the following fields:

| Field | Description |
| --- | --- |
| IN | Interface on which traffic arrived |
| OUT | Outgoing interface, when applicable |
| SRC | Source IP address |
| DST | Destination IP address |
| PROTO | Network or transport protocol |
| SPT | Source port, when applicable |
| DPT | Destination port, when applicable |
| SYN | TCP flag indicating a synchronization request, commonly associated with connection initiation |

A UFW BLOCK entry with a source of `172.16.77.11`, destination of `172.16.77.10`, protocol TCP, and destination port 22 indicates that traffic from Debian to Ubuntu's SSH port was blocked by the logged policy decision.

A blocked connection does not independently establish malicious intent. Legitimate applications, configuration errors, and authorized security testing can all generate blocked traffic. Similarly, the absence of a log entry does not prove that traffic was allowed, since logging settings and rate limits affect visibility.

Validate firewall policies by testing both permitted and restricted traffic and comparing the results with the configured policy and available logs.

---

Return to the [Exercise 05 README](README.md) or continue to the [walkthrough](05-walkthrough.md).
