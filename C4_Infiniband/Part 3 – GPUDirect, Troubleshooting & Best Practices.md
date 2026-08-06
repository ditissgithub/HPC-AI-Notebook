## Part 3 – GPUDirect, Troubleshooting & Best Practices

- [4.14 GPUDirect RDMA](#414-gpudirect-rdma)
- [4.15 Common InfiniBand Problems](#415-common-infiniband-problems)
- [4.16 Production Troubleshooting](#416-production-troubleshooting)
- [4.17 Best Practices](#417-best-practices)
- [4.18 Interview Questions](#418-interview-questions)
- [4.19 InfiniBand Command Cheat Sheet](#419-infiniband-command-cheat-sheet)
- [4.20 Chapter Summary](#420-chapter-summary)

---

# 4.14 GPUDirect RDMA

Modern AI clusters often contain hundreds or thousands of GPUs working together. Efficient communication between GPUs is critical for distributed training.

**GPUDirect RDMA** allows an InfiniBand adapter (HCA) to access GPU memory directly without unnecessary copies through CPU memory.

Traditional Data Path

```
GPU Memory

↓

CPU Memory

↓

InfiniBand Adapter

↓

Network

↓

Remote Server
```

GPUDirect RDMA

```
GPU Memory

↓

InfiniBand Adapter

↓

Network

↓

Remote GPU
```

---

## Benefits

- Lower latency
- Higher bandwidth
- Reduced CPU utilization
- Faster distributed AI training
- Better MPI and NCCL performance

---

## HPC & AI Perspective

GPUDirect RDMA is commonly used with:

- NVIDIA CUDA
- NCCL
- MPI
- Large Language Model (LLM) Training
- Multi-node GPU Clusters

---

# 4.15 Common InfiniBand Problems

Even a healthy cluster can occasionally experience InfiniBand issues. A structured troubleshooting approach is essential.

---

## Problem 1 – Port State Down

Symptoms

- `ibstat` shows **Down**
- MPI jobs fail
- No communication between nodes

Check

```bash
ibstat
```

Possible Causes

- Cable disconnected
- HCA disabled
- Switch issue

---

## Problem 2 – Port Stuck in Initializing

Symptoms

```
State : Initializing
```

Check

```bash
systemctl status opensm
```

Possible Causes

- OpenSM not running
- Fabric not initialized
- Switch connectivity issue

---

## Problem 3 – HCA Not Detected

Check

```bash
lspci | grep Mellanox
```

```bash
ibv_devices
```

Possible Causes

- Driver not loaded
- OFED mismatch
- Hardware failure

---

## Problem 4 – Poor Performance

Symptoms

- Slow MPI jobs
- Low bandwidth
- High latency

Check

```bash
ib_write_bw
```

```bash
ib_read_bw
```

```bash
perfquery
```

Possible Causes

- Link speed mismatch
- Congestion
- Incorrect firmware
- Hardware errors

---

## Problem 5 – MPI Communication Failure

Check

```bash
ibping
```

```bash
ibnetdiscover
```

Possible Causes

- Inactive port
- Routing problem
- Switch failure
- Incorrect MPI configuration

---

# 4.16 Production Troubleshooting

Use a systematic workflow rather than changing configurations randomly.

```
Application Failure

↓

Check HCA

↓

Check Interface

↓

Check OpenSM

↓

Check Fabric

↓

Check Performance

↓

Identify Root Cause
```

---

## Step 1 – Verify Hardware

```bash
lspci | grep Mellanox
```

---

## Step 2 – Verify Driver

```bash
ofed_info
```

```bash
lsmod | grep mlx
```

---

## Step 3 – Verify Interface

```bash
ibstat
```

---

## Step 4 – Verify Fabric

```bash
ibnetdiscover
```

---

## Step 5 – Verify Switches

```bash
ibswitches
```

---

## Step 6 – Verify Performance

```bash
ib_write_bw
```

```bash
ib_read_bw
```

---

## Real Production Example

### Scenario

Users report that distributed MPI jobs are hanging.

### Investigation

```bash
ibstat
```

Port status:

```
Initializing
```

Check OpenSM

```bash
systemctl status opensm
```

Result

```
opensm.service

Inactive
```

Solution

```bash
systemctl start opensm
```

Recheck

```bash
ibstat
```

Port becomes

```
Active
```

MPI jobs execute successfully.

---

# 4.17 Best Practices

- Keep OFED and firmware versions compatible.
- Monitor InfiniBand fabric health regularly.
- Maintain a single active Subnet Manager with standby redundancy.
- Label cables and switch ports clearly.
- Document fabric topology.
- Test bandwidth after maintenance.
- Monitor port error counters.
- Keep firmware updated following vendor recommendations.
- Separate management traffic from InfiniBand traffic.
- Validate the fabric before accepting production workloads.

---

# 4.18 Interview Questions

## Basic

1. What is InfiniBand?
2. Why is InfiniBand preferred over Ethernet in HPC?
3. What is an HCA?
4. What is OpenSM?
5. What is RDMA?

---

## Intermediate

1. Explain the role of a Subnet Manager.
2. What is Mellanox OFED?
3. What does `ibstat` display?
4. What is the purpose of `ibnetdiscover`?
5. What is GPUDirect RDMA?

---

## Advanced

1. A node remains in the **Initializing** state. How would you troubleshoot it?
2. How would you identify an InfiniBand performance bottleneck?
3. Explain the relationship between CUDA, NCCL, RDMA, and InfiniBand.
4. What checks would you perform after replacing an HCA?
5. How would you validate a newly deployed InfiniBand fabric?

---

# 4.19 InfiniBand Command Cheat Sheet

## Device Information

```bash
ibstat
```

```bash
ibv_devices
```

```bash
ibv_devinfo
```

---

## Hardware

```bash
lspci | grep Mellanox
```

```bash
ethtool -i ib0
```

---

## Fabric

```bash
ibhosts
```

```bash
ibswitches
```

```bash
ibnetdiscover
```

```bash
iblinkinfo
```

---

## Performance

```bash
ib_write_bw
```

```bash
ib_read_bw
```

```bash
ib_send_bw
```

```bash
perfquery
```

---

## Services

```bash
systemctl status opensm
```

```bash
journalctl -u opensm
```

---

## Linux

```bash
ip addr show ib0
```

```bash
ip -s link show ib0
```

---

# 4.20 Chapter Summary

This chapter introduced the essential concepts of InfiniBand used in HPC and AI infrastructure.

Topics covered:

- InfiniBand architecture
- Host Channel Adapters (HCAs)
- OpenSM and the Subnet Manager
- Mellanox OFED
- RDMA fundamentals
- GPUDirect RDMA
- Essential administration commands
- Performance monitoring
- Fabric verification
- Production troubleshooting
- Best practices
- Interview preparation

InfiniBand is the foundation of high-performance communication in modern HPC and AI clusters. Understanding its architecture, administration tools, and troubleshooting techniques is essential for managing production environments efficiently.

---

# Chapter Completion Checklist

You should now be able to:

- Explain the purpose of InfiniBand in HPC and AI clusters.
- Describe the roles of HCAs, switches, and OpenSM.
- Understand RDMA and GPUDirect RDMA concepts.
- Verify the health of an InfiniBand fabric.
- Use common InfiniBand administration and diagnostic commands.
- Troubleshoot common connectivity and performance issues.
- Answer interview questions related to InfiniBand administration.

---

# Next Chapter

**Chapter 5 – NVIDIA GPU**

Topics include:

- GPU Architecture
- CUDA
- NVIDIA Drivers
- `nvidia-smi`
- NVML
- MIG (Multi-Instance GPU)
- GPU Scheduling
- GPU Monitoring
- Production Troubleshooting
- Interview Questions
