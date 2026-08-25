## Part 1 – InfiniBand Fundamentals

- [4.1 Why InfiniBand?](#41-why-infiniband)
- [4.2 What is InfiniBand?](#42-what-is-infiniband)
- [4.3 InfiniBand Architecture](#43-infiniband-architecture)
- [4.4 InfiniBand Components](#44-infiniband-components)
- [4.5 RDMA (Remote Direct Memory Access)](#45-rdma-remote-direct-memory-access)
- [4.6 InfiniBand vs Ethernet](#46-infiniband-vs-ethernet)
- [4.7 HPC Use Cases](#47-hpc-use-cases)
- [Key Takeaways](#key-takeaways)

---

# 4.1 Why InfiniBand?

In HPC and AI clusters, applications running on multiple compute nodes continuously exchange data. Standard Ethernet is suitable for management traffic, but large-scale scientific computing and distributed AI training demand **very low latency** and **very high bandwidth**.

InfiniBand was designed specifically for these workloads.

Typical workloads include:

- MPI Applications
- AI Model Training
- GPU Clusters
- CFD Simulations
- Weather Forecasting
- Molecular Dynamics

```
          Compute Node 01
                │
                │
        InfiniBand Fabric
                │
                │
          Compute Node 02
```

InfiniBand enables compute nodes to communicate with minimal delay, improving overall application performance.

---

# 4.2 What is InfiniBand?

InfiniBand is a **high-speed networking technology** primarily used in:

- High Performance Computing (HPC)
- Artificial Intelligence (AI)
- High-speed Storage Systems
- Supercomputers

It provides:

- High Bandwidth
- Low Latency
- RDMA Support
- High Scalability

Unlike traditional Ethernet, InfiniBand is optimized for moving data directly between application memory.

---

## Typical Speeds

| Generation | Speed |
|------------|-------|
| SDR | 10 Gbps |
| DDR | 20 Gbps |
| QDR | 40 Gbps |
| FDR | 56 Gbps |
| EDR | 100 Gbps |
| HDR | 200 Gbps |
| NDR | 400 Gbps |

---

# 4.3 InfiniBand Architecture

A simplified InfiniBand network consists of Host Channel Adapters (HCAs), switches, and a Subnet Manager.

```
            +----------------+
            |   OpenSM       |
            | Subnet Manager |
            +--------+-------+
                     |
        ----------------------------
        |            |             |
        ▼            ▼             ▼
     IB Switch    IB Switch    IB Switch
        |            |             |
    -----        -----         -----
    |   |        |   |         |   |
 Compute Compute Compute   Storage
 Node    Node    Node
```

Unlike Ethernet, InfiniBand requires a **Subnet Manager** to initialize and manage the fabric.

---

# 4.4 InfiniBand Components

## Host Channel Adapter (HCA)

The HCA is the InfiniBand network interface installed in servers.

Responsibilities:

- Send/Receive InfiniBand packets
- Support RDMA
- Connect compute nodes to the IB fabric

Example hardware:

- NVIDIA ConnectX Series

```bash
1. List active HCA devices
$ ib_devices

2. View detailed hardware and port status
$ ibv_devinfo
(or)
$ ibstat

3. Check physical PCI layer connection
$ lspci | grep -i mellanox
```

***Note: An InfiniBand Host Channel Adapter (HCA) is the internal network interface card, while an InfiniBand transceiver is the external optical plugin module that connects the HCA to a network cable***
---

## InfiniBand Switch

An InfiniBand switch connects multiple HCAs together.

Functions:

- Packet forwarding
- Fabric connectivity
- High-speed communication

---

## Subnet Manager (SM)

The Subnet Manager configures the InfiniBand fabric.

Responsibilities:

- Discover devices
- Assign Local IDs (LIDs)
- Configure routes
- Monitor fabric topology

A common implementation is:

```
OpenSM
```

Without a running Subnet Manager, the InfiniBand fabric cannot become operational.

---

# 4.5 RDMA (Remote Direct Memory Access)

RDMA is one of the most important features of InfiniBand.

It allows one system to access the memory of another system **without involving the remote CPU**.

Traditional communication:

```
Application

↓

Kernel

↓

TCP/IP Stack

↓

NIC

↓

Network
```

RDMA communication:

```
Application

↓

HCA

↓

InfiniBand Fabric

↓

Remote Memory
```

---

## Benefits

- Extremely low latency
- High throughput
- Lower CPU utilization
- Faster MPI communication

These characteristics make RDMA ideal for HPC and distributed AI workloads.

---

# 4.6 InfiniBand vs Ethernet

| Feature | Ethernet | InfiniBand |
|----------|-----------|------------|
| Primary Use | General Networking | HPC & AI |
| Latency | Higher | Very Low |
| RDMA | Optional (RoCE) | Native |
| CPU Overhead | Higher | Lower |
| Typical HPC Usage | Management Network | Compute Fabric |

---

## HPC Perspective

A typical cluster may use both technologies:

```
Ethernet

↓

SSH

xCAT

LDAP

Management

-------------------

InfiniBand

↓

MPI

GPU Communication

Lustre

Distributed AI
```

Each network serves a different purpose.

---

# 4.7 HPC Use Cases

InfiniBand is commonly used for:

### MPI Applications

Fast communication between compute nodes.

---

### GPU Clusters

Supports high-speed GPU-to-GPU communication.

---

### Parallel Storage

Provides high-performance access to shared storage systems such as Lustre.

---

### AI Training

Large AI models require continuous synchronization between GPUs across multiple servers. InfiniBand reduces communication overhead and improves training efficiency.

---

## Production Note

In many production HPC environments, administrators maintain **two separate networks**:

- **Ethernet** for provisioning, management, SSH, monitoring, and user access.
- **InfiniBand** for application traffic, MPI communication, and high-performance storage access.

Separating these networks improves both performance and reliability.

---

# Key Takeaways

- InfiniBand is a high-speed interconnect designed for HPC and AI environments.
- It provides significantly lower latency than traditional Ethernet.
- Host Channel Adapters (HCAs), switches, and a Subnet Manager form the core of an InfiniBand fabric.
- RDMA enables direct memory access between systems with minimal CPU involvement.
- Most production HPC clusters use Ethernet for management traffic and InfiniBand for high-performance communication.
- Understanding InfiniBand fundamentals is essential before learning configuration, monitoring, and troubleshooting.

---

## Next Part

**Chapter 4 – Part 2**

Topics:

- OpenSM
- Mellanox OFED
- InfiniBand Interfaces
- Essential InfiniBand Commands
- Performance Monitoring
- Fabric Verification
