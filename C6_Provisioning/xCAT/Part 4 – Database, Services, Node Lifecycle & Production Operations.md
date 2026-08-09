## Part 4 – Database, Services, Node Lifecycle & Production Operations

* [6.50 xCAT Database Architecture](#650-xcat-database-architecture)
* [6.51 Important xCAT Tables](#651-important-xcat-tables)
* [6.52 Node and Network Relationships](#652-node-and-network-relationships)
* [6.53 Site and Global Configuration](#653-site-and-global-configuration)
* [6.54 xCAT Service Architecture](#654-xcat-service-architecture)
* [6.55 DHCP and DNS Generation](#655-dhcp-and-dns-generation)
* [6.56 Production xCAT Commands](#656-production-xcat-commands)
* [6.57 Node Lifecycle Management](#657-node-lifecycle-management)
* [6.58 Power Management](#658-power-management)
* [6.59 Remote Console Management](#659-remote-console-management)
* [6.60 Cluster Scaling](#660-cluster-scaling)
* [6.61 xCAT High Availability](#661-xcat-high-availability)
* [6.62 Production Scenario – Adding a Node](#662-production-scenario--adding-a-node)
* [6.63 Production Scenario – Rebuilding a Node](#663-production-scenario--rebuilding-a-node)
* [6.64 Production Scenario – Removing a Node](#664-production-scenario--removing-a-node)
* [6.65 Troubleshooting Workflow](#665-troubleshooting-workflow)
* [6.66 Production Best Practices](#666-production-best-practices)
* [6.67 Interview Questions](#667-interview-questions)
* [6.68 Part 4 Summary](#668-part-4-summary)

---

# 6.50 xCAT Database Architecture

The xCAT database is one of the most important parts of the xCAT control plane.

Conceptually:

```text
                    Administrator
                          │
                          ▼
                    xCAT Commands
                          │
                          ▼
                    xCAT Database
                          │
          ┌───────────────┼────────────────┐
          ▼               ▼                ▼
        Nodes          Networks          Images
          │               │                │
          └───────────────┼────────────────┘
                          ▼
                  Generated Configuration
                          │
              ┌───────────┼───────────┐
              ▼           ▼           ▼
            DHCP         DNS       Provisioning
```

The database acts as the **desired infrastructure inventory** used by xCAT.

---

## Why the Database Is Important

Suppose a cluster contains:

```text
1000 Compute Nodes
100 GPU Nodes
20 Storage Nodes
10 Management Nodes
```

Manually maintaining all configuration would be difficult.

The database provides centralized information such as:

```text
Node
 ├── Identity
 ├── Network
 ├── Hardware
 ├── Boot configuration
 ├── Provisioning information
 └── Management information
```

---

# 6.51 Important xCAT Tables

xCAT uses database tables to represent different aspects of the cluster.

Common tables encountered in xCAT environments include:

| Table        | Purpose                                       |
| ------------ | --------------------------------------------- |
| `nodes`      | Node identity and basic information           |
| `nodehm`     | Hardware-management information               |
| `nodetype`   | Node type and provisioning-related attributes |
| `noderes`    | Resources associated with nodes               |
| `nodepos`    | Position/rack-related information             |
| `networks`   | Network definitions                           |
| `mac`        | MAC/network interface information             |
| `site`       | Site-wide configuration                       |
| `passwd`     | Node-related credential information           |
| `osimage`    | OS image definitions                          |
| `linuximage` | Linux image information                       |
| `chain`      | Boot/provisioning chain information           |

The exact table set and schema depend on the xCAT version and enabled functionality.

---

## Inspecting Tables

A useful administrative command is:

```bash
tabdump
```

For a specific table:

```bash
tabdump nodes
```

Example conceptual output:

```text
node,groups,status,comments,disable
compute001,compute,,,
compute002,compute,,,
```

The exact columns vary by xCAT release.

---

# 6.52 Node and Network Relationships

A node is connected to one or more networks through its interface information.

Conceptually:

```text
                    Node
                     │
        ┌────────────┼────────────┐
        ▼            ▼            ▼
      eth0          eth1          ib0
        │            │             │
        ▼            ▼             ▼
 Management       Storage      InfiniBand
 Network          Network        Fabric
```

This is particularly important in HPC because different traffic classes may use different fabrics.

---

## Example

```text
compute001

Management:
10.10.10.101

Storage:
10.20.10.101

InfiniBand:
172.16.10.101
```

The actual network addressing and whether IPoIB is used depend on the cluster architecture.

---

# 6.53 Site and Global Configuration

xCAT supports site-wide configuration through the `site` table.

This allows common behavior to be defined centrally.

Examples of concepts that may be configured include:

* DNS domains
* Time synchronization
* Provisioning behavior
* Network settings
* Service defaults
* Cluster-wide parameters

Inspect:

```bash
tabdump site
```

---

## Why Global Configuration Matters

Without centralized configuration:

```text
Node 1 → Configuration A
Node 2 → Configuration B
Node 3 → Configuration C
```

With centralized configuration:

```text
                 Site Configuration
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
       Node 1          Node 2          Node 3
          │              │              │
          └──────────────┼──────────────┘
                         ▼
                  Consistent State
```

This reduces configuration drift.

---

# 6.54 xCAT Service Architecture

A simplified xCAT management architecture is:

```text
                  Administrator
                       │
                       ▼
                  xCAT CLI/API
                       │
                       ▼
                    xcatd
                       │
                       ▼
                  xCAT Database
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
      DHCP            DNS          Provisioning
        │              │              │
        └──────────────┼──────────────┘
                       ▼
                  Compute Nodes
```

The exact daemon/service arrangement depends on the xCAT version and deployment.

---

## xcatd

`xcatd` is the primary xCAT daemon.

Check its status:

```bash
systemctl status xcatd
```

Check logs:

```bash
journalctl -u xcatd
```

---

## xCAT Database Backend

xCAT commonly uses a relational database backend.

The database stores the cluster state used by xCAT commands and services.

A production engineer should understand the relationship:

```text
xCAT CLI
   ↓
xCAT Service
   ↓
Database
   ↓
Generated Configuration
   ↓
Node
```

---

# 6.55 DHCP and DNS Generation

xCAT can generate configuration based on the database.

## DHCP

```bash
makedhcp
```

This uses node/network definitions to generate DHCP configuration.

---

## DNS

```bash
makedns
```

This generates DNS-related configuration from the xCAT database.

---

## Important Principle

Do not think of:

```text
Database
```

and:

```text
DHCP/DNS
```

as unrelated systems.

Instead:

```text
xCAT Database
      │
      ├── Node information
      ├── Network information
      └── Host information
             │
             ▼
      Generated Configuration
             │
       ┌─────┴─────┐
       ▼           ▼
     DHCP          DNS
```

If the database is wrong, generated configuration can also be wrong.

---

# 6.56 Production xCAT Commands

The following commands are particularly useful for an HPC Infrastructure Engineer.

---

## List Definitions

```bash
lsdef
```

Specific node:

```bash
lsdef compute001
```

Detailed:

```bash
lsdef compute001 -i
```

---

## List Groups

```bash
lsdef -t group
```

---

## Create Definition

```bash
mkdef compute001
```

---

## Change Definition

```bash
chdef compute001 <attribute>=<value>
```

---

## Remove Definition

```bash
rmdef compute001
```

---

## View Database Tables

```bash
tabdump
```

Specific table:

```bash
tabdump nodes
```

---

## DHCP

```bash
makedhcp
```

---

## DNS

```bash
makedns
```

---

## Node State

```bash
nodeset compute001
```

---

## Power

```bash
rpower compute001 stat
```

```bash
rpower compute001 on
```

```bash
rpower compute001 off
```

---

## Inventory

```bash
rinv compute001
```

---

## Remote Console

```bash
rcons compute001
```

Availability depends on the configured hardware-management and console infrastructure.

---

# 6.57 Node Lifecycle Management

A compute node has a lifecycle.

```text
New Hardware
     │
     ▼
Inventory
     │
     ▼
Registration
     │
     ▼
Provisioning
     │
     ▼
Validation
     │
     ▼
Production
     │
     ▼
Maintenance
     │
     ▼
Reprovisioning
     │
     ▼
Retirement
```

xCAT can participate in several stages of this lifecycle.

---

## Node States

A production environment may conceptually track:

```text
NEW
 │
 ▼
DISCOVERED
 │
 ▼
PROVISIONING
 │
 ▼
VALIDATING
 │
 ▼
READY
 │
 ▼
PRODUCTION
 │
 ├── MAINTENANCE
 │
 └── RETIRED
```

The actual state model depends on the surrounding automation and operational process.

---

# 6.58 Power Management

HPC nodes frequently have remote management controllers such as:

* IPMI
* Redfish
* BMC interfaces

xCAT can integrate with node hardware-management mechanisms.

---

## Check Power

```bash
rpower compute001 stat
```

---

## Power On

```bash
rpower compute001 on
```

---

## Power Off

```bash
rpower compute001 off
```

---

## Why Remote Power Matters

Consider 1,000 nodes.

Without remote power:

```text
Node Failure
     ↓
Engineer physically visits rack
     ↓
Power cycle
```

With remote management:

```text
Node Failure
     ↓
BMC / xCAT
     ↓
Remote Power Cycle
```

This significantly improves operational efficiency.

---

# 6.59 Remote Console Management

When a node does not boot correctly, SSH is often unavailable.

A remote console can provide visibility before Linux starts.

```text
Node
 │
 ├── BIOS/UEFI
 ├── Bootloader
 ├── Kernel
 └── Linux
```

SSH only becomes available later.

Therefore:

```text
SSH unavailable
       ↓
Remote Console
       ↓
Observe Boot Process
```

---

## Typical Investigation

```text
Power
  ↓
BIOS/UEFI
  ↓
Network Boot
  ↓
DHCP
  ↓
Bootloader
  ↓
Kernel
  ↓
Init
  ↓
Network
  ↓
SSH
```

The console helps determine where the boot process stops.

---

# 6.60 Cluster Scaling

xCAT becomes particularly valuable when scaling clusters.

Imagine:

```text
10 Nodes
```

Manual provisioning might still be manageable.

But:

```text
100 Nodes
```

requires automation.

And:

```text
1000+ Nodes
```

requires a controlled provisioning architecture.

---

## Scaling Model

```text
              xCAT Control Plane
                     │
       ┌─────────────┼─────────────┐
       ▼             ▼             ▼
   Node Group A  Node Group B  Node Group C
       │             │             │
    100 Nodes      200 Nodes      500 Nodes
```

Groups allow administrators to perform controlled operations on subsets of nodes.

---

## Parallel Provisioning

Conceptually:

```text
OS Image
   │
   ├── Node001
   ├── Node002
   ├── Node003
   ├── ...
   └── Node500
```

However, production deployments should consider:

* DHCP load
* Network bandwidth
* Image-server capacity
* Storage throughput
* Switch capacity
* BMC concurrency
* Database load

Scaling provisioning is not simply a matter of adding more nodes to one command.

---

# 6.61 xCAT High Availability

The xCAT management layer can become a critical dependency.

A simplified HA architecture is:

```text
                 VIP
                  │
          ┌───────┴───────┐
          ▼               ▼
      xCAT-MN1         xCAT-MN2
       ACTIVE          STANDBY
          │               │
          └───────┬───────┘
                  ▼
             Shared/Replicated
                Database
```

The exact HA implementation depends on the deployment architecture.

---

## Important HA Components

Consider protecting:

```text
xCAT Service
Database
DHCP
DNS
Images
Configuration
Management Network
BMC Access
```

---

## Failure Scenario

If the primary management server fails:

```text
xCAT-MN1
   │
   X
Failure
   │
   ▼
VIP Failover
   │
   ▼
xCAT-MN2
   │
   ▼
Cluster Management Continues
```

The exact failover behavior depends on the HA technology used.

---

# 6.62 Production Scenario – Adding a Node

Assume:

```text
New Node:
compute101

CPU:
2 × CPU

Memory:
512 GB

Network:
Ethernet + InfiniBand
```

---

## Step 1 – Hardware Inventory

Verify:

```text
Serial
CPU
Memory
NIC
MAC
BMC
HCA
Disk
```

---

## Step 2 – Register Node

Create the node definition.

```bash
mkdef compute101
```

Configure required attributes.

```bash
chdef compute101 <attribute>=<value>
```

---

## Step 3 – Validate

```bash
lsdef compute101 -i
```

---

## Step 4 – Generate Network Configuration

```bash
makedhcp
makedns
```

---

## Step 5 – Provision

Set the required OS image:

```bash
nodeset compute101 osimage=<osimage>
```

Power on:

```bash
rpower compute101 on
```

---

## Step 6 – Validate Linux

```bash
hostname
```

```bash
ip addr
```

```bash
lscpu
```

```bash
free -h
```

---

## Step 7 – Validate HPC Stack

```bash
ibstat
```

```bash
systemctl status slurmd
```

```bash
mount | grep lustre
```

---

## Step 8 – Add to Production

Only after all checks pass:

```text
compute101
    ↓
Validation PASS
    ↓
Slurm READY
    ↓
Production
```

---

# 6.63 Production Scenario – Rebuilding a Node

Suppose:

```text
compute051
```

has severe OS corruption.

Instead of manually repairing hundreds of packages:

```text
Remove Node from Production
          ↓
Reprovision
          ↓
Validate
          ↓
Return to Slurm
```

---

## Workflow

```text
Slurm
 ↓
Drain Node
 ↓
xCAT
 ↓
Reinstall OS
 ↓
Ansible
 ↓
Configure HPC Stack
 ↓
Validate
 ↓
Slurm Resume
```

This approach is often more reliable than performing extensive manual repairs.

---

# 6.64 Production Scenario – Removing a Node

Node retirement should be controlled.

```text
Production
    ↓
Drain
    ↓
Backup Required Data
    ↓
Remove from Scheduling
    ↓
Remove/disable Provisioning
    ↓
Remove DNS/DHCP References
    ↓
Hardware Retirement
```

Never simply delete a node definition without checking dependencies.

Potential dependencies include:

* Slurm
* DNS
* DHCP
* Monitoring
* LDAP/SSSD
* Lustre
* Inventory
* Asset management

---

# 6.65 Troubleshooting Workflow

When xCAT provisioning fails, avoid randomly changing configuration.

Use a layered troubleshooting approach.

```text
Layer 1
Physical Hardware
       ↓
Layer 2
BMC / Power
       ↓
Layer 3
Switch / VLAN
       ↓
Layer 4
DHCP
       ↓
Layer 5
Bootloader
       ↓
Layer 6
Kernel / Installer
       ↓
Layer 7
Linux
       ↓
Layer 8
Post-Install
       ↓
Layer 9
Slurm / HPC Stack
```

---

## Example

### Symptom

Node does not boot.

Start:

```bash
rpower compute001 stat
```

Then:

```text
Is it powered on?
       │
       ├── NO → BMC/Power
       │
       └── YES
             ↓
       Does it get DHCP?
             │
             ├── NO → Network/DHCP
             │
             └── YES
                   ↓
             Does bootloader load?
                   │
                   ├── NO → Boot service
                   │
                   └── YES
                         ↓
                    Does installer run?
```

This is much faster than checking everything simultaneously.

---

# 6.66 Production Best Practices

## Configuration

* Keep xCAT configuration version controlled where practical.
* Document node roles.
* Standardize node naming.
* Maintain accurate MAC/IP mappings.
* Avoid undocumented manual changes.

---

## Images

* Maintain tested golden images.
* Version images.
* Test images on a small node group.
* Keep a known-good rollback image.
* Avoid unnecessary image variations.

---

## Network

* Separate management and workload traffic where appropriate.
* Maintain consistent DNS.
* Validate DHCP before provisioning.
* Document VLANs and subnets.
* Monitor provisioning-network capacity.

---

## Security

* Protect BMC interfaces.
* Restrict management access.
* Secure provisioning services.
* Protect xCAT database credentials.
* Use least privilege.
* Audit administrative changes.

---

## Operations

* Drain nodes before destructive maintenance.
* Validate after provisioning.
* Automate repetitive operations.
* Keep provisioning logs.
* Maintain recovery procedures.

---

# 6.67 Interview Questions

## Basic

1. What is the role of the xCAT database?
2. What is `tabdump`?
3. What is `xcatd`?
4. Why are DHCP and DNS important to xCAT?
5. What is node lifecycle management?

---

## Intermediate

1. Explain the relationship between the xCAT database and generated DHCP/DNS configuration.
2. How would you add a new compute node?
3. How would you rebuild a corrupted node?
4. How would you safely remove a node?
5. How does remote console access help troubleshooting?
6. Why is BMC management important?

---

## Advanced

1. How would you design an HA xCAT management architecture?
2. How would you scale provisioning to 1,000+ nodes?
3. What bottlenecks can appear during mass provisioning?
4. How would you recover if the xCAT database becomes unavailable?
5. How would you design a disaster-recovery strategy for xCAT?
6. How would you prevent configuration drift between xCAT and Ansible?
7. How would you troubleshoot a node that receives DHCP but never reaches the OS installer?

---

# 6.68 Part 4 Summary

xCAT should be viewed as an **infrastructure lifecycle and provisioning control layer**.

The overall model is:

```text
                       xCAT
                        │
          ┌─────────────┼─────────────┐
          ▼             ▼             ▼
       Database       Network       Images
          │          DHCP/DNS         │
          │             │             │
          └─────────────┼─────────────┘
                        ▼
                   Provisioning
                        │
                        ▼
                   Linux Nodes
                        │
              ┌─────────┼─────────┐
              ▼         ▼         ▼
           Ansible    Slurm     Monitoring
              │         │
              ▼         ▼
          HPC Stack  Workloads
```

The production engineer's responsibility is not simply to know xCAT commands.

The important skill is understanding the **entire node lifecycle**:

```text
Hardware
   ↓
Inventory
   ↓
xCAT Definition
   ↓
Network Configuration
   ↓
Provisioning
   ↓
Linux
   ↓
Configuration Management
   ↓
HPC Stack
   ↓
Validation
   ↓
Slurm
   ↓
Production
   ↓
Maintenance
   ↓
Reprovision / Retirement
```

---

# Part 4 Learning Checklist

You should now be able to:

* Explain the xCAT database architecture.
* Identify important xCAT tables.
* Understand node/network relationships.
* Explain site-wide configuration.
* Understand the xCAT service architecture.
* Generate DHCP/DNS configuration.
* Use important production xCAT commands.
* Manage the node lifecycle.
* Perform remote power operations.
* Use remote console concepts for troubleshooting.
* Understand large-scale provisioning.
* Explain xCAT HA considerations.
* Add, rebuild, and retire nodes safely.
* Troubleshoot provisioning layer-by-layer.

---

# Next Part

**Chapter 6 – Part 5**

Topics:

* Advanced xCAT Provisioning
* Large-Scale Cluster Deployment
* xCAT + Ansible Automation
* xCAT + Slurm Integration
* GPU/Heterogeneous Node Provisioning
* InfiniBand and Storage Integration
* Automated Validation
* Configuration Drift
* Production Failure Scenarios
* xCAT Security
* Backup & Disaster Recovery
* Operational Best Practices
* Advanced Interview Questions
