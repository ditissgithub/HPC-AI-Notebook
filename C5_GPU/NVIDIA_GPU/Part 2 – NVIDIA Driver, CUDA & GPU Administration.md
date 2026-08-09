## Part 2 – NVIDIA Driver, CUDA & GPU Administration

- [5.8 NVIDIA Driver](#58-nvidia-driver)
- [5.9 CUDA Toolkit](#59-cuda-toolkit)
- [5.10 Driver vs CUDA Toolkit](#510-driver-vs-cuda-toolkit)
- [5.11 nvidia-smi](#511-nvidia-smi)
- [5.12 NVIDIA Management Library (NVML)](#512-nvidia-management-library-nvml)
- [5.13 GPU Monitoring](#513-gpu-monitoring)
- [5.14 GPU Health Checks](#514-gpu-health-checks)
- [5.15 Essential GPU Commands](#515-essential-gpu-commands)
- [Key Takeaways](#key-takeaways)

---

# 5.8 NVIDIA Driver

The **NVIDIA driver** allows the Linux operating system and applications to communicate with NVIDIA GPUs.

Simplified architecture:

```text
Application
     │
     ▼
CUDA / GPU Libraries
     │
     ▼
NVIDIA Driver
     │
     ▼
GPU Hardware
```

Without a working driver, the GPU may be physically detected by PCIe but will not be usable by CUDA applications.

---

## Verify GPU Detection

```bash
lspci | grep -i nvidia
```

Example:

```text
NVIDIA Corporation GA100 [A100]
```

---

## Check NVIDIA Driver

```bash
nvidia-smi
```

Typical output contains:

```text
Driver Version
CUDA Version
GPU Name
GPU Memory
GPU Utilization
Temperature
Power
Processes
```

---

## Check Loaded NVIDIA Modules

```bash
lsmod | grep nvidia
```

Common modules include:

```text
nvidia
nvidia_drm
nvidia_modeset
nvidia_uvm
```

---

## Driver Installation Considerations

Before installing or upgrading a production NVIDIA driver, verify:

- Linux kernel version
- GPU model
- NVIDIA driver version
- CUDA requirements
- Existing workloads
- DKMS/module support
- Secure Boot configuration, if enabled

A driver upgrade should be treated as a **cluster change**, not simply as a package installation.

---

# 5.9 CUDA Toolkit

**CUDA** is NVIDIA's platform for GPU-accelerated computing.

The CUDA Toolkit provides development and runtime components used to build CUDA applications.

Typical components include:

- `nvcc`
- CUDA runtime libraries
- CUDA libraries
- Development headers
- Profiling and debugging tools

---

## Verify CUDA Compiler

```bash
nvcc --version
```

or:

```bash
nvcc -V
```

---

## CUDA Installation vs Driver

A common mistake is assuming that:

```bash
nvidia-smi
```

proves that the CUDA Toolkit is installed.

It does not.

`nvidia-smi` primarily verifies the NVIDIA driver and GPU management interface.

Check separately:

```bash
which nvcc
```

```bash
nvcc --version
```

---

# 5.10 Driver vs CUDA Toolkit

This distinction is extremely important for GPU administrators.

```text
GPU Hardware
     │
     ▼
NVIDIA Driver
     │
     ├── GPU Management
     │
     └── CUDA Driver API
     
CUDA Toolkit
     │
     ├── nvcc
     ├── Libraries
     └── Development Tools
```

The **driver** communicates with the GPU.

The **CUDA Toolkit** provides tools and libraries used to develop and run CUDA applications.

---

## Compatibility Concept

A newer CUDA application generally requires a sufficiently new NVIDIA driver.

Therefore, before deploying a CUDA version across a cluster, verify the supported driver range.

Example workflow:

```text
Application Requirement

        ↓

CUDA Version

        ↓

Required Driver

        ↓

Installed Driver

        ↓

GPU Compatibility
```

---

# 5.11 nvidia-smi

`nvidia-smi` is one of the most important commands for an HPC-AI Infrastructure Engineer.

Basic command:

```bash
nvidia-smi
```

It provides a snapshot of GPU health and utilization.

---

## Typical Information

```text
GPU
Name
Persistence Mode
Temperature
Power Usage
Memory Usage
GPU Utilization
Compute Processes
```

---

## List GPUs

```bash
nvidia-smi -L
```

Example:

```text
GPU 0: NVIDIA A100-SXM4-80GB
GPU 1: NVIDIA A100-SXM4-80GB
```

---

## Continuous Monitoring

```bash
watch -n 1 nvidia-smi
```

Useful when investigating:

- GPU utilization
- Memory consumption
- Temperature
- Running processes

---

## Query Specific Metrics

```bash
nvidia-smi --query-gpu=name,temperature.gpu,utilization.gpu,memory.used,memory.total --format=csv
```

---

## GPU Processes

```bash
nvidia-smi
```

Look at the process section to identify applications using the GPU.

---

## Resetting a GPU

In controlled maintenance situations, GPU reset may be possible:

```bash
nvidia-smi --gpu-reset -i 0
```

**Production caution:** Do not reset a GPU while active workloads are using it. Verify that no important processes are running first.

---

# 5.12 NVIDIA Management Library (NVML)

**NVML (NVIDIA Management Library)** provides programmatic access to NVIDIA GPU management and monitoring information.

It can expose information such as:

- GPU utilization
- Memory utilization
- Temperature
- Power usage
- Fan speed
- Process information
- ECC information
- GPU topology

---

## Relationship

```text
Monitoring Application
        │
        ▼
       NVML
        │
        ▼
NVIDIA Driver
        │
        ▼
       GPU
```

`nvidia-smi` itself uses NVIDIA's management interfaces, including NVML, to obtain GPU information.

---

## HPC Monitoring

NVML can be used by monitoring systems to collect GPU metrics automatically.

Example:

```text
GPU Node

   │
   ├── NVML
   │
   ├── GPU Metrics
   │
   └── Monitoring System
```

This is useful for cluster-wide GPU monitoring rather than manually running `nvidia-smi` on every node.

---

# 5.13 GPU Monitoring

GPU monitoring should cover more than utilization.

Important metrics include:

| Metric | Why It Matters |
|---|---|
| GPU Utilization | Compute activity |
| Memory Usage | Capacity consumption |
| Temperature | Thermal health |
| Power | Power/thermal behavior |
| ECC Errors | GPU memory reliability |
| PCIe Link | Host-GPU connectivity |
| Processes | Workload identification |

---

## Basic Monitoring

```bash
nvidia-smi
```

---

## Continuous Monitoring

```bash
watch -n 1 nvidia-smi
```

---

## CSV Monitoring

```bash
nvidia-smi \
--query-gpu=timestamp,name,temperature.gpu,utilization.gpu,memory.used,memory.total,power.draw \
--format=csv
```

This format is useful for scripting and automation.

---

# 5.14 GPU Health Checks

A production GPU node should be checked at several layers.

```text
PCIe
  │
  ▼
NVIDIA Driver
  │
  ▼
GPU Detection
  │
  ▼
CUDA
  │
  ▼
Application
```

---

## Step 1 – PCIe Detection

```bash
lspci | grep -i nvidia
```

---

## Step 2 – Driver

```bash
nvidia-smi
```

---

## Step 3 – GPU Inventory

```bash
nvidia-smi -L
```

---

## Step 4 – CUDA Toolkit

```bash
nvcc --version
```

---

## Step 5 – GPU Utilization

```bash
nvidia-smi
```

---

## Step 6 – Kernel Messages

```bash
dmesg | grep -i nvidia
```

or:

```bash
journalctl -k | grep -i nvidia
```

---

# 5.15 Essential GPU Commands

## Hardware Detection

```bash
lspci | grep -i nvidia
```

---

## GPU Status

```bash
nvidia-smi
```

---

## GPU List

```bash
nvidia-smi -L
```

---

## Driver Version

```bash
nvidia-smi --query-gpu=driver_version --format=csv,noheader
```

---

## GPU Information

```bash
nvidia-smi -q
```

---

## Specific GPU

```bash
nvidia-smi -i 0
```

---

## GPU Processes

```bash
nvidia-smi pmon -c 1
```

---

## GPU Topology

```bash
nvidia-smi topo -m
```

This is particularly important in multi-GPU HPC systems because GPU-to-GPU connectivity can influence application performance.

---

## CUDA

```bash
nvcc --version
```

---

## Kernel Modules

```bash
lsmod | grep nvidia
```

---

## NVIDIA Devices

```bash
ls -l /dev/nvidia*
```

---

# HPC Production Check

For a newly provisioned GPU compute node, a basic validation sequence can be:

```bash
lspci | grep -i nvidia

nvidia-smi

nvidia-smi -L

nvcc --version

nvidia-smi topo -m

lsmod | grep nvidia
```

Expected result:

```text
PCIe GPU detected
        ↓
Driver loaded
        ↓
nvidia-smi works
        ↓
GPU visible
        ↓
CUDA environment verified
        ↓
Topology verified
```

---

# Key Takeaways

- The NVIDIA driver provides the primary interface between Linux and the GPU.
- The CUDA Toolkit provides development tools and libraries.
- `nvidia-smi` is the first command to use when diagnosing GPU issues.
- NVML enables programmatic GPU monitoring.
- GPU health checks should cover PCIe, driver, CUDA, hardware, and workload layers.
- GPU utilization alone does not determine GPU health.
- `nvidia-smi topo -m` is valuable when investigating multi-GPU communication and topology.
- Driver and CUDA compatibility must be verified before production deployment.

---

## Next Part

**Chapter 5 – Part 3**

Topics:

- MIG
- GPU Scheduling with Slurm
- GPU Allocation
- NVIDIA Device Plugin
- GPU Containers
- Common GPU Problems
- Production Troubleshooting
- Best Practices
- Interview Questions
- GPU Command Cheat Sheet
- Chapter Summary
