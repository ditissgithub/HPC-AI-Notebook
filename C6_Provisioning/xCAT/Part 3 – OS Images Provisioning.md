## Part 3 – OS Images, Stateful/Stateless Provisioning & Post-Install Configuration

* [6.33 OS Image Management](#633-os-image-management)
* [6.34 Golden Image Concept](#634-golden-image-concept)
* [6.35 Stateful Provisioning](#635-stateful-provisioning)
* [6.36 Stateless Provisioning](#636-stateless-provisioning)
* [6.37 Stateful vs Stateless in HPC](#637-stateful-vs-stateless-in-hpc)
* [6.38 Linux Installation Workflow](#638-linux-installation-workflow)
* [6.39 Kickstart-Based Installation](#639-kickstart-based-installation)
* [6.40 Post-Install Configuration](#640-post-install-configuration)
* [6.41 xCAT + Ansible](#641-xcat--ansible)
* [6.42 xCAT + Slurm](#642-xcat--slurm)
* [6.43 GPU Node Provisioning](#643-gpu-node-provisioning)
* [6.44 InfiniBand Node Provisioning](#644-infiniband-node-provisioning)
* [6.45 Real HPC Provisioning Example](#645-real-hpc-provisioning-example)
* [6.46 Production Best Practices](#646-production-best-practices)
* [6.47 Troubleshooting Scenarios](#647-troubleshooting-scenarios)
* [6.48 Interview Questions](#648-interview-questions)
* [6.49 Part 3 Summary](#649-part-3-summary)

---

# 6.33 OS Image Management

OS provisioning is one of the most important xCAT functions in an HPC environment.

Instead of installing every node manually:

```text
Manual Installation

Node 1 → Install
Node 2 → Install
Node 3 → Install
...
Node 1000 → Install
```

xCAT provides an automated approach:

```text
                   OS Image
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
       Node 1       Node 2       Node 3
          │            │            │
          └────────────┼────────────┘
                       ▼
                Consistent Nodes
```

The image provides a repeatable operating-system baseline.

---

# 6.34 Golden Image Concept

A **golden image** is a tested baseline image that represents the desired state of a particular node class.

Example:

```text
Rocky Linux
    +
Required Packages
    +
Monitoring
    +
Slurm Client
    +
RDMA Stack
    +
GPU Software
    +
Cluster Configuration
    ↓
GPU Golden Image
```

Different node types may have different images.

```text
                    Base OS
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
    CPU Image       GPU Image    High-Memory Image
        │              │              │
        ▼              ▼              ▼
    CPU Nodes       GPU Nodes      HM Nodes
```

---

## Image Design Principle

Do not create unnecessary images.

A better approach is:

```text
Common Base
     │
     ├── CPU Role
     ├── GPU Role
     └── High-Memory Role
```

This reduces image maintenance.

---

# 6.35 Stateful Provisioning

In **stateful provisioning**, the node's operating system is installed on local storage.

```text
Compute Node
      │
      ▼
Local Disk
      │
      └── Linux OS
```

After installation, the node can boot from its local disk.

---

## Typical Flow

```text
Network Boot
     ↓
Installer
     ↓
Local Disk
     ↓
Linux Installation
     ↓
Bootloader
     ↓
Local OS
```

---

## Advantages

* Persistent OS
* Local configuration
* Local package installation
* Suitable for nodes requiring local state
* Easy to understand and troubleshoot

---

## Disadvantages

* Configuration drift is possible.
* Node recovery may require reinstallation.
* Local disks must be maintained.
* Maintaining consistency across many nodes requires additional automation.

---

# 6.36 Stateless Provisioning

In a stateless model, the node obtains its operating environment from centralized infrastructure rather than relying on a persistent local OS installation.

Conceptually:

```text
Provisioning Infrastructure
          │
          ▼
      Boot Image
          │
          ▼
      Compute Node
```

The local disk may be unused for the operating system or used only for temporary/local data depending on the design.

---

## Typical Flow

```text
Power On
   ↓
Network Boot
   ↓
Retrieve Runtime Environment
   ↓
Initialize Node
   ↓
Production
```

---

## Advantages

* Strong configuration consistency
* Fast recovery
* Easy node replacement
* Reduced configuration drift
* Centralized management

---

## Challenges

* More dependency on provisioning infrastructure
* Network becomes critical
* Boot infrastructure must be reliable
* Troubleshooting can be more complex

---

# 6.37 Stateful vs Stateless in HPC

| Feature             | Stateful               | Stateless                |
| ------------------- | ---------------------- | ------------------------ |
| OS stored locally   | Yes                    | Usually no persistent OS |
| Local persistence   | High                   | Low                      |
| Node recovery       | Reinstall/rebuild      | Reboot/redeploy          |
| Configuration drift | Possible               | Lower                    |
| Network dependency  | Primarily provisioning | Strong                   |
| Local disk          | Important              | Optional for OS          |
| Management          | Local + automation     | Centralized              |
| HPC suitability     | High                   | High                     |

There is no universal answer.

The correct design depends on:

* Cluster architecture
* Hardware
* Network
* Storage
* Recovery requirements
* Operational model

---

# 6.38 Linux Installation Workflow

A typical xCAT-based stateful installation can be represented as:

```text
Bare Metal
    │
    ▼
Power On
    │
    ▼
BIOS / UEFI
    │
    ▼
Network Boot
    │
    ▼
DHCP
    │
    ▼
Bootloader
    │
    ▼
Installer
    │
    ▼
Kickstart / Installation Configuration
    │
    ▼
Partition Disk
    │
    ▼
Install Linux
    │
    ▼
Configure Network
    │
    ▼
Install Packages
    │
    ▼
Configure Services
    │
    ▼
Reboot
    │
    ▼
Production Node
```

The exact boot mechanism depends on the xCAT and operating-system environment.

---

# 6.39 Kickstart-Based Installation

On RHEL-family operating systems, **Kickstart** provides automated installation configuration.

A Kickstart file can define:

* Language
* Keyboard
* Network
* Disk partitioning
* Package selection
* Users
* Authentication
* Services
* Post-install commands

Conceptual example:

```text
Installation
    │
    ├── Network
    ├── Storage
    ├── Packages
    ├── Users
    └── Post-install
```

---

## Simplified Kickstart Example

```text
lang en_US.UTF-8
keyboard us

network --bootproto=dhcp --device=eth0

rootpw --lock

firewall --disabled
selinux --enforcing

%packages
@^minimal-environment
chrony
vim
wget
curl
%end

%post
systemctl enable chronyd
%end
```

The exact Kickstart syntax must match the target OS release.

---

# 6.40 Post-Install Configuration

OS installation is only the beginning.

A production HPC node may require:

```text
Linux
 │
 ├── DNS
 ├── NTP/Chrony
 ├── LDAP/SSSD
 ├── Slurm
 ├── InfiniBand
 ├── Lustre
 ├── GPU Driver
 ├── Monitoring
 └── Security
```

This is where configuration management becomes extremely valuable.

---

## Typical Post-Install Flow

```text
xCAT
  ↓
Linux OS
  ↓
Basic Configuration
  ↓
Ansible
  ↓
Cluster Software
  ↓
Validation
  ↓
Slurm
  ↓
Production
```

---

## Example Tasks

Ansible or other automation can configure:

```text
Hostname
DNS
NTP
Packages
Users
LDAP
SSSD
Slurm
OFED
Lustre
GPU
Monitoring
Security
```

---

# 6.41 xCAT + Ansible

xCAT and Ansible solve different problems.

### xCAT

Primarily handles:

```text
Bare Metal
     ↓
OS Provisioning
     ↓
Node Lifecycle
```

### Ansible

Primarily handles:

```text
Configured OS
     ↓
Software Configuration
     ↓
Desired State
```

Together:

```text
                  xCAT
                    │
                    ▼
              Bare-Metal OS
                    │
                    ▼
                 Ansible
                    │
       ┌────────────┼────────────┐
       ▼            ▼            ▼
     Slurm        LDAP        Monitoring
       │
       ▼
    Production
```

---

## Example Division of Responsibility

| Task                     | xCAT | Ansible |
| ------------------------ | ---: | ------: |
| Bare-metal provisioning  |    ✓ |         |
| Network boot             |    ✓ |         |
| OS installation          |    ✓ |         |
| Host configuration       |      |       ✓ |
| Package configuration    |      |       ✓ |
| Slurm configuration      |      |       ✓ |
| Monitoring configuration |      |       ✓ |
| Security hardening       |      |       ✓ |

The exact division can vary by implementation.

---

# 6.42 xCAT + Slurm

xCAT provisions the node.

Slurm manages the node after provisioning.

```text
              xCAT
                │
                ▼
          Compute Node
                │
                ▼
          Linux + Slurm
                │
                ▼
             slurmd
                │
                ▼
            slurmctld
                │
                ▼
          User Workload
```

---

## Lifecycle

```text
New Hardware
     ↓
xCAT Provision
     ↓
OS Validation
     ↓
Slurm Installation
     ↓
slurmd Configuration
     ↓
Node Registration
     ↓
Health Check
     ↓
Slurm Resume
     ↓
Production
```

This separation is essential.

> xCAT creates and prepares the infrastructure; Slurm controls workload execution.

---

# 6.43 GPU Node Provisioning

GPU nodes require additional validation.

Example:

```text
Bare Metal
    ↓
xCAT
    ↓
Linux
    ↓
NVIDIA Driver
    ↓
CUDA
    ↓
GPU Validation
    ↓
Slurm GRES
    ↓
Production
```

---

## GPU Validation

After provisioning:

```bash
lspci | grep -i nvidia
```

Then:

```bash
nvidia-smi
```

Check GPUs:

```bash
nvidia-smi -L
```

Check topology:

```bash
nvidia-smi topo -m
```

---

## GPU Node Checklist

```text
[ ] GPU detected
[ ] Driver loaded
[ ] nvidia-smi works
[ ] CUDA validated
[ ] GPU topology validated
[ ] Slurm GRES configured
[ ] GPU allocation tested
[ ] Monitoring enabled
```

---

# 6.44 InfiniBand Node Provisioning

HPC compute nodes frequently require high-speed InfiniBand networking.

Provisioning flow:

```text
xCAT
 ↓
Linux
 ↓
OFED / RDMA Stack
 ↓
InfiniBand Driver
 ↓
OpenSM / Fabric
 ↓
ibstat
 ↓
RDMA Validation
```

---

## Basic Validation

```bash
ibstat
```

```bash
ibdev2netdev
```

```bash
rdma link
```

Check interfaces:

```bash
ip addr
```

---

## Production Consideration

The node may have:

```text
Ethernet
   +
InfiniBand
   +
GPU
```

All three must be correctly configured for distributed AI workloads.

For example:

```text
GPU
 │
 ├── PCIe
 │
 ▼
InfiniBand HCA
 │
 ▼
RDMA Fabric
 │
 ▼
Remote GPU
```

This becomes especially important for GPU clusters using GPUDirect RDMA.

---

# 6.45 Real HPC Provisioning Example

Consider a new GPU compute node:

```text
Node:
gpu001

GPU:
4 × NVIDIA GPU

Network:
Ethernet + InfiniBand

Storage:
Local OS + Lustre client
```

---

## Phase 1 – Hardware Registration

Record:

```text
Hostname
MAC
IP
BMC
GPU
CPU
Memory
InfiniBand HCA
```

---

## Phase 2 – xCAT Definition

Create/configure the node definition:

```text
gpu001
   │
   ├── IP
   ├── MAC
   ├── Network
   ├── Image
   └── Hardware role
```

---

## Phase 3 – Provision OS

```text
DHCP
 ↓
Network Boot
 ↓
Installer
 ↓
OS
```

---

## Phase 4 – Configure Node

Use Ansible or equivalent automation:

```text
DNS
NTP
LDAP
Slurm
OFED
Lustre
NVIDIA
Monitoring
Security
```

---

## Phase 5 – Hardware Validation

```bash
lscpu
```

```bash
free -h
```

```bash
nvidia-smi
```

```bash
ibstat
```

---

## Phase 6 – Slurm Validation

```bash
systemctl status slurmd
```

```bash
scontrol show node gpu001
```

---

## Phase 7 – GPU Job Test

```bash
srun --gpus=1 nvidia-smi
```

---

## Phase 8 – Production

Only after validation:

```text
Provisioned
     ↓
Validated
     ↓
Healthy
     ↓
Slurm Available
     ↓
Production
```

---

# 6.46 Production Best Practices

## 1. Maintain Golden Images

Keep tested OS images for each node class.

```text
CPU
GPU
High Memory
Service
```

---

## 2. Separate Provisioning and Configuration

Use:

```text
xCAT → Provision
Ansible → Configure
Slurm → Schedule
```

This makes troubleshooting easier.

---

## 3. Validate Before Production

Never put a newly provisioned node directly into production.

Perform:

```text
OS
Network
Storage
InfiniBand
GPU
Slurm
Monitoring
```

validation first.

---

## 4. Version Control

Store:

* Kickstart files
* Ansible playbooks
* xCAT configuration
* Image build definitions
* Slurm configuration

in version control.

---

## 5. Standardize Node Classes

Use clearly defined roles:

```text
CPU
GPU
High-Memory
Login
Storage
Management
```

---

## 6. Automate Hardware Validation

Compare:

```text
Expected Hardware
        vs
Actual Hardware
```

before accepting a node.

---

## 7. Minimize Configuration Drift

After provisioning:

```text
xCAT
  ↓
Ansible
  ↓
Validation
```

Periodic compliance checks should detect unauthorized changes.

---

# 6.47 Troubleshooting Scenarios

## Scenario 1 – OS Installation Never Starts

### Check

```text
Node
 ↓
DHCP
 ↓
Bootloader
 ↓
Kernel
 ↓
Installer
```

Check:

```bash
lsdef gpu001 -i
```

Then verify:

```text
MAC
DHCP
VLAN
Boot configuration
TFTP/HTTP
```

---

# Scenario 2 – OS Installs but Node Has No Network

Check:

```bash
ip addr
```

```bash
ip route
```

```bash
cat /etc/resolv.conf
```

Then verify:

```text
Network configuration
VLAN
DNS
Gateway
```

---

# Scenario 3 – GPU Missing After Provisioning

Check:

```bash
lspci | grep -i nvidia
```

If detected:

```bash
nvidia-smi
```

If `lspci` detects the GPU but `nvidia-smi` fails:

```text
Hardware
   ↓
PCIe       ✓
   ↓
Driver     ✗
```

Investigate the driver/kernel stack.

---

# Scenario 4 – InfiniBand Missing

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
OFED/RDMA
Firmware
Cable
Switch
Subnet Manager
```

---

# Scenario 5 – Node Provisioned but Slurm DOWN

Check:

```bash
systemctl status slurmd
```

Then:

```bash
journalctl -u slurmd
```

On the Slurm controller:

```bash
scontrol show node gpu001
```

Common causes:

* Configuration mismatch
* DNS problem
* Authentication problem
* Resource mismatch
* `slurmd` failure
* GPU/GRES mismatch

---

# 6.48 Interview Questions

## Basic

1. What is a golden image?
2. What is stateful provisioning?
3. What is stateless provisioning?
4. Why is post-install configuration required?
5. What is Kickstart?
6. Why are images useful in HPC?

---

## Intermediate

1. How would you design a GPU node image?
2. How would you integrate xCAT with Ansible?
3. How does xCAT interact with Slurm?
4. What validation should happen after OS provisioning?
5. How would you provision CPU and GPU nodes differently?
6. How would you minimize configuration drift?

---

## Advanced

1. Design an end-to-end workflow for provisioning 1,000 HPC nodes.
2. How would you design a rollback strategy for a broken OS image?
3. A new GPU node provisions successfully but fails Slurm health checks. How would you troubleshoot it?
4. How would you automate hardware validation after provisioning?
5. How would you design xCAT + Ansible for a production HPC environment?
6. How would you upgrade the OS image without disrupting the existing cluster?
7. How would you provision a heterogeneous cluster containing CPU, GPU, and high-memory nodes?

---

# 6.49 Part 3 Summary

The complete provisioning lifecycle can be viewed as:

```text
                  Hardware
                     │
                     ▼
                   xCAT
                     │
                     ▼
                Network Boot
                     │
                     ▼
                  OS Image
                     │
                     ▼
              Linux Installation
                     │
                     ▼
                  Ansible
                     │
        ┌────────────┼────────────┐
        ▼            ▼            ▼
      Slurm       InfiniBand    Lustre
        │            │            │
        └────────────┼────────────┘
                     ▼
                GPU / CPU
                 Validation
                     │
                     ▼
               Health Checks
                     │
                     ▼
                  Slurm
                     │
                     ▼
                Production
```

The key operational model is:

```text
xCAT
 ↓
Provision the node

Ansible
 ↓
Configure the node

Slurm
 ↓
Schedule workloads

Monitoring
 ↓
Observe the node

Engineer
 ↓
Validate and troubleshoot
```

A production HPC-AI Infrastructure Engineer should understand this complete lifecycle rather than treating OS installation as the end of provisioning.

---

# Part 3 Learning Checklist

You should now be able to:

* Explain golden images.
* Compare stateful and stateless provisioning.
* Understand automated Linux installation.
* Explain the role of Kickstart.
* Design post-install configuration.
* Understand xCAT + Ansible integration.
* Understand xCAT + Slurm integration.
* Provision GPU nodes conceptually.
* Validate InfiniBand after provisioning.
* Perform post-provisioning hardware validation.
* Troubleshoot provisioning failures.
* Design a production node onboarding workflow.

---

# Next Part

**Chapter 6 – Part 4**

Topics:

* xCAT Database in Depth
* Node Tables
* Network Tables
* OS Image Tables
* Site & Global Configuration
* DHCP/DNS Generation
* xCAT Service Architecture
* xCAT Commands in Production
* Node Lifecycle Management
* Power & Console Management
* Cluster Scaling
* HA Considerations
* Real Production Scenarios
* Troubleshooting
* Interview Questions
