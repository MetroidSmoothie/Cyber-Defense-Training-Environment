# Exercise 07 — Prerequisite Knowledge

## Suricata IDS & Detection Engineering

This document introduces the concepts behind network intrusion detection, signature-based detection, alert analysis, and detection engineering.

## 1. Intrusion Detection and Prevention

An Intrusion Detection System (IDS) monitors activity for indicators of suspicious or malicious behavior and generates alerts when detection criteria are met. An Intrusion Prevention System (IPS) can additionally intervene to prevent matching traffic or activity.

| System | Function |
| --- | --- |
| IDS | Inspects activity and generates alerts without directly blocking matching traffic. |
| IPS | Inspects activity and can block matching traffic according to its configured policy. |
| Firewall | Controls network traffic according to configured access policies. |

An IDS typically monitors traffic passively, whereas a network IPS typically operates inline, meaning traffic passes through its inspection path. Firewalls and intrusion detection systems complement one another: traffic permitted by a firewall can still contain suspicious content.

## 2. IDS Classifications

| Classification | Description |
| --- | --- |
| Network-based IDS (NIDS) | Inspects network traffic visible to a monitoring interface. |
| Host-based IDS (HIDS) | Monitors activity on individual systems, such as logs, file changes, and processes. |

A NIDS can monitor communication between multiple systems, while a HIDS can provide visibility into activity that may not be observable from the network alone.

## 3. Detection Methodologies

| Method | Description |
| --- | --- |
| Signature-based detection | Identifies activity matching predefined patterns or conditions. |
| Anomaly-based detection | Identifies deviations from expected or established behavior. |
| Protocol analysis | Examines protocol structure, state, and behavior for suspicious or unexpected characteristics. |

Signatures do not necessarily identify a specific malware sample. They can also identify reconnaissance activity, suspicious protocol behavior, or violations of an organization's security policy. These detection methods can overlap.

## 4. Network Traffic Inspection and Visibility

An IDS can only inspect traffic that reaches its monitoring interface. Network topology, sensor placement, routing, and encryption all influence what the sensor can observe.

In this environment, Suricata monitors the OPNsense WAN interface in passive packet capture (PCAP) mode.

| Traffic | Expected visibility on the WAN interface |
| --- | --- |
| Kali (192.168.50.10) to OPNsense WAN (192.168.50.1) | Visible |
| Kali to Ubuntu (172.16.77.10), routed toward OPNsense | Incoming traffic visible, even if subsequently blocked by the firewall |
| Debian (172.16.77.11) to Ubuntu on the same LAN | Normally not visible |

An IDS alert does not establish that a packet passed the firewall or reached its intended destination. Firewall logs and endpoint evidence can help establish what happened after detection.

Encrypted traffic can still expose metadata such as IP addresses, ports, packet sizes, and timing. However, encrypted application payloads generally cannot be inspected as plaintext without an appropriate decryption mechanism.

## 5. Suricata

Suricata is an open-source network IDS, IPS, and network security monitoring engine. It captures and decodes network traffic, tracks flows, analyzes supported application protocols, and evaluates traffic against detection rules.

| Term | Description |
| --- | --- |
| Ruleset | A collection of detection signatures, often grouped by threat type or activity. |
| Flow | A tracked exchange of network traffic between endpoints. |
| Alert | An event generated when traffic satisfies a detection rule. |
| Emerging Threats (ET) Open | A publicly available collection of network detection rules. |
| EVE JSON | Suricata's structured event output format, which can contain alerts and other network telemetry. |

Downloading a ruleset does not necessarily enable every signature within it. Enabled rules, rule policies, and monitoring settings determine which detections are active.

## 6. Detection Signatures

A Suricata signature consists of a rule header and rule options. The header defines the action and traffic criteria, while the options define additional matching conditions and descriptive information.

The following illustrative rule detects an ICMP echo request from Kali to the OPNsense WAN address:

```text
alert icmp 192.168.50.10 any -> 192.168.50.1 any (msg:"LAB ICMP Echo Request"; itype:8; sid:1000001; rev:1;)
```

| Component | Description |
| --- | --- |
| `alert` | Generates an alert when the rule matches. |
| `icmp` | Protocol to inspect. |
| `192.168.50.10 any` | Source address and unrestricted port field (ICMP does not use transport ports). |
| `->` | Direction of the match. |
| `192.168.50.1 any` | Destination address and unrestricted port field. |
| `msg` | Human-readable alert message. |
| `itype:8` | Matches IPv4 ICMP echo requests. |
| `sid` | Signature identifier, which must be unique within the active ruleset. |
| `rev` | Revision number of the signature. |

Other rule options can match payload content, TCP flags, application-layer fields, or flow state. Thresholding can help control how frequently alerts are generated.

Rules may also use network variables such as `$HOME_NET` and `$EXTERNAL_NET`. Their meaning depends on the sensor configuration; an external network is not necessarily the public internet.

The example above is a learning reference, not a required configuration step. An ICMP echo request is not inherently malicious.

## 7. Detection Engineering

Detection engineering is the process of developing, validating, maintaining, and improving detection logic to identify activity of security interest.

```text
Define detection objective
          |
          v
Develop detection logic
          |
          v
Test matching and non-matching traffic
          |
          v
Analyze results and tune the rule
          |
          v
Monitor and maintain the detection
```

| Testing method | Purpose |
| --- | --- |
| Positive test | Confirm that activity intended to match generates an alert. |
| Negative test | Confirm that selected activity outside the intended criteria does not generate an alert. |
| Tuning | Adjust detection criteria to improve usefulness and reduce unnecessary alerts. |

A successful positive test demonstrates that a detection works under the tested conditions. It does not establish that every variation of the activity will be detected.

## 8. Alert Analysis

IDS alerts contain information that helps analysts identify and investigate observed network activity.

| Alert field | Description |
| --- | --- |
| Timestamp | Time the event was recorded. |
| Source and destination | Network endpoints associated with the matching traffic. |
| Protocol and ports | Communication protocol and transport ports, where applicable. |
| Signature message | Description of the matched detection rule. |
| Signature ID | Identifier used to locate the corresponding rule. |
| Classification | Category of activity associated with the signature. |
| Severity or priority | Assigned importance of the detection, not necessarily the actual impact of the event. |
| Action | How the detection engine handled matching traffic; this is not necessarily the firewall's decision. |

For example, an alert indicating a possible network scan may identify reconnaissance activity. It does not independently establish malicious intent, a successful connection, or a compromised system.

Analysts can correlate IDS alerts with firewall logs, packet captures, and endpoint logs to develop a more complete understanding of the activity.

## 9. Detection Limitations

| Limitation | Description |
| --- | --- |
| False positive | Benign activity incorrectly identified as a threat. |
| False negative | Relevant malicious activity that is not detected. |
| Incomplete visibility | Traffic that does not traverse the monitoring interface cannot be inspected there. |
| Encryption | Encrypted payloads can limit content-based detection. |
| Rule coverage | Disabled, outdated, or missing signatures can leave gaps in detection. |
| Packet loss | Sensor overload or capture limitations can prevent complete inspection. |
| Alert fatigue | Excessive low-value alerts can make meaningful events harder to identify. |

Detection rules should be maintained and evaluated against the environment they protect. An absence of alerts does not prove an absence of malicious activity, and an alert alone does not prove a compromise.

---

Return to the [Exercise 07 README](README.md) or continue to the [walkthrough](07-walkthrough.md).
