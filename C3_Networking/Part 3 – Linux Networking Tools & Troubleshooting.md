## Part 3 – Linux Networking Tools & Troubleshooting

- [3.19 Linux Networking Commands](#319-linux-networking-commands)
- [3.20 NetworkManager & nmcli](#320-networkmanager--nmcli)
- [3.21 Firewall Management](#321-firewall-management)
- [3.22 Network Troubleshooting Workflow](#322-network-troubleshooting-workflow)
- [3.23 Production Networking Scenarios](#323-production-networking-scenarios)
- [3.24 Networking Best Practices](#324-networking-best-practices)
- [3.25 Networking Interview Questions](#325-networking-interview-questions)
- [3.26 Chapter Summary](#326-chapter-summary)

---

# 3.19 Linux Networking Commands

Linux provides powerful command-line tools for monitoring, configuring, and troubleshooting networks.

---

## 1. ip Command

The **ip** command is the modern replacement for `ifconfig`, `route`, and `arp`.

View interfaces

```bash
ip addr
```

Short form

```bash
ip a
```

View interface status

```bash
ip link
```

View routing table

```bash
ip route
```

View neighbor (ARP) table

```bash
ip neigh
```

Bring interface up

```bash
ip link set ens1f0 up
```

Bring interface down

```bash
ip link set ens1f0 down
```

Assign an IP address

```bash
ip addr add 192.168.1.100/24 dev ens1f0
```

---

## 2. ss Command

`ss` displays socket and network connection information.

Display all connections

```bash
ss
```

Listening TCP ports

```bash
ss -lnt
```

Listening UDP ports

```bash
ss -lnu
```

Listening services with process names

```bash
ss -tulpn
```

Established connections

```bash
ss -tan
```

---

## 3. ping

Checks basic IP connectivity.

```bash
ping compute01
```

Limit packets

```bash
ping -c 4 compute01
```

---

## 4. traceroute

Shows the path packets take.

```bash
traceroute compute01
```

Alternative

```bash
tracepath compute01
```

---

## 5. tcpdump

Captures network packets for analysis.

Capture on interface

```bash
tcpdump -i ens1f0
```

Capture ICMP traffic

```bash
tcpdump -i ens1f0 icmp
```

Capture SSH traffic

```bash
tcpdump port 22
```

Write capture to file

```bash
tcpdump -w capture.pcap
```

---

## 6. ethtool

Displays and configures NIC information.

View NIC details

```bash
ethtool ens1f0
```

Driver information

```bash
ethtool -i ens1f0
```

Statistics

```bash
ethtool -S ens1f0
```

---

## 7. Legacy Commands

Although deprecated, these commands may still appear in older systems.

```bash
ifconfig

netstat

route

arp
```

Modern replacements are:

| Legacy | Modern |
|---------|--------|
| ifconfig | ip |
| route | ip route |
| arp | ip neigh |
| netstat | ss |

---

# 3.20 NetworkManager & nmcli

NetworkManager simplifies network configuration on Linux.

Service

```bash
systemctl status NetworkManager
```

---

## nmcli

List devices

```bash
nmcli device
```

Connection status

```bash
nmcli connection show
```

Activate connection

```bash
nmcli connection up ens1f0
```

Deactivate connection

```bash
nmcli connection down ens1f0
```

Display IP configuration

```bash
nmcli device show
```

---

## HPC Note

Many production HPC clusters disable NetworkManager on compute nodes and use static configurations managed by provisioning tools such as xCAT or automation platforms like Ansible.

---

# 3.21 Firewall Management

Enterprise Linux systems commonly use **firewalld**.

---

## Service Status

```bash
systemctl status firewalld
```

---

## Basic Commands

View status

```bash
firewall-cmd --state
```

List active zones

```bash
firewall-cmd --get-active-zones
```

List open services

```bash
firewall-cmd --list-services
```

List open ports

```bash
firewall-cmd --list-ports
```

Reload configuration

```bash
firewall-cmd --reload
```

---

## nftables

Modern Linux firewall framework.

View rules

```bash
nft list ruleset
```

---

## Production Note

A blocked port can prevent communication between cluster components such as Slurm controllers, LDAP servers, monitoring systems, or management nodes. Always verify firewall rules when troubleshooting connectivity issues.

---

# 3.22 Network Troubleshooting Workflow

A structured approach reduces troubleshooting time.

```
Problem Reported

↓

Check Interface

↓

Check IP Address

↓

Check Routing

↓

Check DNS

↓

Check Firewall

↓

Capture Traffic

↓

Identify Root Cause
```

---

## Step 1 – Interface

```bash
ip link

ip addr
```

---

## Step 2 – Connectivity

```bash
ping
```

---

## Step 3 – Routing

```bash
ip route
```

---

## Step 4 – DNS

```bash
host

dig

nslookup
```

---

## Step 5 – Open Ports

```bash
ss -tulpn
```

---

## Step 6 – Packet Capture

```bash
tcpdump
```

---

## Step 7 – Hardware

```bash
ethtool
```

---

# 3.23 Production Networking Scenarios

## Scenario 1 – Cannot Reach Another Node

Symptoms

- Ping fails
- SSH unavailable

Check

```bash
ip addr

ip route

ping

ss
```

Possible causes

- Interface down
- Wrong IP
- Incorrect route
- Firewall
- Switch issue

---

## Scenario 2 – Hostname Not Resolving

Symptoms

```
ssh compute01

↓

Name or service not known
```

Check

```bash
host compute01

dig compute01

cat /etc/resolv.conf
```

Possible causes

- DNS server unavailable
- Incorrect DNS configuration
- Missing DNS record

---

## Scenario 3 – Slow Network Performance

Check

```bash
ethtool

ip -s link

ss

tcpdump
```

Possible causes

- Duplex mismatch
- Packet loss
- MTU mismatch
- Congestion
- Faulty cable or switch port

---

## Scenario 4 – Service Not Accessible

Example

SSH service running but users cannot connect.

Check

```bash
ss -lnt

firewall-cmd --list-ports

systemctl status sshd
```

---

# 3.24 Networking Best Practices

- Use static IP addresses for infrastructure nodes.
- Separate management, storage, and user traffic.
- Document IP addressing schemes.
- Use meaningful hostnames.
- Keep DNS records consistent.
- Monitor interface errors and packet drops.
- Standardize MTU settings across the network.
- Use redundancy for critical links.
- Test connectivity after network changes.
- Maintain accurate network diagrams.

---

# 3.25 Networking Interview Questions

## Basic

1. What is the OSI model?
2. Explain the TCP/IP model.
3. Difference between MAC and IP address.
4. What is a subnet mask?
5. What is CIDR?

---

## Intermediate

1. Explain ARP.
2. What is a VLAN?
3. What is network bonding?
4. Difference between a switch and a router.
5. Explain MTU and Jumbo Frames.
6. What is the purpose of DNS?
7. How does DHCP work?

---

## Advanced

1. How would you troubleshoot a node that cannot communicate with the management network?
2. A hostname resolves incorrectly. How would you investigate?
3. What commands would you use to diagnose packet loss?
4. How would you identify whether a firewall is blocking a service?
5. What information can you obtain from `tcpdump`?

---

# 3.26 Chapter Summary

This chapter introduced the networking concepts required for administering Linux-based HPC and AI infrastructure.

Topics covered:

- Networking fundamentals
- OSI and TCP/IP models
- IPv4, subnetting, and CIDR
- Ethernet, switching, and ARP
- VLANs and network bonding
- Routing, gateways, DNS, and DHCP
- MTU and Jumbo Frames
- Linux networking commands
- NetworkManager and firewalld
- Structured troubleshooting
- Production networking scenarios
- Interview preparation

Networking forms the communication layer for nearly every HPC service. A solid understanding of these fundamentals is essential before moving on to high-performance interconnects such as **InfiniBand**, which is covered in the next chapter.

---

# Networking Quick Reference

## Interface

```bash
ip addr
ip link
```

## Routing

```bash
ip route
```

## Connectivity

```bash
ping
tracepath
```

## DNS

```bash
host
dig
nslookup
```

## Ports

```bash
ss -tulpn
```

## Packet Capture

```bash
tcpdump
```

## NIC Information

```bash
ethtool
```

## Firewall

```bash
firewall-cmd --list-all
nft list ruleset
```

---

# Next Chapter

**Chapter 4 – InfiniBand**

Topics include:

- InfiniBand Architecture
- RDMA Fundamentals
- OpenSM
- Mellanox OFED
- GPUDirect RDMA
- Performance Monitoring
- Troubleshooting
- Production Commands
- Interview Questions
