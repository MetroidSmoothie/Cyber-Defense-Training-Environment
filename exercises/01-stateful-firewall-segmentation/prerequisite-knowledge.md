# Prerequisite Knowledge — Stateful Firewall & Network Segmentation

The following concepts will help you understand what is happening throughout Exercise 01 and why the firewall behaves the way it does. You don't need to memorize everything before beginning the exercise. Refer back to this file as needed.

## 1. IPv4 Addressing and Subnets

An IPv4 address identifies a network interface at the network layer. A subnet is a logical grouping of IP addresses. Devices use a subnet mask to determine whether a destination is on their local network or must be reached through a router.

CIDR (Classless Inter-Domain Routing) notation expresses the number of bits in the network prefix. For example, `/24` corresponds to the subnet mask `255.255.255.0`.

| CIDR | Meaning |
|---|---|
| `172.16.77.0/24` | Internal lab subnet, with conventional host addresses .1 through .254 |
| `192.168.50.0/24` | External lab subnet, with conventional host addresses .1 through .254 |
| `192.168.50.10/32` | One specific IPv4 address |

A firewall rule using `/32` can restrict access to a single source or destination IP rather than an entire subnet.

## 2. Network Interfaces and Routing

A network interface connects a system to a network. A firewall can have multiple interfaces, each connected to a different network segment.

- **LAN (Local Area Network):** Typically the internal network.
- **WAN (Wide Area Network):** Typically the upstream or external-facing network. In this lab, WAN is an isolated simulated external network, not the public internet.
- **Router:** Forwards packets between networks according to its routing table.
- **Default gateway:** The next hop used when no more specific route matches a destination.

In this environment, OPNsense connects the external and internal subnets. Kali reaches the internal subnet through OPNsense using a specific route. Devices on the same subnet can ordinarily communicate directly without traversing the firewall.

Routing determines where traffic should go; firewall policy determines whether that traffic is permitted.

## 3. Network Protocols

| Protocol | Description |
|---|---|
| TCP (Transmission Control Protocol) | Connection-oriented transport protocol providing reliable, ordered delivery. |
| UDP (User Datagram Protocol) | Connectionless transport protocol without built-in delivery or ordering guarantees. |
| ICMP (Internet Control Message Protocol) | Network-layer protocol used for diagnostic and control messages, including ping. |

TCP normally establishes a connection using a three-way handshake: SYN, SYN-ACK, ACK. A stateful firewall can track this exchange and subsequent traffic. UDP has no equivalent handshake, but a firewall can maintain temporary state for UDP traffic. ICMP does not use TCP or UDP port numbers.

## 4. Ports and Network Services

TCP and UDP port numbers identify transport-layer endpoints. A listening service waits for incoming traffic on a particular address and port.

| Service | Default port | Transport |
|---|---:|---|
| SSH | 22 | TCP |
| DNS | 53 | UDP/TCP |
| HTTP | 80 | TCP |
| HTTPS | 443 | TCP |

When a client connects to a web server, it typically uses a temporary source port (an **ephemeral port**) and destination port 80 or 443. The server's response reverses the source and destination endpoints.

For example:

```text
Request:  192.168.50.10:52000 -> 172.16.77.10:80
Response: 172.16.77.10:80    -> 192.168.50.10:52000
```

The source port in this example is illustrative. A port number suggests a service but does not prove which application or protocol is actually in use.

## 5. Firewall Rules

A firewall controls network traffic according to a defined policy. Rules can match characteristics such as:

| Rule component | Purpose |
|---|---|
| Interface and direction | Where traffic enters or leaves the firewall |
| Source | Where traffic originates |
| Destination | Where traffic is going |
| Protocol | TCP, UDP, ICMP, etc. |
| Destination port | The targeted TCP or UDP service |
| Action | Permit, silently block, or reject matching traffic |
| Logging | Record matching traffic for investigation |

OPNsense interface rules normally evaluate traffic as it enters the selected interface. For ordinary quick interface rules, the first matching rule determines the action. Other rule types and existing connection states can affect processing.

A **block** typically drops traffic silently, whereas a **reject** returns an error to the sender. A **pass** permits matching traffic. Permitting traffic does not guarantee that the destination service is running or reachable.

## 6. Stateful vs. Stateless Firewalls

A stateless firewall evaluates packets independently against its rules. A stateful firewall also maintains information about tracked traffic in a **state table**.

When an allowed TCP connection is established, a stateful firewall can recognize its return traffic without requiring a separate rule allowing arbitrary inbound connections to the client's ephemeral port.

A state table may include source and destination addresses, ports, protocol, connection state, and expiration information. TCP states such as `ESTABLISHED` and `FIN_WAIT_2` describe different stages of a connection's lifecycle. Entries can remain temporarily after an application finishes communicating.

Existing states may continue to permit traffic after a firewall rule is changed or removed. When validating a new policy, distinguish new connections from existing tracked connections.

## 7. Network Segmentation and Least Privilege

Network segmentation divides a network into separate areas to control communication and limit the spread of security incidents. Simply placing devices in different subnets does not guarantee isolation; routing and access controls determine which traffic can cross the boundary.

**Default deny** means traffic is blocked unless explicitly permitted. **Least privilege** means granting only the access necessary for a particular function.

For example, permitting HTTP from Kali to Ubuntu does not require permitting SSH or ICMP between those same systems. A narrowly scoped firewall rule reduces unnecessary exposure.

Segmentation is one layer of defense. It does not replace secure host configurations, authentication, patching, or application security.

## 8. Firewall Logging and Connectivity Testing

Firewall logs provide visibility into traffic that matches logged rules. Common fields include timestamps, interfaces, direction, source and destination addresses, ports, protocol, action, and rule identifiers.

A blocked connection is not automatically evidence of malicious activity, and a permitted connection is not automatically evidence of legitimate activity. Context matters.

Different connectivity tests answer different questions:

| Observation | What it tells you |
|---|---|
| Successful ping | ICMP echo traffic can travel to the target and back. |
| Failed ping | ICMP did not succeed; the target may still offer accessible TCP services. |
| Successful TCP connection | A TCP connection to the tested endpoint could be established. |
| HTTP 200 OK | The web server successfully handled the HTTP request. |
| Firewall block log | The firewall recorded a matching blocked packet. |

A timeout alone does not prove a firewall caused the failure. Routing, service availability, and host configuration can also affect connectivity. Comparing test results with firewall logs and state information helps establish what actually happened.

---

Return to the [Exercise 01 README](README.md) or continue to the [walkthrough](01-walkthrough.md).
