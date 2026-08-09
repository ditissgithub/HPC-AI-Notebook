## Part 2 – Node Definitions, Networks, Images & Discovery

* [6.16 xCAT Node Definitions](#616-xcat-node-definitions)
* [6.17 Node Groups](#617-node-groups)
* [6.18 Node Attributes](#618-node-attributes)
* [6.19 xCAT Networks](#619-xcat-networks)
* [6.20 Network Interfaces](#620-network-interfaces)
* [6.21 DHCP Configuration](#621-dhcp-configuration)
* [6.22 DNS Configuration](#622-dns-configuration)
* [6.23 OS Images](#623-os-images)
* [6.24 Boot Configuration](#624-boot-configuration)
* [6.25 Node Discovery](#625-node-discovery)
* [6.26 Hardware Discovery](#626-hardware-discovery)
* [6.27 Provisioning Commands](#627-provisioning-commands)
* [6.28 Practical xCAT Example](#628-practical-xcat-example)
* [6.29 Production Deployment Workflow](#629-production-deployment-workflow)
* [6.30 Troubleshooting Checklist](#630-troubleshooting-checklist)
* [6.31 Interview Questions](#631-interview-questions)
* [6.32 Part 2 Summary](#632-part-2-summary)

---

# 6.16 xCAT Node Definitions

The xCAT database maintains information about cluster nodes through **node definitions**.

A node definition represents the configuration and identity of a physical or logical node.

Example:

```text
compute001
    │
    ├── hostname
    ├── IP address
    ├── MAC address
    ├── network
    ├── OS image
    ├── boot configuration
    └── hardware information
```

A node definition becomes the basis for many xCAT operations.

---

## View Node Definitions

```bash
lsdef
```

View one node:

```bash
lsdef compute001
```

More detailed output:

```bash
lsdef compute001 -i
```

---

## Why Node Definitions Matter

Suppose a new compute node has:

```text
Hostname: compute001
MAC:      00:11:22:33:44:55
IP:       10.10.10.101
```

If xCAT has an incorrect MAC address:

```text
xCAT Database
     │
     └── Wrong MAC
            │
            ▼
        DHCP fails
            │
            ▼
       Node not booting
```

Therefore, accurate node definitions are critical.

---

# 6.17 Node Groups

Managing hundreds of nodes individually is inefficient.

xCAT supports grouping nodes logically.

Example:

```text
compute
├── compute001
├── compute002
├── compute003
└── compute004
```

Another example:

```text
gpu
├── gpu001
├── gpu002
└── gpu003
```

You can organize nodes according to:

* Hardware type
* Function
* Location
* GPU type
* CPU architecture
* Cluster role

---

## Example Production Groups

```text
all
├── login
├── compute
├── gpu
├── storage
├── management
└── service
```

More specific grouping:

```text
gpu
├── a100
├── h100
└── mi250
```

This makes large-scale administration easier.

---

# 6.18 Node Attributes

xCAT node definitions can contain multiple attributes.

Common concepts include:

```text
Node
│
├── IP
├── MAC
├── Network
├── OS
├── Image
├── Boot method
├── Hardware information
└── Group membership
```

The exact attributes depend on the xCAT version and deployment.

---

## View Attributes

```bash
lsdef compute001 -i
```

Example conceptual output:

```text
Object name: compute001
    ip=10.10.10.101
    mac=00:11:22:33:44:55
    netboot=xnba
```

The actual attributes present in a production environment may differ.

---

## Modify Attributes

xCAT uses `chdef` to modify definitions.

Example:

```bash
chdef compute001 ip=10.10.10.101
```

Multiple attributes can be configured together according to the node's definition.

---

# 6.19 xCAT Networks

HPC clusters commonly have multiple networks.

For example:

```text
                 HPC Cluster
                     │
        ┌────────────┼─────────────┐
        ▼            ▼             ▼
   Management     Storage       Compute
    Network       Network       Network
        │            │             │
      xCAT         Lustre       Slurm/MPI
```

An advanced HPC cluster may additionally have:

```text
Management
Provisioning
Ethernet
InfiniBand
Storage
Out-of-band/BMC
```

---

## Why Separate Networks?

Network separation improves:

* Performance
* Security
* Fault isolation
* Management
* Troubleshooting

For example:

```text
Management Network
        │
        └── xCAT / SSH / Monitoring

InfiniBand
        │
        └── MPI / RDMA / GPU communication

Storage Network
        │
        └── Lustre traffic
```

---

# 6.20 Network Interfaces

A compute node may contain several interfaces.

Example:

```text
compute001

eth0
 └── Management

eth1
 └── Storage

ib0
 └── InfiniBand

eno2
 └── Additional Ethernet
```

xCAT needs accurate network information when provisioning and configuring nodes.

---

## Verify Interfaces on Linux

```bash
ip link
```

or:

```bash
ip addr
```

---

## Verify MAC Address

```bash
ip link show
```

Example:

```text
link/ether 00:11:22:33:44:55
```

The MAC address may be used for node identification during network boot.

---

# 6.21 DHCP Configuration

DHCP is a critical part of network provisioning.

Simplified flow:

```text
Compute Node
     │
     │ DHCP Request
     ▼
   DHCP
     │
     ├── IP Address
     ├── Network Information
     └── Boot Information
             │
             ▼
        Compute Node
```

---

## Generate DHCP Configuration

xCAT can generate DHCP configuration from its database.

Common command:

```bash
makedhcp
```

For a specific node:

```bash
makedhcp compute001
```

The exact syntax and options depend on the installed xCAT version.

---

## Verify DHCP

On the management/provisioning server, check the DHCP service according to the distribution and DHCP implementation.

For example:

```bash
systemctl status dhcpd
```

Check logs:

```bash
journalctl -u dhcpd
```

---

## Common DHCP Failure

### Symptom

Node powers on but never receives an IP address.

Check:

```text
Node MAC
   ↓
xCAT Database
   ↓
DHCP Configuration
   ↓
DHCP Service
   ↓
Network/VLAN
```

Potential causes:

* Incorrect MAC
* DHCP service stopped
* Incorrect subnet
* VLAN problem
* Switch configuration
* Duplicate IP
* Firewall/network filtering

---

# 6.22 DNS Configuration

DNS provides hostname resolution.

Example:

```text
compute001
    ↓
10.10.10.101
```

and:

```text
10.10.10.101
    ↓
compute001
```

Both forward and reverse resolution can be important in HPC environments.

---

## Generate DNS Configuration

xCAT can generate DNS-related configuration:

```bash
makedns
```

---

## Test Forward Lookup

```bash
getent hosts compute001
```

or:

```bash
dig compute001
```

---

## Test Reverse Lookup

```bash
dig -x 10.10.10.101
```

---

## Why DNS Matters

Incorrect DNS can cause problems with:

* Slurm
* MPI
* LDAP
* Lustre
* Monitoring
* SSH
* Cluster automation

A common HPC troubleshooting rule is:

> Always verify hostname resolution before investigating higher-level services.

---

# 6.23 OS Images

OS images are a central concept in automated provisioning.

Instead of manually installing Linux on every node:

```text
OS Image
   │
   ├── Node 1
   ├── Node 2
   ├── Node 3
   └── Node N
```

This creates a consistent baseline.

---

## Typical Image Contents

An HPC compute-node image may contain:

```text
Linux
├── Kernel
├── System Libraries
├── Network Configuration
├── Basic Packages
├── Monitoring Agent
├── Slurm Client
├── OFED / RDMA Stack
├── GPU Driver
└── Cluster Configuration
```

The exact contents depend on the provisioning strategy.

---

## Image Types

Common conceptual approaches include:

### Golden Image

A known-good baseline image.

```text
Golden Image
     ↓
All Compute Nodes
```

### Role-Based Images

Different node classes receive different images.

```text
CPU Image
GPU Image
High-Memory Image
Login Image
Storage Image
```

This is particularly useful in heterogeneous HPC clusters.

---

# 6.24 Boot Configuration

A compute node must know how it should boot.

Conceptually:

```text
Firmware
   ↓
Network Boot
   ↓
DHCP
   ↓
Bootloader
   ↓
Kernel / Initrd
   ↓
Installer or Runtime
```

The exact mechanism can vary with:

* BIOS vs UEFI
* PXE/iPXE
* Network boot architecture
* xCAT configuration
* OS version

---

## Verify Boot Mode

On Linux:

```bash
[ -d /sys/firmware/efi ] && echo UEFI || echo BIOS
```

---

## Boot Problems

If DHCP works but the node does not proceed to installation, investigate:

```text
DHCP
  ↓
Boot filename / parameters
  ↓
TFTP / HTTP / boot server
  ↓
Kernel
  ↓
Initrd
```

This distinction is important.

A node receiving an IP address does **not** necessarily mean that network provisioning is working completely.

---

# 6.25 Node Discovery

Node discovery is the process of identifying hardware and registering it within the cluster-management system.

Typical information includes:

* MAC address
* IP address
* BMC address
* CPU
* Memory
* PCIe devices
* GPU
* Network adapters

---

## Discovery Concept

```text
Unknown Server
      │
      ▼
Hardware Discovery
      │
      ├── MAC
      ├── CPU
      ├── Memory
      ├── GPU
      └── Network
      │
      ▼
xCAT Node Definition
```

---

## Why Discovery Matters

Consider a cluster containing:

```text
100 CPU nodes
50 GPU nodes
20 High-Memory nodes
```

Incorrect hardware classification can lead to:

* Wrong OS image
* Wrong Slurm configuration
* Incorrect GPU setup
* Incorrect network configuration
* Incorrect resource allocation

---

# 6.26 Hardware Discovery

Hardware discovery should validate what the node actually contains.

Useful Linux commands include:

## CPU

```bash
lscpu
```

## Memory

```bash
free -h
```

or:

```bash
lsmem
```

## PCI Devices

```bash
lspci
```

## GPU

```bash
lspci | grep -i nvidia
```

## Network

```bash
ip link
```

## InfiniBand

```bash
ibstat
```

---

## Hardware Validation Model

```text
Expected Hardware
        │
        ▼
Actual Hardware
        │
        ▼
Compare
        │
   ┌────┴────┐
   │         │
Match      Mismatch
   │         │
   ▼         ▼
Accept    Investigate
```

This is an important production practice.

---

# 6.27 Provisioning Commands

Important xCAT commands to remember:

| Command    | Purpose                                                        |
| ---------- | -------------------------------------------------------------- |
| `lsdef`    | Display definitions                                            |
| `mkdef`    | Create definitions                                             |
| `chdef`    | Modify definitions                                             |
| `rmdef`    | Remove definitions                                             |
| `makedhcp` | Generate DHCP configuration                                    |
| `makedns`  | Generate DNS configuration                                     |
| `nodeset`  | Set node boot/install state                                    |
| `rpower`   | Control node power                                             |
| `rinv`     | Query node inventory                                           |
| `rcons`    | Console access, depending on configuration                     |
| `rinstall` | Initiate installation, depending on xCAT version/configuration |

---

## `nodeset`

`nodeset` is important for controlling the desired network-boot/install state.

Conceptually:

```bash
nodeset compute001 osimage=<image>
```

The exact image specification depends on the xCAT configuration.

Then:

```bash
nodeset compute001
```

can be used to inspect the current setting.

---

## Power Management

Depending on the configured BMC/IPMI infrastructure:

```bash
rpower compute001 stat
```

Power on:

```bash
rpower compute001 on
```

Power off:

```bash
rpower compute001 off
```

Always verify the target node before issuing destructive power commands.

---

# 6.28 Practical xCAT Example

Assume a new node:

```text
Hostname : compute001
IP       : 10.10.10.101
MAC      : 00:11:22:33:44:55
Role     : CPU Compute
```

---

## Step 1 – Define the Node

Conceptually:

```bash
mkdef compute001
```

Then configure required attributes:

```bash
chdef compute001 ip=10.10.10.101
```

Additional node attributes would be configured according to the site's xCAT schema.

---

## Step 2 – Verify Definition

```bash
lsdef compute001 -i
```

---

## Step 3 – Generate DHCP

```bash
makedhcp
```

---

## Step 4 – Generate DNS

```bash
makedns
```

---

## Step 5 – Set Provisioning State

Example conceptual command:

```bash
nodeset compute001 osimage=<osimage>
```

---

## Step 6 – Power Node

```bash
rpower compute001 on
```

---

## Step 7 – Monitor Boot

Check:

```bash
rcons compute001
```

if console access is configured.

---

## Step 8 – Verify Node

After installation:

```bash
ping compute001
```

```bash
ssh compute001
```

Then validate:

```bash
hostname
ip addr
lscpu
free -h
```

---

# 6.29 Production Deployment Workflow

A production provisioning workflow should be controlled.

```text
Hardware Arrival
      │
      ▼
Hardware Inventory
      │
      ▼
MAC/BMC Registration
      │
      ▼
xCAT Node Definition
      │
      ▼
Network Validation
      │
      ▼
OS Image Selection
      │
      ▼
DHCP/DNS Generation
      │
      ▼
Network Boot
      │
      ▼
OS Installation
      │
      ▼
Post-Install Configuration
      │
      ▼
Hardware Validation
      │
      ▼
Slurm Configuration
      │
      ▼
Monitoring
      │
      ▼
Production
```

---

# 6.30 Troubleshooting Checklist

## Node Does Not Receive IP

Check:

```bash
lsdef compute001 -i
```

Verify:

```text
MAC
IP
Network
DHCP
VLAN
Switch
```

---

## Node Receives IP but Does Not Boot

Check:

```text
DHCP
↓
Boot configuration
↓
TFTP/HTTP
↓
Kernel
↓
Initrd
```

---

## OS Installation Fails

Check:

```text
OS image
Installation logs
Disk
Network
Repository
Kickstart/installation configuration
```

---

## Node Boots but Configuration Is Wrong

Check:

```text
Image
Post-install scripts
Ansible
Configuration management
Network configuration
```

---

## Node Is Healthy but Slurm Does Not See It

xCAT provisioning is not the same as Slurm registration.

Check:

```bash
systemctl status slurmd
```

Then:

```bash
scontrol show node compute001
```

The troubleshooting path becomes:

```text
xCAT
 ↓
Linux
 ↓
Network
 ↓
Slurm
```

---

# 6.31 Interview Questions

## Basic

1. What is an xCAT node definition?
2. Why are node groups useful?
3. What information should be associated with a compute node?
4. What is an OS image?
5. What is the role of DHCP during provisioning?

---

## Intermediate

1. Explain the difference between DHCP success and successful provisioning.
2. What is the purpose of `nodeset`?
3. What does `makedhcp` do?
4. What does `makedns` do?
5. How would you register a new compute node?
6. How would you validate the hardware after provisioning?

---

## Advanced

1. A node gets a DHCP address but does not load the boot image. Explain your troubleshooting approach.
2. One node receives another node's provisioning configuration. What would you investigate?
3. How would you safely provision 500 nodes?
4. How would you design separate images for CPU, GPU, and high-memory nodes?
5. How would you integrate xCAT provisioning with Ansible?
6. How would you prevent configuration drift after provisioning?

---

# 6.32 Part 2 Summary

The xCAT database is the foundation for automated cluster provisioning.

The key relationships are:

```text
Node Definition
      │
      ├── Identity
      ├── Network
      ├── Hardware
      ├── Image
      └── Boot State
             │
             ▼
          xCAT
             │
       ┌─────┴─────┐
       ▼           ▼
     DHCP         DNS
       │           │
       └─────┬─────┘
             ▼
        Network Boot
             │
             ▼
       OS Provisioning
             │
             ▼
       Post-Install
             │
             ▼
       Production Node
```

The most important operational principle is:

> **Provisioning starts with correct inventory and node definitions.**

If the database contains incorrect hardware, MAC, IP, network, or image information, every subsequent provisioning step can fail.

---

# Part 2 Learning Checklist

You should now be able to:

* Understand xCAT node definitions.
* Organize nodes into groups.
* Understand node attributes.
* Explain multi-network HPC environments.
* Understand DHCP-based provisioning.
* Understand DNS requirements.
* Explain OS images.
* Understand network boot stages.
* Explain node and hardware discovery.
* Use common xCAT provisioning commands.
* Describe a production node deployment workflow.
* Troubleshoot common provisioning failures.

---

# Next Part

**Chapter 6 – Part 3**

Topics:

* xCAT OS Image Management
* Stateless Provisioning
* Stateful Provisioning
* Kickstart / Installation Workflow
* Post-Install Configuration
* Software Deployment
* xCAT + Ansible Integration
* xCAT + Slurm Integration
* GPU Node Provisioning
* InfiniBand Node Configuration
* Real HPC Provisioning Example
* Production Best Practices
* Troubleshooting Scenarios
