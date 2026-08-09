## Part 5 – Advanced HPC-AI Troubleshooting & Final Checklist

> **Notebook focus:** Concise troubleshooting patterns for distributed workloads, performance degradation, cross-layer failures, and production recovery.

* [17.51 Performance Degradation](#1751-performance-degradation)
* [17.52 Slow HPC Job](#1752-slow-hpc-job)
* [17.53 MPI Job Failure](#1753-mpi-job-failure)
* [17.54 GPU Performance Problem](#1754-gpu-performance-problem)
* [17.55 Network Performance Problem](#1755-network-performance-problem)
* [17.56 Lustre Performance Problem](#1756-lustre-performance-problem)
* [17.57 Multi-Node Failure Correlation](#1757-multi-node-failure-correlation)
* [17.58 Configuration Drift](#1758-configuration-drift)
* [17.59 Safe Production Changes](#1759-safe-production-changes)
* [17.60 Final Troubleshooting Checklist](#1760-final-troubleshooting-checklist)
* [17.61 Chapter 17 Quick Revision](#1761-chapter-17-quick-revision)



# 17.51 Performance Degradation

A performance problem is different from a hard failure.

Example:

```text
Job completes
      ↓
But takes 2 hours instead of 30 minutes
```

Start with:

```text
Application
     ↓
CPU / GPU
     ↓
Memory
     ↓
Network
     ↓
Storage
     ↓
Scheduler
```

Compare:

```text
Healthy Job
     vs
Slow Job
```

Look for differences rather than guessing.

---

# 17.52 Slow HPC Job

Check Slurm:

```bash
sacct -j <JOBID> \
--format=JobID,State,Elapsed,AllocCPUS,MaxRSS,NodeList
```

Check node:

```bash
uptime
free -h
iostat -xz 1
```

For CPU workload:

```bash
mpstat -P ALL 1
```

For GPU workload:

```bash
nvidia-smi
```

For network-intensive workloads:

```bash
ibstat
rdma link
```

For storage-intensive workloads:

```bash
lfs df -h
iostat -xz 1
```

Possible bottlenecks:

```text
CPU
Memory
GPU
Network
Storage
Application
```

---

# 17.53 MPI Job Failure

Typical symptoms:

```text
MPI initialization failed
Connection failure
Rank failure
Timeout
Application hang
```

Check allocation:

```bash
scontrol show job <JOBID>
```

Check nodes:

```bash
scontrol show hostnames <JOBID>
```

Test connectivity:

```bash
srun hostname
```

For InfiniBand:

```bash
ibstat
ibv_devinfo
```

Check MPI environment:

```bash
which mpirun
which mpiexec
```

Check libraries:

```bash
ldd <application>
```

Potential causes:

```text
Wrong MPI version
Library mismatch
Network failure
InfiniBand problem
DNS/hostname problem
Lustre issue
Environment mismatch
```

---

# 17.54 GPU Performance Problem

A GPU being available does not guarantee good performance.

Check:

```bash
nvidia-smi
```

Monitor:

```bash
nvidia-smi dmon
```

Look at:

```text
GPU utilization
Memory utilization
Power
Temperature
Clocks
```

Check PCIe information:

```bash
nvidia-smi -q | grep -i -A5 "PCI"
```

Possible causes:

```text
CPU bottleneck
PCIe bottleneck
GPU memory pressure
Thermal throttling
Poor application utilization
Data transfer bottleneck
Storage bottleneck
```

Mental model:

```text
CPU
 ↓
PCIe / NVLink
 ↓
GPU Memory
 ↓
GPU Compute
```

---

# 17.55 Network Performance Problem

A network can be operational but slow.

Check:

```bash
ip -s link
```

Look for:

```text
RX errors
TX errors
Dropped packets
```

Ethernet:

```bash
ethtool <interface>
```

InfiniBand:

```bash
ibstat
```

RDMA:

```bash
rdma link
```

Use performance tools where available:

```bash
iperf3
ib_write_bw
ib_read_bw
```

Compare:

```text
Expected bandwidth
        vs
Measured bandwidth
```

Potential causes:

```text
MTU mismatch
Packet loss
Bad cable
Switch issue
Link degradation
Incorrect bonding
Fabric congestion
```

---

# 17.56 Lustre Performance Problem

Check filesystem:

```bash
lfs df -h
```

Check stripe:

```bash
lfs getstripe <path>
```

Check client I/O:

```bash
iostat -xz 1
```

Look for:

```text
High latency
Low throughput
High utilization
```

Potential causes:

```text
OST imbalance
Poor striping
Network congestion
Backend storage load
Metadata bottleneck
Small I/O workload
```

Mental model:

```text
Application
     ↓
Lustre Client
     ↓
Network
     ↓
MDT / OST
     ↓
Backend Storage
```

---

# 17.57 Multi-Node Failure Correlation

If 50 nodes fail simultaneously:

**Do not start with node 1.**

Ask:

```text
What do all affected nodes share?
```

Check:

```text
Common switch
Common rack
Common power domain
Common network
Common LDAP
Common DNS
Common filesystem
Common Slurm configuration
Common software update
```

Example:

```text
Node 1 ─┐
Node 2 ─┤
Node 3 ─┤── Same switch
Node 4 ─┘
```

This strongly suggests investigating the switch/fabric path before individual nodes.

---

# 17.58 Configuration Drift

Two supposedly identical nodes may behave differently.

Compare:

```bash
uname -r
rpm -qa
ip addr
ip route
systemctl --failed
```

GPU nodes:

```bash
nvidia-smi
```

InfiniBand:

```bash
ibstat
```

Slurm:

```bash
slurmd -C
```

Compare configuration files:

```bash
diff node1.conf node2.conf
```

Typical drift:

```text
Kernel
Driver
Package
Slurm configuration
Network configuration
Mounts
Environment
Permissions
```

### Production principle

> **Standardized nodes reduce troubleshooting complexity.**

---

# 17.59 Safe Production Changes

Before making a change:

```text
1. Understand the problem
2. Collect evidence
3. Identify impact
4. Check maintenance/change policy
5. Make the smallest change
6. Validate
7. Document
```

Avoid unnecessary actions such as:

```text
Rebooting all nodes
Restarting unrelated services
Changing cluster-wide configuration
Reinstalling drivers blindly
Deleting logs
```

Prefer:

```text
One node
    ↓
Validate
    ↓
Small batch
    ↓
Validate
    ↓
Cluster-wide
```

---

# 17.60 Final Troubleshooting Checklist

## Linux

```text
☐ CPU
☐ Memory
☐ Disk
☐ Processes
☐ Services
☐ Kernel
```

## Network

```text
☐ Interface
☐ IP
☐ Route
☐ DNS
☐ MTU
☐ Port
```

## InfiniBand

```text
☐ HCA
☐ Port ACTIVE
☐ OpenSM
☐ RDMA
☐ Fabric
```

## GPU

```text
☐ PCIe detection
☐ Driver
☐ nvidia-smi
☐ GPU utilization
☐ Slurm GRES
```

## Storage

```text
☐ Mount
☐ Capacity
☐ Quota
☐ I/O
☐ Lustre health
```

## Slurm

```text
☐ Node state
☐ slurmd
☐ Job state
☐ Partition
☐ Account
☐ QoS
```

## Authentication

```text
☐ NSS
☐ SSSD
☐ LDAP
☐ PAM
☐ SSH
```

---

# 17.61 Chapter 17 Quick Revision

The complete troubleshooting model:

```text
                    Incident
                       │
                       ▼
                     Scope
                       │
                       ▼
                    Evidence
                       │
                       ▼
              Identify failing layer
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
      Linux          Network        Storage
        │              │              │
        ▼              ▼              ▼
      Slurm             IB           Lustre
        │
        ▼
       GPU
        │
        ▼
     Application
        │
        ▼
      Root Cause
        │
        ▼
       Fix
        │
        ▼
     Validate
        │
        ▼
    Document
```

### Golden Rules

```text
1. Scope before debugging.
2. Collect evidence before changing anything.
3. Separate symptom from root cause.
4. Troubleshoot layer by layer.
5. Compare healthy vs failed nodes.
6. Treat shared failures as infrastructure problems.
7. Make the smallest safe production change.
8. Validate with a real health check or workload.
9. Document the root cause.
10. Prevent recurrence where possible.
```

> **HPC-AI Infrastructure Engineer mindset:** Don't ask only **"What command fixes this?"** Ask **"Which layer is failing, what evidence proves it, and what is the safest way to restore service?"**

# End of Chapter 17 – Part 5
