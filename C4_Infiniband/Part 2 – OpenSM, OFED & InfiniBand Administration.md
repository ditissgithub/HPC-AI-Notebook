# Part 2 – OpenSM, OFED & InfiniBand Administration

- [4.8 OpenSM (Subnet Manager)](#48-opensm-subnet-manager)
- [4.9 Mellanox OFED](#49-mellanox-ofed)
- [4.10 InfiniBand Interfaces](#410-infiniband-interfaces)
- [4.11 Essential InfiniBand Commands](#411-essential-infiniband-commands)
- [4.12 Performance Monitoring](#412-performance-monitoring)
- [4.13 Fabric Verification](#413-fabric-verification)
- [Key Takeaways](#key-takeaways)

---

# 4.8 OpenSM (Subnet Manager)

An InfiniBand fabric requires a **Subnet Manager (SM)** to initialize and manage the network.

The most commonly used Subnet Manager in Linux HPC clusters is:

```
OpenSM
```

Without OpenSM (or another Subnet Manager), compute nodes cannot communicate over the InfiniBand fabric.

---

## Responsibilities of OpenSM

- Discover InfiniBand devices
- Assign Local Identifiers (LIDs)
- Configure routing tables
- Monitor fabric topology
- Detect topology changes

```
            OpenSM
               │
    -------------------------
    │           │          │
 Compute01  Compute02  Storage
```

---

## OpenSM Service

Check status

```bash
systemctl status opensm
```

Start service

```bash
systemctl start opensm
```

Enable at boot

```bash
systemctl enable opensm
```

View logs

```bash
journalctl -u opensm
```

---

## Production Note

Large HPC clusters generally run **only one active Subnet Manager** with one or more standby Subnet Managers for high availability.

---

# 4.9 Mellanox OFED

**Mellanox OFED (OpenFabrics Enterprise Distribution)** provides the drivers and utilities required for InfiniBand devices.

It includes:

- Kernel Drivers
- RDMA Libraries
- Performance Tools
- Diagnostic Utilities
- Firmware Utilities

---

## Verify Driver

```bash
ofed_info
```

Example

```
MLNX_OFED_LINUX-24.x
```

---

## Verify Loaded Modules

```bash
lsmod | grep mlx
```

Typical output

```
mlx5_core
mlx5_ib
ib_core
rdma_cm
```

---

## Check PCI Device

```bash
lspci | grep Mellanox
```

Example

```
Mellanox ConnectX-6
```

---

## View Driver Information

```bash
ethtool -i ib0
```

---

## HPC Perspective

A mismatched firmware version, kernel module, or OFED package is one of the most common causes of InfiniBand issues after operating system upgrades.

---

# 4.10 InfiniBand Interfaces

Linux identifies InfiniBand interfaces similarly to Ethernet.

Common interface names

```
ib0

ib1
```

---

## Display Interface

```bash
ip addr show ib0
```

or

```bash
ip a
```

---

## Display Link State

```bash
ip link show ib0
```

---

## Bring Interface Up

```bash
ip link set ib0 up
```

---

## Interface States

| State | Meaning |
|--------|----------|
| Down | Interface disabled |
| Initializing | Waiting for Subnet Manager |
| Active | Ready for communication |

---

## Production Note

If an InfiniBand interface remains in the **Initializing** state, the first thing to verify is whether the Subnet Manager (OpenSM) is running.

---

# 4.11 Essential InfiniBand Commands

The following commands are frequently used by HPC administrators.

---

## ibstat

Displays HCA status.

```bash
ibstat
```

Shows

- Port State
- Physical State
- Link Speed
- Firmware Version

---

## ibv_devices

List RDMA-capable devices.

```bash
ibv_devices
```

---

## ibdev2netdev

Maps InfiniBand devices to Linux interfaces.

```bash
ibdev2netdev
```

Example

```
mlx5_0

↓

ib0
```

---

## ibhosts

Displays discovered hosts.

```bash
ibhosts
```

---

## ibswitches

Lists InfiniBand switches.

```bash
ibswitches
```

---

## ibnetdiscover

Discovers the entire fabric topology.

```bash
ibnetdiscover
```

Useful for:

- Fabric validation
- Topology verification
- Troubleshooting

---

## iblinkinfo

Displays link information.

```bash
iblinkinfo
```

---

## ibping

Tests communication between InfiniBand nodes.

```bash
ibping
```

---

## ib_write_bw

Measures RDMA write bandwidth.

```bash
ib_write_bw
```

---

## ib_read_bw

Measures RDMA read bandwidth.

```bash
ib_read_bw
```

---

## ib_send_bw

Measures send bandwidth.

```bash
ib_send_bw
```

---

# 4.12 Performance Monitoring

Monitoring ensures the InfiniBand fabric operates efficiently.

---

## Link Status

```bash
ibstat
```

---

## Interface Counters

```bash
cat /sys/class/infiniband/*/ports/1/counters/*
```

---

## Port Errors

```bash
perfquery
```

Useful for identifying:

- Symbol Errors
- Link Down Events
- Packet Errors
- Receive Errors

---

## Linux Interface Statistics

```bash
ip -s link show ib0
```

---

## Hardware Information

```bash
ibv_devinfo
```

Displays

- Device Capabilities
- Firmware Version
- Port Information
- Maximum MTU

---

# 4.13 Fabric Verification

After deploying an HPC cluster, verify that the InfiniBand fabric is healthy.

---

## Checklist

✔ OpenSM is running

```bash
systemctl status opensm
```

---

✔ HCA detected

```bash
ibv_devices
```

---

✔ Interface Active

```bash
ibstat
```

---

✔ Topology discovered

```bash
ibnetdiscover
```

---

✔ Switch visible

```bash
ibswitches
```

---

✔ Links healthy

```bash
iblinkinfo
```

---

✔ RDMA bandwidth

```bash
ib_write_bw
```

---

## Production Workflow

```
Server Boot

↓

OFED Drivers Loaded

↓

OpenSM Running

↓

HCAs Discovered

↓

LIDs Assigned

↓

Ports Become Active

↓

Fabric Verified

↓

MPI Jobs Start
```

---

# Key Takeaways

- OpenSM is responsible for initializing and managing the InfiniBand fabric.
- Mellanox OFED provides drivers, RDMA libraries, and administration tools.
- Linux represents InfiniBand devices as interfaces such as `ib0`.
- Commands like `ibstat`, `ibnetdiscover`, and `iblinkinfo` are essential for day-to-day administration.
- Always verify the health of the fabric before running HPC workloads.
- A systematic verification process helps identify issues before they affect users.

---

## Next Part

**Chapter 4 – Part 3**

Topics:

- GPUDirect RDMA
- Common InfiniBand Problems
- Production Troubleshooting
- Best Practices
- Interview Questions
- InfiniBand Command Cheat Sheet
- Chapter Summary
