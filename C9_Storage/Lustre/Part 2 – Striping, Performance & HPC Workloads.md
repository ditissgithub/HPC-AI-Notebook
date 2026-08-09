# Chapter 9 – Lustre Storage

## Part 2 – Striping, Performance & HPC Workloads

* [9.11 Lustre Striping](#911-lustre-striping)
* [9.12 Stripe Count and Stripe Size](#912-stripe-count-and-stripe-size)
* [9.13 Checking File Layout](#913-checking-file-layout)
* [9.14 Choosing a Stripe Configuration](#914-choosing-a-stripe-configuration)
* [9.15 Lustre for HPC Workloads](#915-lustre-for-hpc-workloads)
* [9.16 Lustre for AI Workloads](#916-lustre-for-ai-workloads)
* [9.17 I/O Performance Monitoring](#917-io-performance-monitoring)
* [9.18 Common Performance Problems](#918-common-performance-problems)
* [9.19 Important Commands](#919-important-commands)
* [9.20 Quick Revision](#920-quick-revision)


# 9.11 Lustre Striping

Lustre can distribute a file across multiple OSTs.

Without striping:

```text id="83w4u0"
File
 │
 ▼
OST01
```

With striping:

```text id="f8k3q0"
             File
               │
       ┌───────┼───────┐
       ▼       ▼       ▼
     OST01   OST02   OST03
```

This enables parallel I/O.

---

# 9.12 Stripe Count and Stripe Size

Two important concepts:

### Stripe Count

Number of OSTs used by a file.

```bash
lfs setstripe -c 4 /lustre/project
```

Conceptually:

```text
Stripe Count = 4

File
 ├── OST01
 ├── OST02
 ├── OST03
 └── OST04
```

### Stripe Size

Amount of consecutive file data placed on one OST before moving to the next stripe.

Example:

```bash
lfs setstripe -S 4M -c 4 /lustre/project
```

Meaning conceptually:

```text
4 MB → OST01
4 MB → OST02
4 MB → OST03
4 MB → OST04
...
```

The optimal values depend on workload and filesystem design.

---

# 9.13 Checking File Layout

Check striping:

```bash
lfs getstripe /lustre/project/file.dat
```

Example information may include:

```text id="2sdf89"
stripe_count
stripe_size
OST index
```

Check a directory:

```bash
lfs getstripe /lustre/project
```

Remember:

> Stripe settings affect newly created files; changing a directory's default layout does not automatically rewrite existing files.

---

# 9.14 Choosing a Stripe Configuration

There is no universal "best" stripe configuration.

Consider the workload.

### Small Files

```text id="3c84u4"
Millions of small files
        ↓
Metadata-heavy workload
        ↓
MDT becomes important
```

Increasing OST stripe count may not solve a metadata bottleneck.

### Large Sequential Files

```text id="av3w7r"
Large file
   ↓
Multiple OSTs
   ↓
Parallel throughput
```

Striping across multiple OSTs can improve throughput.

### Many Concurrent Jobs

```text id="6n1f7q"
1000 jobs
   │
   ├── Read
   ├── Write
   └── Metadata operations
          │
          ▼
      Lustre
```

The workload must be considered at cluster scale rather than for a single benchmark.

---

# 9.15 Lustre for HPC Workloads

Common HPC workloads:

```text id="gq9p1r"
MPI
 │
 ├── Checkpointing
 ├── Simulation output
 ├── Large datasets
 └── Parallel reads/writes
```

Typical data path:

```text id="9up0v3"
Compute Nodes
      │
      ▼
 Lustre Clients
      │
      ▼
 ┌────┴─────┐
 MDT       OSTs
            │
            ▼
        Storage
```

Lustre is particularly useful when many nodes need to access the same dataset concurrently.

---

# 9.16 Lustre for AI Workloads

AI training can generate heavy parallel reads.

Example:

```text id="5k0d6e"
          AI Training
              │
      ┌───────┼───────┐
      ▼       ▼       ▼
    GPU01   GPU02   GPU03
      │       │       │
      └───────┼───────┘
              ▼
            Lustre
              │
       Training Dataset
```

Potential bottleneck:

```text id="e6g7j3"
GPUs waiting
     ↓
Data loading slow
     ↓
Storage throughput insufficient
```

Therefore AI infrastructure requires attention to:

* Aggregate throughput
* Metadata performance
* File layout
* Concurrent access
* Dataset organization
* Checkpoint performance

---

# 9.17 I/O Performance Monitoring

Useful Linux tools:

```bash
iostat -xz 1
```

```bash
sar -d 1
```

Check Lustre:

```bash
lfs df -h
```

Lustre statistics/tools available vary by Lustre version and deployment.

Useful areas to monitor:

```text id="2mb6g6"
OST utilization
MDT utilization
I/O throughput
I/O latency
Network throughput
Client errors
Filesystem capacity
```

---

# 9.18 Common Performance Problems

## Problem 1 – Single OST Bottleneck

```text id="p4h3y5"
Application
    │
    ▼
  OST01  ← overloaded
    │
    ▼
Low throughput
```

Check filesystem usage:

```bash
lfs df -h
```

Check file layout:

```bash
lfs getstripe <file>
```

---

## Problem 2 – Metadata Bottleneck

Symptoms:

```text id="t1d0n5"
Many small files
      ↓
Slow create/stat/open
      ↓
MDT pressure
```

Investigate:

```text
MDT utilization
Metadata operation rate
Application file pattern
```

---

## Problem 3 – Client I/O Bottleneck

Symptoms:

```text id="y9x5tq"
One node slow
Others normal
```

Check:

```bash
iostat -xz 1
dmesg | grep -i lustre
```

Then compare with another client.

---

## Problem 4 – Network Bottleneck

Lustre performance depends heavily on the storage network.

```text id="7v0j6a"
Client
  │
  ▼
Network
  │
  ▼
Lustre Server
```

A network bottleneck can appear as a storage bottleneck.

Check:

```bash
ip -s link
```

For an InfiniBand-based storage network:

```bash
ibstat
```

and appropriate RDMA/IB monitoring tools.

---

# 9.19 Important Commands

### Filesystem

```bash
df -hT
mount | grep lustre
lfs df -h
```

### Striping

```bash
lfs getstripe <file>
lfs getstripe <directory>
```

```bash
lfs setstripe -c 4 <directory>
```

```bash
lfs setstripe -S 4M -c 4 <directory>
```

### Linux I/O

```bash
iostat -xz 1
sar -d 1
```

### Kernel

```bash
dmesg | grep -i lustre
journalctl -k | grep -i lustre
```

### Network

```bash
ip -s link
```

---

# 9.20 Quick Revision

## Striping

```text
File
 │
 ├── OST01
 ├── OST02
 ├── OST03
 └── OST04
```

## Performance Layers

```text
Application
     ↓
Lustre Client
     ↓
Network
     ↓
MDT / OST
     ↓
Storage
```

A slow Lustre workload does **not** automatically mean the storage disks are slow.

Check all layers:

```text
Application
    ↓
Client
    ↓
Network
    ↓
MDT / OST
    ↓
Backend Storage
```

### HPC Engineer Rule

> **Always identify whether the bottleneck is metadata, data throughput, client, network, or backend storage before changing Lustre parameters.**

# End of Chapter 9 – Part 2
