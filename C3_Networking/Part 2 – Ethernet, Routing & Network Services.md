## Part 2 – Ethernet, Routing & Network Services

- [3.10 Ethernet Fundamentals](#310-ethernet-fundamentals)
- [3.11 Switching](#311-switching)
- [3.12 ARP (Address Resolution Protocol)](#312-arp-address-resolution-protocol)
- [3.13 VLAN (Virtual LAN)](#313-vlan-virtual-lan)
- [3.14 Network Bonding](#314-network-bonding)
- [3.15 Routing & Gateway](#315-routing--gateway)
- [3.16 DNS (Domain Name System)](#316-dns-domain-name-system)
- [3.17 DHCP (Dynamic Host Configuration Protocol)](#317-dhcp-dynamic-host-configuration-protocol)
- [3.18 MTU & Jumbo Frames](#318-mtu--jumbo-frames)
- [Key Takeaways](#key-takeaways)

---

# 3.10 Ethernet Fundamentals

Ethernet is the most widely used **Layer 2 (Data Link Layer)** technology for connecting devices in a Local Area Network (LAN).

Every Ethernet device is identified by a **MAC (Media Access Control) Address**.

```
+-----------------------+
| Ethernet Frame        |
+-----------------------+
| Destination MAC       |
| Source MAC            |
| EtherType             |
| Payload               |
| Frame Check Sequence  |
+-----------------------+
```

---

## MAC Address

A MAC address is a **48-bit unique hardware identifier** assigned to a Network Interface Card (NIC).

Example

```
00:1A:4B:7F:2D:91
```

View MAC Address

```bash
ip link
```

or

```bash
ip addr
```

---

## Ethernet Speed

Common Ethernet speeds used in HPC environments:

| Speed | Typical Usage |
|--------|---------------|
| 1 Gbps | Management Network |
| 10 Gbps | Storage / Login Nodes |
| 25 Gbps | AI Clusters |
| 40 Gbps | HPC Clusters |
| 100 Gbps | Modern HPC & AI |
| 200/400 Gbps | Large AI Supercomputers |

---

# 3.11 Switching

A **network switch** connects multiple devices within a LAN and forwards Ethernet frames based on MAC addresses.

```
         Switch
      ┌────┼────┐
      │    │    │
      ▼    ▼    ▼
   Node1 Node2 Storage
```

Unlike a hub, a switch sends traffic **only to the destination port**, reducing unnecessary network traffic.

---

## MAC Address Table

Switches maintain a MAC Address Table to learn which MAC address is connected to which port.

```
MAC Address

↓

Switch Table

↓

Port Number
```

---

## HPC Perspective

Typical HPC clusters use separate switches for:

- Management Ethernet
- Storage Network
- High-Speed InfiniBand Fabric

This separation improves performance and fault isolation.

---

# 3.12 ARP (Address Resolution Protocol)

ARP maps an **IP Address** to a **MAC Address**.

Before sending data on an Ethernet network, Linux must know the destination MAC address.

```
Application

↓

Destination IP

↓

ARP Lookup

↓

Destination MAC

↓

Ethernet Frame
```

---

## ARP Request

```
Who has 192.168.1.10 ?

↓

Broadcast

↓

Reply

↓

MAC Address
```

---

## View ARP Cache

```bash
ip neigh
```

Legacy command

```bash
arp -n
```

---

## Flush ARP Cache

```bash
ip neigh flush all
```

---

## Production Note

If a compute node cannot communicate with the management node despite correct IP settings, check the ARP table. Incorrect or stale ARP entries can prevent communication, especially after replacing NICs or changing network configurations.

---

# 3.13 VLAN (Virtual LAN)

A VLAN logically separates one physical network into multiple isolated broadcast domains.

Example

```
Physical Switch

│

├── VLAN 10 → Management

├── VLAN 20 → Storage

└── VLAN 30 → Users
```

---

## Benefits

- Improved security
- Reduced broadcast traffic
- Better traffic isolation
- Simplified network management

---

## Linux VLAN Interface

Example

```bash
ip link add link ens1f0 name ens1f0.100 type vlan id 100
```

Bring interface up

```bash
ip link set ens1f0.100 up
```

---

## View VLAN Interfaces

```bash
ip -d link
```

---

## HPC Example

Large clusters often use separate VLANs for:

- xCAT provisioning
- User login
- Storage traffic
- Cluster management

---

# 3.14 Network Bonding

Bonding combines multiple physical NICs into one logical interface.

Purpose:

- High Availability
- Increased Bandwidth
- Fault Tolerance

```
NIC1

      \
       \
        Bond0

       /

NIC2
```

---

## Common Bonding Modes

| Mode | Description |
|------|-------------|
| Mode 0 | Round Robin |
| Mode 1 | Active Backup |
| Mode 4 | 802.3ad (LACP) |
| Mode 6 | Adaptive Load Balancing |

---

## View Bond Information

```bash
cat /proc/net/bonding/bond0
```

---

## HPC Perspective

Bonding is commonly used for:

- Management networks
- Storage connectivity
- High availability of critical services

High-speed MPI traffic generally uses dedicated InfiniBand interfaces rather than Ethernet bonding.

---

# 3.15 Routing & Gateway

A router connects different networks.

```
Network A

↓

Router

↓

Network B
```

---

## View Routing Table

```bash
ip route
```

Example

```
default via 192.168.1.1

192.168.1.0/24 dev ens1f0
```

---

## Default Gateway

The default gateway forwards packets to destinations outside the local subnet.

View gateway

```bash
ip route
```

Add route

```bash
ip route add 10.20.0.0/16 via 192.168.1.1
```

Delete route

```bash
ip route del 10.20.0.0/16
```

---

## HPC Perspective

Most compute nodes communicate internally without using the default gateway. Login nodes and management servers typically require external routing for software updates and user access.

---

# 3.16 DNS (Domain Name System)

DNS converts **hostnames** into **IP addresses**.

Instead of remembering:

```
192.168.10.15
```

Users access:

```
compute001.cluster.local
```

---

## DNS Resolution

```
Hostname

↓

DNS Server

↓

IP Address
```

---

## DNS Configuration

```
/etc/resolv.conf
```

---

## Useful Commands

Lookup hostname

```bash
host compute001
```

Detailed lookup

```bash
dig compute001
```

Test resolution

```bash
nslookup compute001
```

---

## HPC Perspective

xCAT, Slurm, LDAP, and monitoring systems rely heavily on consistent DNS records. Incorrect hostname resolution often causes cluster deployment and scheduling issues.

---

# 3.17 DHCP (Dynamic Host Configuration Protocol)

DHCP automatically assigns:

- IP Address
- Subnet Mask
- Gateway
- DNS Server

to client systems.

---

## DHCP Process

```
Discover

↓

Offer

↓

Request

↓

Acknowledge
```

(DORA Process)

---

## HPC Perspective

During provisioning, xCAT commonly provides DHCP services so newly powered compute nodes receive temporary network configuration before installation begins.

---

## Verify Lease

```bash
nmcli device show
```

or

```bash
ip addr
```

---

# 3.18 MTU & Jumbo Frames

**MTU (Maximum Transmission Unit)** defines the largest packet that can be transmitted without fragmentation.

Standard Ethernet MTU

```
1500 Bytes
```

Jumbo Frame

```
9000 Bytes
```

---

## View MTU

```bash
ip link
```

---

## Change MTU

```bash
ip link set dev ens1f0 mtu 9000
```

---

## Verify Path MTU

```bash
tracepath compute001
```

Test without fragmentation

```bash
ping -M do -s 8972 compute001
```

---

## Why Jumbo Frames?

Advantages:

- Lower CPU overhead
- Higher throughput
- Better storage performance
- Improved large data transfers

---

## HPC Note

Jumbo Frames are beneficial **only if every device along the communication path supports the same MTU**. A mismatch can cause packet loss or communication failures.

---

# Key Takeaways

- Ethernet provides Layer 2 communication using MAC addresses.
- Switches forward frames efficiently using MAC address tables.
- ARP maps IP addresses to MAC addresses on local networks.
- VLANs logically separate traffic over shared physical infrastructure.
- Network bonding improves availability and redundancy.
- Routers and gateways connect different networks.
- DNS translates hostnames into IP addresses.
- DHCP automates network configuration during system boot and provisioning.
- MTU configuration directly affects network efficiency and storage performance.

---

## Next Part

**03-Networking.md – Part 3**

Topics:

- Linux Networking Commands
- `ip`
- `ss`
- `tcpdump`
- `ethtool`
- `nmcli`
- NetworkManager
- firewalld
- nftables Basics
- Network Troubleshooting Commands
