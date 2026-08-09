# Chapter 5 – NVIDIA GPU & AI Accelerators

This chapter covers GPU and accelerator infrastructure used in modern HPC-AI clusters.

## Topics Covered

* NVIDIA GPU Architecture
* NVIDIA Driver & CUDA
* `nvidia-smi` & NVML
* GPU Monitoring & Health Checks
* MIG
* Slurm GPU Scheduling & GRES
* GPU Containers
* GPUDirect RDMA
* AMD Instinct & ROCm
* Intel Data Center GPUs & oneAPI
* Intel Gaudi / Habana
* Google TPU
* AWS Trainium & Inferentia
* Multi-Vendor Accelerator Architecture
* Production Troubleshooting
* GPU/Accelerator Commands
* Interview Questions

## Engineer Perspective

The goal is to understand the complete accelerator stack:

```text
Hardware
   ↓
Driver
   ↓
Vendor Runtime
   ↓
Communication Library
   ↓
Framework
   ↓
Slurm / Kubernetes
   ↓
HPC-AI Workload
```

The focus is on **operating, monitoring, scheduling, troubleshooting, and integrating accelerators in production HPC-AI infrastructure**.
