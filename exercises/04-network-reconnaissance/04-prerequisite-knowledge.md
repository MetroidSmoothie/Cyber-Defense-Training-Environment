# Prerequisite Knowledge — Exercise 04

## Network Reconnaissance & Firewall Testing

This document introduces the concepts relevant to discovering network hosts, identifying exposed services, and validating firewall controls.

## 1. Network Reconnaissance

Network reconnaissance is the process of gathering information about a network, its hosts, and the services they provide. Security professionals use reconnaissance to identify exposed resources, assess attack surfaces, and validate security controls.

| Type | Description | Example |
| --- | --- | --- |
| Passive reconnaissance | Collecting information without directly probing the target | Reviewing existing network logs or public DNS records |
| Active reconnaissance | Sending traffic to a target to obtain information | Host discovery, port scanning, service enumeration |

Active reconnaissance can generate observable network traffic and may trigger firewall logs or intrusion detection alerts. Only scan systems you own or have authorization to test.

## 2. Host Discovery

Host discovery attempts to determine which systems are reachable or responsive on a network.

| Technique | Description |
| --- | --- |
| ARP discovery | Uses Address Resolution Protocol to identify IPv4 hosts on the local network segment. ARP requests are not normally forwarded by routers. |
| ICMP discovery | Sends ICMP messages, such as echo requests, to determine whether a host responds. |
| TCP discovery | Sends TCP probes and uses responses to infer whether a host is reachable. |

A host that does not respond to a discovery probe is not necessarily offline. Firewalls, host security settings, and network connectivity issues can prevent responses.

In this lab, Kali and OPNsense share an external network segment. Ubuntu resides on a separate internal segment. Discovering hosts on the external segment does not establish whether hosts on the internal segment are reachable.

## 3. TCP Port Scanning

Port scanning attempts to determine which network ports are accessible on a target. TCP scanning techniques use the behavior of the TCP protocol to infer port states.

| Scan type | Description |
| --- | --- |
| TCP SYN scan | Sends a SYN and evaluates the response without normally completing the TCP handshake. An open port typically returns SYN-ACK, after which the scanner sends a reset. |
| TCP connect scan | Uses the operating system to attempt a complete TCP connection. An open port typically completes the three-way handshake. |

Both techniques generate network traffic and can be detected. A SYN scan is sometimes called a half-open scan, but this does not mean it is invisible to security monitoring tools.

## 4. Port States

Port states describe what a scanner can infer from its probes, not necessarily the underlying configuration of the target.

| State | Meaning | Typical TCP scan observation |
| --- | --- | --- |
| Open | A service accepts connections on the port from the scanner's perspective. | SYN-ACK |
| Closed | The port is reachable, but no service is accepting connections, or a device actively rejects the probe. | TCP RST |
| Filtered | The scanner cannot determine whether the port is open or closed because probes or responses appear to be filtered. | No response or certain ICMP unreachable messages |

A port may appear open from one network segment and filtered from another. A filtered port does not establish whether a service is listening behind the firewall.

## 5. Service Enumeration and Version Detection

Service enumeration attempts to identify the applications or protocols accessible on a target. Version detection goes further by attempting to identify the software product and version.

- **Banner grabbing:** Examining identifying information returned by a service, such as an SSH identification string or HTTP Server header.
- **Service probing:** Sending protocol-specific requests and comparing responses against known service characteristics.

| Port (TCP) | Common service | Example software |
| --- | --- | --- |
| 22 | SSH | OpenSSH |
| 80 | HTTP | Apache HTTP Server |
| 443 | HTTPS | Apache HTTP Server, Nginx |

Port numbers alone do not prove which service is running. Applications can use nonstandard ports, and banners may be hidden, modified, or inaccurate. A reported software version also does not independently establish that a vulnerability is present; vendors may backport security fixes without changing the upstream version number.

## 6. Network Exposure and Attack Surface

A system's network attack surface consists of the network-accessible services and interfaces through which it could potentially be attacked. Exposure depends on both the services listening on the host and the network path available to the source.

In this lab, the same Ubuntu server can present different results depending on where a scan originates:

| Source | Network position | Expected baseline visibility |
| --- | --- | --- |
| Debian | Internal network, same subnet as Ubuntu | SSH and HTTP accessible; HTTPS closed if no listener exists |
| Kali | External lab network, separated by OPNsense | SSH, HTTP, and HTTPS filtered by the baseline WAN policy |

The internal network is not inherently trustworthy. Services accessible to internal hosts may still be exposed to compromised systems or unauthorized users.

## 7. Firewall Testing and Security Control Validation

Firewall testing determines whether observed network behavior matches the intended security policy. A firewall rule should permit only the traffic required for a legitimate purpose while restricting unnecessary access.

For example, a narrowly scoped rule allowing Kali to access Ubuntu over HTTP should not also permit SSH or HTTPS. Comparing scan results before and after introducing the rule helps validate that the exception is working as intended.

Firewall logs can provide additional evidence by identifying the source and destination addresses, protocol, destination port, interface, action, and matching rule when those details are logged.

A successful connection demonstrates reachability for the tested source, destination, protocol, and port. It does not prove that every other path or service is secured.

## 8. Reconnaissance Limitations

Reconnaissance results should be interpreted in the context of the scan method, network position, and security controls in place.

| Observation | Important consideration |
| --- | --- |
| Host appears down | Discovery probes may be blocked even when the host is online. |
| Port appears filtered | A firewall may be dropping traffic, but packet loss or routing problems can produce similar results. |
| Port appears open | A reachable service is not necessarily vulnerable. |
| Service version is reported | Identification may be inaccurate or incomplete. |
| No firewall log is found | Logging may be disabled, filtered, or unavailable. |

Skipping host discovery causes a scanner to attempt scanning the target without first establishing that it responds to discovery probes. It does not prove that the target is online.

Corroborate scan results with other evidence, such as host listening sockets, routing information, packet captures, and firewall logs. Distinguish what you directly observed from what you infer about the cause.

---

Return to the [Exercise 04 README](README.md) or continue to the [walkthrough](04-walkthrough.md).
