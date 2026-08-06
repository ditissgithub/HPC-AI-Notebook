---

# Part 2 – HPC Software Stack and Cluster Architecture

---

## Table of Contents

- [2.1 HPC Software Stack](#21-hpc-software-stack)
- [2.2 HPC Job Lifecycle](#22-hpc-job-lifecycle)
- [2.3 Data Flow Inside an HPC Cluster](#23-data-flow-inside-an-hpc-cluster)
- [2.4 Compute Layer (CPU & GPU)](#24-compute-layer-cpu--gpu)
- [2.5 Memory Layer](#25-memory-layer)
- [2.6 Storage Layer](#26-storage-layer)
- [2.7 Network Fabric](#27-network-fabric)
- [2.8 Putting Everything Together](#28-putting-everything-together)
- [Production Insight](#production-insight)
- [Key Takeaways](#key-takeaways)

---

# 2.1 HPC Software Stack

## Concept

An HPC cluster is much more than a collection of servers. It is a layered software ecosystem where each layer provides a specific capability required to execute scientific or AI workloads.

Each layer depends on the layer beneath it.

```
+----------------------------------------------------+
|                 User Applications                  |
| MPI | OpenMP | CUDA | Python | TensorFlow | PyTorch|
+----------------------------------------------------+
|          Resource Manager / Scheduler              |
|                     Slurm                          |
+----------------------------------------------------+
| Authentication | Provisioning | Monitoring         |
| LDAP | xCAT | Prometheus | Grafana | Logging       |
+----------------------------------------------------+
|           Operating System (Linux)                 |
+----------------------------------------------------+
| CPU | GPU | Memory | Network | Storage             |
+----------------------------------------------------+
```

---

## Software Stack Layers

| Layer | Purpose |
|--------|---------|
| Applications | Scientific simulations, AI training, analytics |
| Runtime Libraries | MPI, CUDA, NCCL, OpenMP |
| Scheduler | Allocates compute resources |
| Provisioning | Deploys operating systems and manages nodes |
| Authentication | User identity management |
| Monitoring | Observability and alerting |
| Operating System | Hardware abstraction and resource management |
| Hardware | Physical compute infrastructure |

---

## Why Layered Architecture?

A layered architecture provides:

- Scalability
- Maintainability
- Modularity
- Easier troubleshooting
- Technology independence

For example:

Changing the scheduler does not require replacing the storage system.

Replacing GPUs does not require redesigning authentication.

---

# 2.2 HPC Job Lifecycle

## Concept

Users do not directly execute workloads on compute nodes.

Instead, every workload follows a controlled execution path managed by the scheduler.

This ensures:

- Fair resource allocation
- Security
- Efficient utilization
- Job accounting

---

## Job Lifecycle Overview

```
User

↓

Login Node

↓

Submit Job

↓

Slurm Scheduler

↓

Queue

↓

Resource Allocation

↓

Compute Node

↓

Application Execution

↓

Read / Write Data

↓

Job Completion

↓

Results Returned
```

---

## Step 1 — User Login

Users connect to the login node using SSH.

```
ssh researcher@login01
```

The login node is used for:

- Editing code
- Compiling applications
- Preparing datasets
- Submitting jobs

Heavy computations should **never** run on login nodes.

---

## Step 2 — Job Submission

Users describe resource requirements.

Example:

- CPUs
- Memory
- GPUs
- Runtime
- Partition

The scheduler records the request.

---

## Step 3 — Queue

Jobs enter a waiting queue.

The scheduler determines:

- Priority
- Available resources
- Policies
- Reservations

Not every submitted job starts immediately.

---

## Step 4 — Scheduling

The scheduler evaluates:

- Available nodes
- CPU availability
- GPU availability
- Memory
- Time limits
- Queue priority

The best placement is selected.

---

## Step 5 — Execution

Allocated compute nodes execute the application.

```
Application

↓

CPU

GPU

Memory

Filesystem

Network
```

The user normally never logs into compute nodes manually.

---

## Step 6 — Data Processing

Applications read data from shared storage.

```
Compute Node

↓

Lustre

↓

Input Data

↓

Processing

↓

Output Data
```

Large AI workloads may read terabytes of data during execution.

---

## Step 7 — Completion

The scheduler releases resources.

The cluster becomes available for the next workload.

---

# 2.3 Data Flow Inside an HPC Cluster

Understanding data movement is essential.

Applications spend significant time moving data rather than performing calculations.

---

## Overall Data Flow

```
User

↓

Login Node

↓

Scheduler

↓

Compute Nodes

↓

CPU / GPU

↓

Memory

↓

Network

↓

Shared Storage

↓

Results
```

---

## Detailed Workflow

```
User

↓

Source Code

↓

Compilation

↓

Job Submission

↓

Slurm

↓

Compute Node

↓

Application Starts

↓

Input Dataset

↓

Memory

↓

CPU / GPU

↓

Output

↓

Lustre

↓

User Downloads Results
```

---

## Why Data Flow Matters

Poor data flow leads to:

- Slow jobs
- Network congestion
- Storage bottlenecks
- GPU starvation

Understanding the path helps identify performance issues.

---

# 2.4 Compute Layer (CPU & GPU)

The compute layer performs calculations.

It consists of CPUs and increasingly GPUs.

---

## CPU

The CPU is responsible for:

- Operating system execution
- Scheduling
- MPI communication
- General-purpose computation
- File system operations

```
CPU

↓

Instructions

↓

Execution

↓

Results
```

CPUs are optimized for flexibility and low-latency decision making.

---

## GPU

GPUs accelerate massively parallel workloads.

Typical AI workloads include:

- Matrix multiplication
- Neural network training
- Scientific simulations
- Deep learning inference

```
Application

↓

CUDA

↓

GPU

↓

Thousands of Parallel Threads
```

---

## CPU vs GPU

| CPU | GPU |
|-----|-----|
| Few powerful cores | Thousands of lightweight cores |
| Low latency | High throughput |
| Sequential tasks | Parallel tasks |
| OS management | Numerical acceleration |

Modern HPC clusters frequently combine both.

---

## Heterogeneous Computing

Modern systems combine multiple processor types.

```
Application

↓

CPU

↓

GPU

↓

Accelerated Execution
```

This architecture is known as **heterogeneous computing**.

---

# 2.5 Memory Layer

Memory acts as the workspace for applications.

Without sufficient memory, applications cannot execute efficiently.

---

## Memory Hierarchy

```
Registers

↓

CPU Cache

↓

Main Memory (RAM)

↓

Local SSD

↓

Shared Storage
```

Higher levels are faster but smaller.

Lower levels are larger but slower.

---

## Memory Usage

Applications store:

- Variables
- Matrices
- Neural network parameters
- Temporary buffers
- Input datasets

Large AI models may require hundreds of gigabytes of memory.

---

## Memory Bottlenecks

Common issues include:

- Insufficient RAM
- NUMA imbalance
- Excessive swapping
- Memory fragmentation

These significantly reduce application performance.

---

# 2.6 Storage Layer

Storage holds user data and application outputs.

Unlike desktop systems, HPC clusters use shared storage accessible from every compute node.

---

## Storage Architecture

```
Users

↓

Compute Nodes

↓

High-Speed Network

↓

Parallel File System

↓

Storage Servers
```

---

## Storage Types

| Type | Purpose |
|------|---------|
| Local Disk | Operating system |
| NVMe SSD | Temporary scratch space |
| Parallel Storage | Shared datasets |
| Archive Storage | Long-term retention |

---

## Parallel File Systems

Examples include:

- Lustre
- IBM Spectrum Scale (GPFS)
- BeeGFS

These allow many compute nodes to read and write simultaneously.

---

## Storage Requirements

A production storage system should provide:

- High throughput
- Redundancy
- Scalability
- Fault tolerance
- Shared access

---

# 2.7 Network Fabric

The network fabric connects every component of the cluster.

Without a high-speed network, an HPC cluster cannot function efficiently.

---

## Cluster Network

```
Management Node

↓

Core Switch

↓

Compute Nodes

↓

Storage

↓

GPU Nodes
```

---

## Why Network Performance Matters

Applications frequently exchange:

- MPI messages
- Storage traffic
- GPU communication
- Job control messages

Network performance directly affects overall application speed.

---

## Ethernet vs InfiniBand

| Ethernet | InfiniBand |
|-----------|------------|
| General networking | HPC communication |
| Higher latency | Extremely low latency |
| Lower bandwidth | Very high bandwidth |
| Standard enterprise | Scientific computing |

Dedicated chapters will explore InfiniBand in detail.

---

## Network Components

Typical HPC networks include:

- Management Network
- High-Speed Fabric
- Storage Network
- Out-of-band management

Separating traffic improves reliability and performance.

---

# 2.8 Putting Everything Together

The following diagram illustrates how the software stack, hardware, and users interact.

```
                     Users
                       │
                       ▼
               Login Node
                       │
                       ▼
            Slurm Scheduler
                       │
        ┌──────────────┼──────────────┐
        │              │              │
        ▼              ▼              ▼
   Compute01      Compute02      ComputeN
        │              │              │
        ├──────────────┼──────────────┤
        │
        ▼
   CPU + GPU + Memory
        │
        ▼
 High-Speed Network Fabric
        │
        ▼
 Parallel Storage System
        │
        ▼
  Output Results
```

Every component contributes to successful job execution.

Removing any layer compromises the functionality of the entire cluster.

---

# Production Insight

Production HPC environments rarely experience failures caused by a single component. A user's job may remain pending because no GPUs are available, fail because the shared filesystem is unavailable, or terminate because of insufficient memory. Effective troubleshooting requires understanding how compute, storage, networking, scheduling, and authentication interact as one integrated platform.

---

# Key Takeaways

- HPC clusters are built using a layered software architecture.
- Workloads follow a controlled job lifecycle managed by a scheduler.
- Data movement is as important as computation.
- Modern clusters combine CPUs and GPUs for heterogeneous computing.
- Memory hierarchy directly impacts application performance.
- Parallel storage enables shared, high-throughput access to data.
- High-speed network fabrics minimize communication latency.
- Every layer of the HPC stack depends on the layers beneath it.

---

## Next Part

**Part 3 – Core HPC Technologies**

Topics covered:

- Linux
- xCAT
- Slurm
- LDAP
- Lustre
- InfiniBand
- NVIDIA GPU
