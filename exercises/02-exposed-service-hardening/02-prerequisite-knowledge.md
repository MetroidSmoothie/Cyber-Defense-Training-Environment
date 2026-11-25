# Prerequisite Knowledge — Exercise 02: Exposed Service Hardening

This exercise introduces the concepts behind reducing the attack surface of a Linux server by controlling how network services listen for incoming connections. You do not need to memorize these concepts before beginning the exercise, but understanding them will help you interpret the results of your testing.

If you are unfamiliar with IP addressing, ports, network segmentation, or firewall filtering, review the [Exercise 01 Prerequisite Knowledge](../01-stateful-firewall-segmentation/prerequisite-knowledge.md) first.

## 1. Attack Surface

An attack surface is the collection of points through which an attacker could interact with a system and potentially compromise it. Network services, applications, user accounts, and management interfaces are all examples of potential exposure.

An attack vector is a method or path used to attempt a compromise. For example, exploiting a vulnerable web application or attempting to authenticate using stolen SSH credentials are attack vectors.

A listening service is not necessarily vulnerable. However, every accessible service introduces functionality that must be configured, maintained, and secured.

## 2. System Hardening

System hardening is the process of reducing unnecessary exposure and strengthening the security configuration of a system.

Common hardening measures include:

- Disabling unnecessary services.
- Restricting which network addresses services listen on.
- Applying security updates.
- Restricting administrative access.
- Configuring secure authentication and permissions.

The **principle of least functionality** states that a system should provide only the functionality necessary to fulfill its intended purpose. The **principle of least privilege** states that users, processes, and other entities should receive only the permissions necessary to perform their responsibilities.

Hardening should preserve required functionality. Disabling every service on a server may reduce exposure, but it also defeats the purpose of the server.

## 3. Network Services and Listening Sockets

A network service provides functionality to other systems over a network. In our lab, Ubuntu provides two services:

| Service | Protocol and port | Purpose |
|---|---|---|
| SSH (OpenSSH) | TCP/22 | Remote administration |
| HTTP (Apache) | TCP/80 | Web content delivery |

A server uses a listening socket to wait for incoming connections. A socket is an operating system abstraction representing a communication endpoint. A TCP listening socket is associated with a local IP address and port.

A service can be running without listening on a network port. Similarly, a listening socket can exist before the application responsible for handling a connection has started, as we will see with socket activation.

A listening port does not necessarily mean the service is reachable from every network. Routing, firewalls, and the address to which the socket is bound all influence accessibility.

## 4. Interface and Address Binding

Binding determines the local address and port on which a service accepts connections. Although commonly referred to as interface binding, binding to an IP address does not necessarily restrict traffic to a particular physical interface.

| Listening address | Meaning |
|---|---|
| `0.0.0.0:22` | Listens on all local IPv4 addresses on TCP port 22 |
| `[::]:22` | Listens on all local IPv6 addresses on TCP port 22; IPv4 behavior depends on socket configuration |
| `127.0.0.1:22` | Listens on the IPv4 loopback address only |
| `[::1]:22` | Listens on the IPv6 loopback address only |
| `172.16.77.10:22` | Listens on the specified local IPv4 address on TCP port 22 |

Wildcard addresses (`0.0.0.0` and `::`) allow services to listen on multiple local addresses. Loopback addresses allow communication within the host itself and are not normally reachable from other hosts.

Consider a server with separate production and management IP addresses. Binding SSH to the management address can reduce exposure by preventing SSH from accepting connections addressed to the production IP. Additional firewall rules may still be necessary to restrict which clients can reach the management address.

In this exercise, we will restrict SSH on Ubuntu from wildcard listening addresses to its lab IPv4 address, `172.16.77.10`. Because Debian already connects to this address, its SSH connectivity should remain unchanged. The change reduces the scope of the listener, rather than blocking Debian.

## 5. Linux Service Management

A daemon is a process that typically runs in the background to provide a service. OpenSSH's server daemon is known as `sshd`.

Systemd is a system and service manager used by many Linux distributions. It manages units that represent resources and activities on a system.

| Unit type | Purpose |
|---|---|
| Service (`.service`) | Manages the execution of a process or service |
| Socket (`.socket`) | Manages a communication socket and can activate a service when needed |

### Socket Activation

With conventional service operation, an application starts and creates its own listening socket. With systemd socket activation, systemd can create the listening socket and start the associated service when a connection arrives.

```text
Incoming SSH connection
          |
          v
     ssh.socket
          |
          v
     ssh.service
```

This means `ssh.service` can be inactive while SSH is still available through an active `ssh.socket`. An inactive service unit alone is therefore not sufficient evidence that a network service is unavailable.

### Configuration Overrides

Systemd supports drop-in configuration files that modify existing unit settings without directly editing the distribution-provided unit file. This makes changes easier to maintain and reverse.

Some settings, including listening socket definitions, can contain multiple values. An override may need to clear existing values before defining replacements.

In our lab, systemd controls the SSH listening socket. Changing the OpenSSH daemon configuration alone may therefore not change the actual listening addresses.

## 6. Port Scanning and Service Exposure

Port scanning is a technique used to determine which network ports are accessible from a particular source. Administrators can use it to identify exposed services and validate security controls.

| Port state | Meaning |
|---|---|
| Open | A service is accepting connections on the port |
| Closed | The port is reachable, but no service is accepting connections, or the connection is actively refused |
| Filtered | The scanner cannot determine whether the port is open because probes or responses are being filtered or otherwise prevented |

These states describe what a scanner observes from its particular network location. They are not necessarily a complete representation of the target system.

For example, Kali may observe TCP/22 as filtered because OPNsense blocks its traffic, while Debian observes the same port as open because it resides on Ubuntu's local network.

A port scan should be performed only against systems you own or are authorized to test.

## 7. Defense in Depth

Defense in depth is the use of multiple complementary security controls to reduce reliance on any single control.

| Security control | Function |
|---|---|
| Network firewall | Restricts traffic crossing network boundaries |
| Host firewall | Restricts traffic at the individual system |
| Service binding | Restricts the local addresses on which a service accepts connections |
| Authentication | Verifies the identity of users attempting to access a service |
| Patching | Addresses known software vulnerabilities |

These controls serve different purposes. Restricting a service's listening address does not replace firewall filtering, authentication, or patching.

In our lab, OPNsense controls traffic between the external and internal networks, while Ubuntu's service configuration controls how SSH listens on the host itself.

## 8. Configuration Validation

A configuration file describes intended behavior. The effective configuration and runtime behavior determine what the system is actually doing.

When validating a security configuration, consider three perspectives:

| Perspective | Question |
|---|---|
| Configuration | What is the system configured to do? |
| Runtime state | What is the system actually doing? |
| External observation | What can another system access? |

A configuration change may require a reload or restart before it takes effect. Multiple components can also influence the same service, as demonstrated by systemd socket activation and OpenSSH.

You should verify that the intended security restriction is effective while ensuring legitimate functionality remains available. In this exercise, an unchanged scan against Ubuntu's permitted IP address is an expected result; inspecting the actual listening sockets provides the evidence that the binding has changed.

---

Return to the [Exercise 02 README](README.md) or continue to the [walkthrough](02-walkthrough.md).
