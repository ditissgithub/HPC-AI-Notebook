## Part 1 – Architecture, Components & Data Flow

* [9.1 What is Lustre?](#91-what-is-lustre)
* [9.2 Why Lustre in HPC](#92-why-lustre-in-hpc)
* [9.3 Lustre Architecture](#93-lustre-architecture)
* [9.4 MGS](#94-mgs)
* [9.5 MDT](#95-mdt)
* [9.6 OST](#96-ost)
* [9.7 Lustre Client](#97-lustre-client)
* [9.8 Lustre Data Flow](#98-lustre-data-flow)
* [9.9 Important Commands](#99-important-commands)
* [9.10 Quick Revision](#910-quick-revision)

---

# 9.1 What is Lustre?

**Lustre** is a parallel distributed filesystem designed for large-scale HPC workloads.

It is optimized for:

* High throughput
* Parallel I/O
* Large files
* Large numbers of clients
* Scientific and AI workloads

Typical environment:

```text
HPC Cluster
    │
    ├── Compute Nodes
    ├── GPU Nodes
    └── Login Nodes
             │
             ▼
           Lustre
```

---

# 9.2 Why Lustre in HPC

Traditional filesystem:

```text
Many Clients
     │
     ▼
Single Server
     │
     ▼
Storage
```

This can become an I/O bottleneck.

Lustre separates metadata and bulk data:

```text
             Lustre
                │
       ┌────────┴────────┐
       ▼                 ▼
   Metadata            Data
       │                 │
      MDT               OSTs
```

This allows many clients to access storage in parallel.

---

# 9.3 Lustre Architecture

Core components:

```text
                    Lustre
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
       MGS            MDT            OSTs
        │              │              │
   Configuration    Metadata        File Data
                       │              │
                       └──────┬───────┘
                              ▼
                           Clients
```

### Components

| Component | Purpose                         |
| --------- | ------------------------------- |
| MGS       | Stores filesystem configuration |
| MDT       | Stores metadata                 |
| OST       | Stores file data                |
| Client    | Mounts and accesses Lustre      |

---

# 9.4 MGS

**MGS – Management Server**

Stores Lustre filesystem configuration and target information.

Conceptually:

```text
MGS
 │
 ├── Filesystem configuration
 ├── Target information
 └── Configuration parameters
```

Check mounted Lustre filesystems:

```bash
mount | grep lustre
```

---

# 9.5 MDT

**MDT – Metadata Target**

Stores filesystem metadata such as:

* File names
* Directory structure
* Permissions
* Ownership
* File attributes
* Layout information

Example:

```text
/home/user01/model.bin

Metadata
   │
   ├── owner
   ├── permissions
   ├── size
   └── layout
```

The MDT does **not** normally hold the bulk contents of the file.

---

# 9.6 OST

**OST – Object Storage Target**

Stores the actual file data.

Example:

```text
model.bin
   │
   ├── Data → OST0001
   ├── Data → OST0002
   └── Data → OST0003
```

Multiple OSTs allow parallel I/O.

---

# 9.7 Lustre Client

A compute node mounts the Lustre filesystem as a client.

Example:

```bash
mount | grep lustre
```

Typical mount:

```text
lustre-mgs:/fs01 /lustre
```

Check:

```bash
df -h /lustre
```

Lustre clients communicate with the metadata and storage targets through the Lustre network.

---

# 9.8 Lustre Data Flow

Suppose an application writes:

```text
/lustre/project/data.bin
```

Conceptually:

```text
Application
     │
     ▼
Lustre Client
     │
     ├──────────────► MDT
     │                │
     │             Metadata
     │
     └──────────────► OSTs
                      │
                  File Data
```

For parallel workloads:

```text
             Application
                  │
             Lustre Client
                  │
        ┌─────────┼─────────┐
        ▼         ▼         ▼
      OST01     OST02     OST03
        │         │         │
        └─────────┼─────────┘
                  ▼
             Parallel I/O
```

This is why Lustre is widely used for HPC workloads.

---

# 9.9 Important Commands

## Check Lustre Mount

```bash
mount | grep lustre
```

```bash
df -hT | grep lustre
```

## Check Filesystem

```bash
lfs df
```

```bash
lfs df -h
```

## Filesystem Information

```bash
lfs getstripe /lustre/path/file
```

## Directory Stripe

```bash
lfs getstripe /lustre/path
```

## Set Stripe

Example:

```bash
lfs setstripe -c 4 /lustre/project
```

`-c 4` requests a stripe count of four OST objects for newly created files/directories as applicable.

## Check Client

```bash
mount | grep lustre
```

## Check Kernel Messages

```bash
dmesg | grep -i lustre
```

or:

```bash
journalctl -k | grep -i lustre
```

---

# 9.10 Quick Revision

## Remember the Four Components

```text
MGS → Configuration
MDT → Metadata
OST → Data
Client → Access
```

## Core Architecture

```text
                 Lustre
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
         MDT                 OSTs
     Metadata              File Data
          │                   │
          └─────────┬─────────┘
                    ▼
                  Client
                    │
                    ▼
               HPC Workload
```

## Essential Commands

```bash
mount | grep lustre
df -hT
lfs df
lfs df -h
lfs getstripe <path>
lfs setstripe -c <count> <path>
dmesg | grep -i lustre
```

> **HPC Engineer takeaway:** Lustre separates **metadata (MDT)** from **bulk file data (OSTs)** and enables parallel access from many HPC clients.

# End of Chapter 9 – Part 1
