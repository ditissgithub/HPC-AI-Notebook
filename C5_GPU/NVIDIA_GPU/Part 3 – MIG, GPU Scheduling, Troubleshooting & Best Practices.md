## Part 3 – MIG, GPU Scheduling, Troubleshooting & Best Practices

- [5.16 Multi-Instance GPU (MIG)](#516-multi-instance-gpu-mig)
- [5.17 GPU Scheduling with Slurm](#517-gpu-scheduling-with-slurm)
- [5.18 GPU Allocation](#518-gpu-allocation)
- [5.19 NVIDIA GPU Containers](#519-nvidia-gpu-containers)
- [5.20 Common GPU Problems](#520-common-gpu-problems)
- [5.21 Production GPU Troubleshooting](#521-production-gpu-troubleshooting)
- [5.22 GPU Best Practices](#522-gpu-best-practices)
- [5.23 Interview Questions](#523-interview-questions)
- [5.24 GPU Command Cheat Sheet](#524-gpu-command-cheat-sheet)
- [5.25 Chapter Summary](#525-chapter-summary)

---

# 5.16 Multi-Instance GPU (MIG)

**MIG (Multi-Instance GPU)** allows supported NVIDIA GPUs to be partitioned into multiple isolated GPU instances.

Instead of assigning an entire physical GPU to one workload:

```text
Physical GPU
     │
     └── One Workload
```

MIG allows:

```text
Physical GPU
     │
 ┌───┼───┬───┐
 ▼   ▼   ▼   ▼
MIG MIG MIG MIG
 1   2   3   4
```

Each MIG instance receives dedicated portions of GPU resources such as compute and memory.

---

## Why MIG?

MIG is useful when workloads do not require a complete GPU.

Example:

```text
GPU = 80 GB

Job A → 20 GB
Job B → 20 GB
Job C → 20 GB
Job D → 20 GB
```

Instead of leaving unused resources on a large GPU, multiple workloads can run concurrently using supported MIG configurations.

---

## MIG Requirements

MIG support depends on:

- GPU model
- NVIDIA driver
- GPU firmware
- Supported software stack

Not every NVIDIA GPU supports MIG.

---

## Check MIG Capability

```bash
nvidia-smi -q | grep -i MIG
```

---

## Enable MIG

On supported GPUs:

```bash
nvidia-smi -i 0 -mig 1
```

Verify:

```bash
nvidia-smi
```

**Production Note:** Enabling or changing MIG configuration can affect running workloads and may require GPU reset or reboot depending on the GPU and software environment.

---

# 5.17 GPU Scheduling with Slurm

In HPC environments, users normally do not directly select GPUs using `nvidia-smi`.

Instead, a workload manager such as **Slurm** allocates GPUs.

```text
User

↓

Slurm

↓

GPU Allocation

↓

Compute Node

↓

CUDA Application
```

Slurm tracks available GPU resources and prevents multiple jobs from accidentally using the same GPU.

---

## Example Job Request

```bash
srun --gres=gpu:1 nvidia-smi
```

This requests one GPU for the job.

Depending on the configured Slurm version and cluster configuration, GPU resources can also be requested using the newer `--gpus` options.

Example:

```bash
srun --gpus=1 nvidia-smi
```

---

## GPU Partition

A cluster may define a dedicated GPU partition:

```text
Partition

gpu

Nodes

gpu001
gpu002
gpu003
```

Users submit jobs to the partition:

```bash
sbatch -p gpu job.sh
```

---

# 5.18 GPU Allocation

Slurm uses configured GPU resources to control job access.

Common concepts include:

- GRES
- GPU Type
- GPU Count
- GPU Binding
- GPU Allocation

---

## GRES

**GRES = Generic RESources**

GPU resources are commonly configured through GRES.

Example:

```text
NodeName=gpu001 Gres=gpu:a100:4
```

The exact configuration depends on the Slurm version and cluster design.

---

## Request Specific GPU Type

Example:

```bash
srun --gres=gpu:a100:1 hostname
```

or:

```bash
srun --gpus=a100:1 nvidia-smi
```

---

## Check Allocated Resources

```bash
scontrol show job JOB_ID
```

Check node:

```bash
scontrol show node gpu001
```

---

## Check GPU Visibility

Inside the allocated job:

```bash
echo $CUDA_VISIBLE_DEVICES
```

Slurm and the CUDA software stack can use environment variables to restrict which GPUs the application sees.

---

# 5.19 NVIDIA GPU Containers

Modern AI clusters frequently run GPU workloads inside containers.

Typical architecture:

```text
Application Container
        │
        ▼
CUDA Libraries
        │
        ▼
NVIDIA Container Runtime
        │
        ▼
Host NVIDIA Driver
        │
        ▼
GPU
```

The GPU driver normally remains on the **host**.

The container provides the user-space libraries required by the application.

---

## Container GPU Test

With a properly configured NVIDIA container environment, a GPU-enabled container can run:

```bash
nvidia-smi
```

The container should see only the GPUs assigned to it.

---

## HPC Perspective

GPU containers are commonly used with:

- Docker
- Podman
- Enroot
- Pyxis
- Kubernetes
- NVIDIA Container Toolkit

For Slurm-based HPC systems, **Enroot + Pyxis** is a common approach for running containerized AI workloads.

---

# 5.20 Common GPU Problems

## Problem 1 – `nvidia-smi` Fails

Example:

```text
NVIDIA-SMI has failed because it couldn't communicate with the NVIDIA driver.
```

Check:

```bash
lsmod | grep nvidia
```

```bash
dmesg | grep -i nvidia
```

```bash
lspci | grep -i nvidia
```

Possible causes:

- Driver not loaded
- Driver/kernel mismatch
- Failed module build
- GPU hardware issue

---

## Problem 2 – GPU Not Detected

Check:

```bash
lspci | grep -i nvidia
```

If the GPU is not visible at PCIe level, investigate:

- Hardware
- PCIe slot
- BIOS configuration
- Power
- Firmware

---

## Problem 3 – CUDA Application Cannot Run

Check:

```bash
nvidia-smi
```

Then:

```bash
nvcc --version
```

Also check the application's CUDA requirements.

Possible causes:

- Incompatible driver
- Missing CUDA libraries
- Incorrect environment
- Container/runtime mismatch

---

## Problem 4 – GPU Memory Exhausted

Check:

```bash
nvidia-smi
```

Identify processes consuming memory.

Possible causes:

- Large model
- Memory leak
- Multiple processes sharing a GPU
- Incorrect GPU allocation

---

## Problem 5 – Low GPU Utilization

Symptoms:

```text
GPU Utilization: 10%
```

Possible causes:

- CPU bottleneck
- Data loading bottleneck
- Storage bottleneck
- Network communication bottleneck
- Small workload
- Synchronization overhead

Do not assume that low GPU utilization means the GPU is faulty.

---

# 5.21 Production GPU Troubleshooting

Use a layered troubleshooting approach.

```text
GPU Hardware
     │
     ▼
PCIe Detection
     │
     ▼
NVIDIA Driver
     │
     ▼
CUDA
     │
     ▼
Slurm Allocation
     │
     ▼
Container Runtime
     │
     ▼
Application
```

---

## Example Scenario

### Problem

A user reports:

> "My Slurm GPU job starts, but the application cannot see the GPU."

### Step 1 – Check allocation

```bash
scontrol show job JOB_ID
```

---

### Step 2 – Check GPU visibility

```bash
echo $CUDA_VISIBLE_DEVICES
```

---

### Step 3 – Check GPU

```bash
nvidia-smi
```

---

### Step 4 – Check CUDA

```bash
nvcc --version
```

---

### Step 5 – If using containers

Test:

```bash
nvidia-smi
```

inside the container.

---

## Possible Root Causes

```text
Slurm allocation incorrect
        OR
CUDA_VISIBLE_DEVICES incorrect
        OR
Container GPU runtime incorrect
        OR
Driver/CUDA compatibility issue
```

---

# 5.22 GPU Best Practices

- Standardize NVIDIA driver versions across GPU node groups.
- Validate driver and CUDA compatibility before deployment.
- Keep GPU firmware consistent.
- Use Slurm to allocate GPUs instead of manual GPU selection.
- Monitor temperature, power, memory, and ECC health.
- Test GPUs after provisioning.
- Avoid changing driver versions on production nodes without validation.
- Maintain a known-good GPU node image.
- Validate GPU visibility inside containers.
- Monitor GPU utilization at cluster level.
- Document GPU models and their capabilities.

---

# 5.23 Interview Questions

## Basic

1. What is a GPU?
2. What is CUDA?
3. What does `nvidia-smi` do?
4. What is NVML?
5. What is GPU memory?

---

## Intermediate

1. Explain the difference between NVIDIA Driver and CUDA Toolkit.
2. What is MIG?
3. What is GPU scheduling?
4. What is Slurm GRES?
5. What is `CUDA_VISIBLE_DEVICES`?
6. How do you identify GPU processes?

---

## Advanced

1. `nvidia-smi` fails after a kernel upgrade. How would you troubleshoot it?
2. A GPU is visible through `lspci` but not through `nvidia-smi`. What could be wrong?
3. A Slurm job receives a GPU but the application cannot see it. How would you troubleshoot it?
4. GPU utilization is only 20% during distributed training. What would you investigate?
5. Explain the relationship between Slurm, CUDA, NVIDIA Driver, NCCL, InfiniBand, and GPUs.
6. How would you validate a newly provisioned GPU compute node?

---

# 5.24 GPU Command Cheat Sheet

## Hardware

```bash
lspci | grep -i nvidia
```

## GPU Status

```bash
nvidia-smi
```

## List GPUs

```bash
nvidia-smi -L
```

## Detailed Information

```bash
nvidia-smi -q
```

## GPU Topology

```bash
nvidia-smi topo -m
```

## GPU Processes

```bash
nvidia-smi pmon -c 1
```

## Driver

```bash
lsmod | grep nvidia
```

```bash
nvidia-smi --query-gpu=driver_version --format=csv,noheader
```

## CUDA

```bash
nvcc --version
```

## GPU Devices

```bash
ls -l /dev/nvidia*
```

## Kernel Logs

```bash
dmesg | grep -i nvidia
```

## Slurm GPU Allocation

```bash
srun --gpus=1 nvidia-smi
```

```bash
scontrol show job JOB_ID
```

---

# 5.25 Chapter Summary

NVIDIA GPUs are a critical component of modern HPC-AI infrastructure.

This chapter covered:

- GPU architecture
- CUDA
- NVIDIA drivers
- CUDA Toolkit
- `nvidia-smi`
- NVML
- GPU monitoring
- GPU health checks
- MIG
- Slurm GPU scheduling
- GRES
- GPU allocation
- `CUDA_VISIBLE_DEVICES`
- GPU containers
- Common GPU failures
- Production troubleshooting
- GPU best practices
- Interview questions
- Essential GPU commands

The important operational model is:

```text
                  GPU Infrastructure
                         │
        ┌────────────────┼────────────────┐
        ▼                ▼                ▼
     Hardware         Software         Scheduler
        │                │                │
       GPU          Driver + CUDA       Slurm
        │                │                │
        └────────────────┼────────────────┘
                         ▼
                   AI / HPC Workload
                         │
                         ▼
                 InfiniBand / Lustre
```

An HPC-AI Infrastructure Engineer should be able to troubleshoot the complete path:

```text
GPU Hardware
    ↓
PCIe
    ↓
NVIDIA Driver
    ↓
CUDA
    ↓
Slurm
    ↓
Container Runtime
    ↓
Application
    ↓
NCCL / MPI
    ↓
InfiniBand
    ↓
Lustre / Storage
```

This end-to-end understanding is more valuable in production than knowing individual GPU commands in isolation.

---

# Chapter Completion Checklist

You should now be able to:

- Explain the role of NVIDIA GPUs in HPC and AI.
- Understand the NVIDIA driver and CUDA software stack.
- Use `nvidia-smi` for GPU administration.
- Perform basic GPU health checks.
- Explain MIG and its purpose.
- Understand Slurm GPU allocation and GRES.
- Understand GPU visibility through `CUDA_VISIBLE_DEVICES`.
- Troubleshoot GPU problems systematically.
- Understand GPU containers at a high level.
- Connect GPU troubleshooting with InfiniBand, Slurm, and storage.
- Use essential NVIDIA GPU commands confidently.

---

# Next Chapter

**Chapter 6 – xCAT**

Topics:

- xCAT Architecture
- xCAT Database
- DHCP & DNS
- Node Discovery
- PXE / Network Boot
- Stateless & Stateful Provisioning
- OS Images
- Compute Node Provisioning
- Post-Provisioning
- Production Troubleshooting
- xCAT Commands
- Interview Questions
