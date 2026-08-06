## Part 1 – Networking Fundamentals
---

- [3.1 Why Networking Matters in HPC](#31-why-networking-matters-in-hpc)
- [3.2 What is a Computer Network?](#32-what-is-a-computer-network)
- [3.3 Types of Networks](#33-types-of-networks)
- [3.4 OSI Model](#34-osi-model)
- [3.5 TCP/IP Model](#35-tcpip-model)
- [3.6 Encapsulation & Decapsulation](#36-encapsulation--decapsulation)
- [3.7 IPv4 Addressing](#37-ipv4-addressing)
- [3.8 Subnet Mask & CIDR](#38-subnet-mask--cidr)
- [3.9 Linux Network Interfaces](#39-linux-network-interfaces)
- [Key Takeaways](#key-takeaways)

---

# 3.1 Why Networking Matters in HPC

Networking is the **backbone of every HPC and AI cluster**.

Unlike a standalone Linux server, an HPC cluster consists of multiple nodes that must communicate efficiently to work as a single system.

Typical communication includes:

- User login to Login Nodes
- xCAT provisioning Compute Nodes
- Slurm scheduling jobs
- LDAP authentication
- Lustre storage access
- MPI communication between compute nodes
- GPU communication (RDMA / InfiniBand)

```
             Users
               │
               ▼
        Login Node
               │
      ┌────────┴────────┐
      ▼                 ▼
  Compute 01       Compute 02
      │                 │
      └────────┬────────┘
               ▼
         Shared Storage
```

Without a properly configured network, the cluster cannot function.

---

# 3.2 What is a Computer Network?

A **computer network** is a collection of devices connected together to exchange data.

Examples:

- Two laptops connected via Ethernet
- Office LAN
- Data Center
- HPC Cluster
- AI Supercomputer

Every device connected to the network is called a **Node**.

Examples:

- Laptop
- Server
- Switch
- Router
- Storage
- GPU Server

---

## Basic Communication

```
Node A

   │

Ethernet Cable

   │

Switch

   │

Node B
```

Data travels in the form of **packets**.

---

# 3.3 Types of Networks

| Type | Description |
|-------|-------------|
| LAN | Local Area Network |
| WAN | Wide Area Network |
| MAN | Metropolitan Area Network |
| SAN | Storage Area Network |
| InfiniBand Fabric | High-speed HPC interconnect |

---

## HPC Example

```
Management Network

192.168.x.x

↓

Provisioning

↓

Ethernet

--------------------

High-Speed Network

InfiniBand

↓

MPI

↓

GPU Communication

--------------------

Storage Network

Lustre
```

Most HPC clusters use **multiple dedicated networks** instead of a single shared network.

---

# 3.4 OSI Model

The **OSI (Open Systems Interconnection)** model divides network communication into seven layers.

```
+---------------------------+
| 7 Application             |
+---------------------------+
| 6 Presentation            |
+---------------------------+
| 5 Session                 |
+---------------------------+
| 4 Transport               |
+---------------------------+
| 3 Network                 |
+---------------------------+
| 2 Data Link               |
+---------------------------+
| 1 Physical                |
+---------------------------+
```

---

## Layer Overview

| Layer | Examples |
|--------|----------|
| Application | HTTP, SSH, DNS |
| Presentation | Encryption, Compression |
| Session | Session Management |
| Transport | TCP, UDP |
| Network | IP |
| Data Link | Ethernet, MAC |
| Physical | Cable, Fiber |

---

## Why Learn OSI?

Although Linux primarily follows the TCP/IP model, the OSI model is widely used for:

- Troubleshooting
- Network design
- Interview questions
- Understanding protocol responsibilities

---

# 3.5 TCP/IP Model

The TCP/IP model is the practical networking model used on Linux systems.

```
+---------------------------+
| Application               |
+---------------------------+
| Transport                 |
+---------------------------+
| Internet                  |
+---------------------------+
| Network Access            |
+---------------------------+
```

---

## Layer Responsibilities

| Layer | Protocols |
|--------|-----------|
| Application | SSH, HTTP, DNS, LDAP |
| Transport | TCP, UDP |
| Internet | IPv4, IPv6, ICMP |
| Network Access | Ethernet, Wi-Fi, InfiniBand |

---

## OSI vs TCP/IP

| OSI | TCP/IP |
|-----|---------|
| Application | Application |
| Presentation | Application |
| Session | Application |
| Transport | Transport |
| Network | Internet |
| Data Link | Network Access |
| Physical | Network Access |

---

# 3.6 Encapsulation & Decapsulation

When data is transmitted, each layer adds its own header.

```
Application Data

↓

TCP Header

↓

IP Header

↓

Ethernet Header

↓

Frame
```

At the receiving system:

```
Frame

↓

Ethernet

↓

IP

↓

TCP

↓

Application
```

This process is called:

- **Encapsulation** (Sending)
- **Decapsulation** (Receiving)

---

# 3.7 IPv4 Addressing

Every device on a network requires a unique IP address.

Example

```
192.168.10.15
```

IPv4 consists of **32 bits**.

```
192 .168 .10 .15

8     8    8   8
```

---

## Public vs Private IP

Private address ranges:

| Network | Range |
|----------|-------|
| Class A | 10.0.0.0/8 |
| Class B | 172.16.0.0 – 172.31.255.255 |
| Class C | 192.168.0.0/16 |

Most HPC clusters use **private IP addresses** for internal communication.

---

# 3.8 Subnet Mask & CIDR

A subnet mask separates the **network portion** from the **host portion** of an IP address.

Example

```
IP Address

192.168.1.25

Subnet Mask

255.255.255.0
```

CIDR notation

```
192.168.1.25/24
```

Common CIDR Prefixes

| CIDR | Subnet Mask |
|------|-------------|
| /8 | 255.0.0.0 |
| /16 | 255.255.0.0 |
| /24 | 255.255.255.0 |
| /25 | 255.255.255.128 |
| /26 | 255.255.255.192 |

---

## Why Subnetting?

Subnetting helps:

- Reduce broadcast traffic
- Improve security
- Organize network segments
- Separate management, storage, and compute networks in HPC environments

---

# 3.9 Linux Network Interfaces

A **network interface** is the connection point between Linux and the network.

Examples

```
lo

ens1f0

ens1f1

ib0

bond0
```

---

## View Interfaces

```bash
ip addr
```

Short form

```bash
ip a
```

Display interface status

```bash
ip link
```

---

## View Routing Table

```bash
ip route
```

---

## Check Connectivity

```bash
ping google.com
```

or

```bash
ping 192.168.1.10
```

---

## Display Socket Information

```bash
ss -tulpn
```

---

## Interface Statistics

```bash
ip -s link
```

---

## Production Note

In HPC clusters, administrators commonly work with multiple network interfaces on a single node, for example:

- `bond0` → Management Network
- `ib0` → InfiniBand Fabric
- `eno1` → BMC/IPMI or provisioning network

Understanding the purpose of each interface is essential when troubleshooting connectivity issues.

---

# Key Takeaways

- Networking is a fundamental component of every HPC cluster.
- HPC systems commonly use separate management, storage, and high-speed networks.
- The OSI model provides a conceptual understanding of network communication.
- Linux networking is based on the TCP/IP model.
- Encapsulation and decapsulation explain how data moves through the network stack.
- Every node requires a unique IP address and appropriate subnet configuration.
- Linux administrators use the `ip` command to inspect interfaces and routing.
- Correct interface configuration is critical for services such as xCAT, Slurm, Lustre, and SSH.

---

## Next Part

**03-Networking.md – Part 2**

Topics:

- Ethernet Fundamentals
- Switching
- ARP
- VLAN
- Bonding
- Routing
- Gateway
- DNS
- DHCP
- MTU & Jumbo Frames
