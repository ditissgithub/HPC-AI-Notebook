## Part 5 & Part 6 – Production Operations, Troubleshooting & Quick Revision

# Part 5 – Production xCAT Operations

## 6.69 xCAT in a Production HPC Workflow

In a production HPC cluster, xCAT is primarily responsible for **bare-metal provisioning and node lifecycle management**.

```text
Hardware
   ↓
BMC / Power
   ↓
xCAT
   ↓
DHCP / DNS / Network Boot
   ↓
Linux OS
   ↓
Ansible / Configuration
   ↓
Slurm
   ↓
Production
```

A practical division of responsibility is:

| Component    | Primary Responsibility              |
| ------------ | ----------------------------------- |
| xCAT         | Provisioning & node lifecycle       |
| Ansible      | Configuration & software deployment |
| Slurm        | Workload scheduling                 |
| LDAP/SSSD    | Authentication                      |
| Lustre       | Parallel storage                    |
| InfiniBand   | HPC communication                   |
| NVIDIA stack | GPU acceleration                    |
| Monitoring   | Health & observability              |

---

## 6.70 Adding a New Compute Node

Example:

```text
Node      : compute101
Role      : CPU Compute
Management: 10.10.10.101
```

### Workflow

```text
Hardware Inventory
       ↓
MAC/BMC Registration
       ↓
xCAT Node Definition
       ↓
DHCP/DNS
       ↓
OS Provisioning
       ↓
Post-Install Configuration
       ↓
Hardware Validation
       ↓
Slurm Validation
       ↓
Production
```

Useful commands:

```bash
lsdef compute101 -i
```

```bash
nodeset compute101
```

```bash
rpower compute101 on
```

After provisioning:

```bash
hostname
ip addr
lscpu
free -h
```

Then validate HPC components:

```bash
ibstat
systemctl status slurmd
```

---

# 6.71 GPU Node Provisioning

GPU nodes require additional validation.

```text
xCAT
 ↓
Linux
 ↓
NVIDIA Driver
 ↓
CUDA
 ↓
InfiniBand
 ↓
Lustre
 ↓
Slurm GRES
 ↓
GPU Workload
```

Important checks:

```bash
lspci | grep -i nvidia
```

```bash
nvidia-smi
```

```bash
nvidia-smi topo -m
```

```bash
ibstat
```

```bash
scontrol show node gpu001
```

The node should not be placed into production until the complete stack is validated.

---

# 6.72 xCAT + Ansible

A useful production architecture is:

```text
                xCAT
                  │
                  ▼
            Bare-Metal OS
                  │
                  ▼
               Ansible
                  │
       ┌──────────┼──────────┐
       ▼          ▼          ▼
     Slurm      LDAP       Monitoring
       │
       ▼
   Production
```

### xCAT

Handles:

* Hardware provisioning
* Network boot
* OS deployment
* Initial node lifecycle

### Ansible

Handles:

* Packages
* Configuration
* Services
* Slurm
* LDAP/SSSD
* Monitoring
* Security
* Cluster software

This separation makes the infrastructure easier to maintain.

---

# 6.73 xCAT + Slurm

Remember:

> **xCAT provisions the node; Slurm manages the workload.**

```text
             xCAT
               │
               ▼
          Compute Node
               │
               ▼
          Linux + slurmd
               │
               ▼
           slurmctld
               │
               ▼
          User Job
```

After provisioning:

```bash
systemctl status slurmd
```

Check from the controller:

```bash
scontrol show node compute101
```

A node can be correctly provisioned by xCAT but still be unavailable to Slurm.

---

# 6.74 Configuration Drift

Configuration drift occurs when a node gradually differs from the desired cluster configuration.

Example:

```text
Golden Configuration
        │
        ├── Node01 ✓
        ├── Node02 ✓
        ├── Node03 ✗
        └── Node04 ✓
```

Possible causes:

* Manual package installation
* Manual configuration changes
* Different driver versions
* Uncontrolled updates
* Failed automation

Best approach:

```text
xCAT
 ↓
Standard OS
 ↓
Ansible
 ↓
Validation
 ↓
Monitoring
```

Use version control for configuration wherever practical.

---

# 6.75 Backup & Recovery

Protect the xCAT control plane.

Important items include:

```text
xCAT Database
Configuration
OS Images
Kickstart Files
Network Configuration
Automation Code
Credentials / Secrets
```

Conceptually:

```text
Production xCAT
      │
      ├── Database Backup
      ├── Configuration Backup
      └── Image Backup
               │
               ▼
          Recovery System
```

A backup is useful only if it has been tested through a recovery procedure.

---

# Part 6 – Troubleshooting & Quick Revision

# 6.76 xCAT Troubleshooting Flow

Always troubleshoot from the bottom upward.

```text
Physical Hardware
       ↓
Power / BMC
       ↓
Switch / VLAN
       ↓
DHCP
       ↓
Network Boot
       ↓
OS Installation
       ↓
Linux
       ↓
HPC Software
       ↓
Slurm
```

---

## Problem 1 – Node Does Not Power On

Check:

```bash
rpower compute101 stat
```

Investigate:

```text
BMC
Power
Hardware
Network to BMC
```

---

## Problem 2 – Node Has No IP

Check:

```text
MAC Address
   ↓
xCAT Definition
   ↓
DHCP
   ↓
VLAN
   ↓
Switch Port
```

Useful:

```bash
lsdef compute101 -i
```

```bash
journalctl -u dhcpd
```

---

## Problem 3 – DHCP Works but OS Does Not Install

Check:

```text
DHCP
 ↓
Boot Configuration
 ↓
TFTP/HTTP
 ↓
Kernel
 ↓
Initrd
 ↓
Installer
```

The important distinction is:

> **DHCP success does not mean provisioning success.**

---

## Problem 4 – Node Boots but Linux Configuration Is Wrong

Check:

```text
OS Image
 ↓
Kickstart
 ↓
Post-Install
 ↓
Ansible
 ↓
Configuration
```

---

## Problem 5 – GPU Not Detected

First:

```bash
lspci | grep -i nvidia
```

If GPU is visible:

```bash
nvidia-smi
```

If `lspci` sees the GPU but `nvidia-smi` fails, investigate the NVIDIA driver/kernel stack.

---

## Problem 6 – InfiniBand Not Available

Check:

```bash
ibstat
```

```bash
rdma link
```

```bash
ibdev2netdev
```

Investigate:

```text
HCA
Driver
Firmware
Cable
Switch
Subnet Manager
```

---

## Problem 7 – Node Is DOWN in Slurm

Check:

```bash
systemctl status slurmd
```

Then:

```bash
journalctl -u slurmd
```

Controller:

```bash
scontrol show node compute101
```

Think in layers:

```text
xCAT
 ↓
Linux
 ↓
Network
 ↓
Authentication
 ↓
Slurm
```

---

# 6.77 Essential xCAT Commands

### Node Management

```bash
lsdef
lsdef compute001 -i
mkdef
chdef
rmdef
```

### Database

```bash
tabdump
tabdump nodes
tabdump networks
tabdump site
```

### Provisioning

```bash
nodeset compute001
makedhcp
makedns
```

### Power

```bash
rpower compute001 stat
rpower compute001 on
rpower compute001 off
```

### Inventory / Console

```bash
rinv compute001
rcons compute001
```

---

# 6.78 xCAT Quick Architecture

Remember this diagram:

```text
                 xCAT
                  │
        ┌─────────┼─────────┐
        ▼         ▼         ▼
    Database    DHCP       DNS
        │
        ▼
   OS / Images
        │
        ▼
 Network Boot
        │
        ▼
 Compute Nodes
        │
        ▼
     Linux
        │
   ┌────┼────┐
   ▼    ▼    ▼
Slurm IB   Lustre
   │
   ▼
HPC Jobs
```

---

# 6.79 xCAT Interview Questions

### Basic

1. What is xCAT?
2. Why is xCAT used in HPC?
3. What is a node definition?
4. What is an OS image?
5. What is the role of DHCP?

### Intermediate

6. Explain the xCAT provisioning workflow.
7. Difference between stateful and stateless provisioning?
8. What does `nodeset` do?
9. What does `makedhcp` do?
10. How does xCAT interact with Ansible?
11. How does xCAT interact with Slurm?

### Production

12. A node does not receive DHCP. How do you troubleshoot it?
13. DHCP works but the node does not boot. What do you check?
14. A node is provisioned but Slurm reports it as DOWN. What do you investigate?
15. How would you provision 1,000 nodes?
16. How would you recover a corrupted compute node?
17. How would you design HA for the xCAT management layer?

---

# 6.80 HPC-AI Engineer Perspective

Do not memorize xCAT commands only.

Understand the lifecycle:

```text
                    Hardware
                       │
                       ▼
                     xCAT
                       │
                       ▼
                  Linux Node
                       │
            ┌──────────┼──────────┐
            ▼          ▼          ▼
        InfiniBand   Lustre     NVIDIA
            │          │          │
            └──────────┼──────────┘
                       ▼
                     Slurm
                       │
                       ▼
                   AI / HPC
                   Workload
```

The key engineering question is:

> **If a newly provisioned node does not become a healthy Slurm compute node, which layer failed?**

Think systematically:

```text
Hardware
   ↓
BMC
   ↓
Network
   ↓
xCAT
   ↓
Linux
   ↓
Drivers
   ↓
Storage
   ↓
InfiniBand
   ↓
Slurm
   ↓
Workload
```

This layered troubleshooting approach is more valuable than memorizing individual commands.

---

# 6.81 Chapter 6 – Final Revision

### xCAT in one sentence

> **xCAT is a cluster-management and bare-metal provisioning framework used to deploy and manage HPC compute infrastructure at scale.**

### Remember the responsibilities

```text
xCAT      → Provision
Ansible   → Configure
LDAP      → Authenticate
Lustre    → Store
InfiniBand→ Communicate
NVIDIA    → Accelerate
Slurm     → Schedule
Monitoring→ Observe
```

### Most Important Commands

```bash
lsdef
chdef
nodeset
makedhcp
makedns
rpower
rinv
rcons
tabdump
```

### Most Important Troubleshooting Principle

```text
Do not jump directly to Slurm.

Check:

Hardware
 → Network
 → Provisioning
 → Linux
 → Drivers
 → HPC Services
 → Slurm
```

**Chapter 6 complete.**
