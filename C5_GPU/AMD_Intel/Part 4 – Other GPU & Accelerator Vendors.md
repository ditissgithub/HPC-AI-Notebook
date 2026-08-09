## Part 4 – Other GPU & Accelerator Vendors

- [5.26 Why Learn Other GPU Vendors?](#526-why-learn-other-gpu-vendors)
- [5.27 AMD Instinct](#527-amd-instinct)
- [5.28 AMD ROCm](#528-amd-rocm)
- [5.29 Intel Data Center GPUs](#529-intel-data-center-gpus)
- [5.30 Intel Gaudi AI Accelerators](#530-intel-gaudi-ai-accelerators)
- [5.31 Habana – Historical & Current Context](#531-habana--historical--current-context)
- [5.32 Google TPU](#532-google-tpu)
- [5.33 AWS AI Accelerators](#533-aws-ai-accelerators)
- [5.34 Other Important AI Accelerators](#534-other-important-ai-accelerators)
- [5.35 NVIDIA vs AMD vs Intel vs TPU](#535-nvidia-vs-amd-vs-intel-vs-tpu)
- [5.36 Accelerator Software Stack](#536-accelerator-software-stack)
- [5.37 Multi-Vendor HPC-AI Architecture](#537-multi-vendor-hpc-ai-architecture)
- [5.38 Multi-Vendor Troubleshooting](#538-multi-vendor-troubleshooting)
- [5.39 Interview Questions](#539-interview-questions)
- [5.40 Accelerator Command Cheat Sheet](#540-accelerator-command-cheat-sheet)
- [5.41 Chapter Summary](#541-chapter-summary)

---

# 5.26 Why Learn Other GPU Vendors?

NVIDIA is currently one of the most important GPU platforms in HPC and AI, but an HPC-AI Infrastructure Engineer should not design their knowledge around a single vendor.

Production environments may contain:

- NVIDIA GPUs
- AMD Instinct accelerators
- Intel Data Center GPUs
- Intel Gaudi AI accelerators
- Google TPUs
- AWS Trainium
- AWS Inferentia
- Other specialized accelerators

The infrastructure principles remain similar:

```text
Accelerator Hardware
        ↓
Kernel Driver
        ↓
Vendor Runtime / SDK
        ↓
Communication Libraries
        ↓
Framework
        ↓
Scheduler
        ↓
Application
```

The implementation changes by vendor.

---

# 5.27 AMD Instinct

AMD's primary data-center accelerator family for HPC and AI is **AMD Instinct**.

Important product families include:

- MI100
- MI200 Series
- MI250
- MI300 Series
- MI325X
- MI350 Series

AMD Instinct accelerators are used for:

- HPC
- AI Training
- AI Inference
- Scientific Computing

---

## Simplified Architecture

```text
              AMD GPU
                 │
          AMD Instinct
                 │
        ┌────────┴────────┐
        │                 │
     Compute            HBM
        │                 │
        └────────┬────────┘
                 │
              PCIe /
          High-speed Fabric
```

Modern AMD accelerators are designed for high-bandwidth memory and large-scale parallel workloads.

---

# 5.28 AMD ROCm

**ROCm** is AMD's open software platform for GPU computing.

It is the closest conceptual equivalent to NVIDIA's CUDA ecosystem.

```text
NVIDIA                         AMD

CUDA                           ROCm
 │                              │
CUDA Runtime                   HIP
 │                              │
NVIDIA Driver                  AMD Driver
 │                              │
GPU                            Instinct GPU
```

---

## Important ROCm Components

ROCm includes technologies such as:

- HIP
- ROCr
- RCCL
- rocBLAS
- MIOpen
- ROCm SMI / related management tooling

---

## HIP

**HIP (Heterogeneous-compute Interface for Portability)** provides a programming environment that can make GPU applications more portable across CUDA and AMD environments.

Conceptually:

```text
Application

      ↓

     HIP

   ┌──┴───┐
   ↓      ↓
CUDA    ROCm
```

This is useful when organizations want to reduce dependence on a single GPU ecosystem.

---

## AMD GPU Monitoring

Depending on the ROCm/software version, administrators may use tools such as:

```bash
rocm-smi
```

or newer ROCm management tooling.

Example:

```bash
rocm-smi
```

The exact commands available depend on the ROCm version and GPU generation.

---

# 5.29 Intel Data Center GPUs

Intel also produces data-center GPUs and accelerators for HPC and AI.

Important families include:

- Intel Data Center GPU Max Series
- Intel Data Center GPU Flex Series

Intel GPUs can be used for:

- HPC
- AI
- Visualization
- Data-center acceleration

---

## Intel GPU Software

Intel provides a software ecosystem based around:

- oneAPI
- SYCL
- Level Zero
- oneAPI libraries

---

## oneAPI

Intel oneAPI is designed to provide a unified programming model across different types of compute devices.

```text
              oneAPI
                │
       ┌────────┼────────┐
       ▼        ▼        ▼
      CPU      GPU     FPGA
```

---

## SYCL

SYCL is a C++-based programming model for heterogeneous computing.

It allows developers to write applications that can target different accelerator architectures.

---

## Intel GPU Monitoring

Depending on the hardware and installed software stack, tools may include:

```bash
xpu-smi
```

Example:

```bash
xpu-smi discovery
```

and:

```bash
xpu-smi stats
```

Exact functionality depends on the Intel GPU generation and installed toolkit.

---

# 5.30 Intel Gaudi AI Accelerators

Intel Gaudi is an important accelerator platform specifically focused on AI workloads.

Gaudi originated from **Habana Labs**, which Intel acquired.

Examples include:

- Gaudi
- Gaudi2
- Gaudi3

---

## Important Distinction

Gaudi is an **AI accelerator**, but it is not simply another traditional GPU product line.

It uses a different architecture and software stack.

```text
NVIDIA

GPU → CUDA

AMD

GPU → ROCm

Intel Gaudi

AI Accelerator → SynapseAI
```

---

# 5.31 Habana – Historical & Current Context

**Habana Labs** developed AI accelerators before becoming part of Intel.

Therefore, administrators may encounter both terms:

```text
Habana
   ↓
Intel acquisition
   ↓
Intel Gaudi
```

Older documentation may refer to:

- Habana Gaudi
- Habana Labs
- Habana drivers
- Habana SynapseAI

Current deployments may refer primarily to:

- Intel Gaudi
- Gaudi software stack
- Intel Gaudi accelerators

---

## Gaudi Software Stack

The software ecosystem includes:

- SynapseAI
- Habana drivers
- Framework integrations
- Communication libraries

The architecture can be viewed as:

```text
PyTorch / TensorFlow
        ↓
Gaudi Framework Integration
        ↓
SynapseAI
        ↓
Gaudi Driver
        ↓
Gaudi Hardware
```

---

# 5.32 Google TPU

**Tensor Processing Unit (TPU)** is Google's custom accelerator architecture designed primarily for machine learning workloads.

TPUs are particularly important for:

- Large-scale machine learning
- Tensor operations
- Deep learning training
- Google Cloud AI workloads

---

## TPU Architecture Concept

```text
AI Application

      ↓

ML Framework

      ↓

XLA

      ↓

TPU Runtime

      ↓

TPU
```

TPUs are fundamentally different from NVIDIA and AMD GPUs, but the infrastructure principles remain similar.

---

# 5.33 AWS AI Accelerators

AWS provides its own AI accelerator families.

Important examples:

### AWS Trainium

Designed primarily for:

- AI training
- Large-scale model training

### AWS Inferentia

Designed primarily for:

- AI inference
- Cost-efficient model serving

---

## Conceptual Architecture

```text
Application
     ↓
AWS ML Framework
     ↓
Neuron SDK
     ↓
Trainium / Inferentia
```

AWS Neuron provides the software ecosystem for these accelerators.

---

# 5.34 Other Important AI Accelerators

An HPC-AI Infrastructure Engineer may also encounter specialized accelerator technologies.

Examples include:

| Vendor / Platform | Accelerator |
|---|---|
| NVIDIA | GPU |
| AMD | Instinct GPU |
| Intel | Data Center GPU |
| Intel | Gaudi |
| Google | TPU |
| AWS | Trainium |
| AWS | Inferentia |
| Graphcore | IPU |
| Cerebras | Wafer-Scale Engine |
| Groq | LPU |
| Qualcomm | AI Accelerators |
| Huawei | Ascend |

These architectures differ significantly, but infrastructure management follows common principles.

---

# 5.35 NVIDIA vs AMD vs Intel vs TPU

| Feature | NVIDIA | AMD | Intel GPU | Intel Gaudi | Google TPU |
|---|---|---|---|---|---|
| Accelerator | GPU | GPU | GPU | AI Accelerator | TPU |
| Software | CUDA | ROCm | oneAPI | SynapseAI | XLA |
| Main API | CUDA | HIP | SYCL | Gaudi APIs | XLA |
| AI | Excellent | Strong | Strong | Strong | Strong |
| HPC | Excellent | Excellent | Strong | Primarily AI | Specialized |
| Scheduler | Slurm/K8s | Slurm/K8s | Slurm/K8s | Slurm/K8s | Cloud-oriented |
| HBM | Yes | Yes | Yes on relevant HPC GPUs | Yes | Yes |

The exact capabilities depend heavily on the specific accelerator generation.

---

# 5.36 Accelerator Software Stack

The most important concept is to understand the **layers**, not memorize every vendor command.

## NVIDIA

```text
Application
    ↓
PyTorch / TensorFlow
    ↓
CUDA / NCCL
    ↓
NVIDIA Driver
    ↓
GPU
```

---

## AMD

```text
Application
    ↓
PyTorch / TensorFlow
    ↓
ROCm / RCCL
    ↓
AMD Driver
    ↓
Instinct GPU
```

---

## Intel GPU

```text
Application
    ↓
Framework
    ↓
oneAPI / SYCL
    ↓
Level Zero / Driver
    ↓
Intel GPU
```

---

## Intel Gaudi

```text
Application
    ↓
PyTorch
    ↓
SynapseAI
    ↓
Gaudi Driver
    ↓
Gaudi
```

---

## Google TPU

```text
Application
    ↓
Framework
    ↓
XLA
    ↓
TPU Runtime
    ↓
TPU
```

---

# 5.37 Multi-Vendor HPC-AI Architecture

A production HPC-AI environment may contain multiple accelerator types.

```text
                    HPC-AI Cluster
                          │
             ┌────────────┼────────────┐
             │            │            │
             ▼            ▼            ▼
         NVIDIA        AMD Instinct   Intel
           GPU             GPU       Accelerator
             │            │            │
           CUDA          ROCm       oneAPI /
                                      Gaudi
             │            │            │
             └────────────┼────────────┘
                          ▼
                       Slurm
                          │
                          ▼
                    HPC Workloads
```

---

## Why Multi-Vendor?

Organizations may use multiple vendors because of:

- Availability
- Cost
- Procurement strategy
- Performance requirements
- Energy efficiency
- Software compatibility
- Vendor diversification

---

# 5.38 Multi-Vendor Troubleshooting

The troubleshooting methodology should remain consistent.

---

## Layer 1 – Hardware

Check:

```text
PCIe
Power
Firmware
Thermals
```

---

## Layer 2 – Driver

Verify:

```text
Vendor Driver
Kernel Module
Device Detection
```

---

## Layer 3 – Runtime

Verify:

```text
CUDA
ROCm
oneAPI
SynapseAI
TPU Runtime
```

---

## Layer 4 – Framework

Verify:

```text
PyTorch
TensorFlow
JAX
MPI
```

---

## Layer 5 – Scheduler

Verify:

```text
Slurm
Kubernetes
Device Plugin
Resource Allocation
```

---

## Layer 6 – Application

Verify:

```text
GPU Visibility
Memory
Communication
Performance
```

---

## Universal Troubleshooting Model

```text
             Application
                  │
                  ▼
              Framework
                  │
                  ▼
            Accelerator
              Runtime
                  │
                  ▼
               Driver
                  │
                  ▼
              Hardware
```

When troubleshooting, move **layer by layer** instead of immediately reinstalling drivers.

---

# 5.39 Interview Questions

## Basic

1. What GPU vendors are commonly used in HPC and AI?
2. What is AMD Instinct?
3. What is ROCm?
4. What is Intel oneAPI?
5. What is Intel Gaudi?
6. What is a TPU?

---

## Intermediate

1. Compare CUDA and ROCm.
2. What is HIP?
3. What is SYCL?
4. What is the difference between Intel GPU and Intel Gaudi?
5. What are AWS Trainium and Inferentia?
6. Why would an organization deploy multiple accelerator vendors?

---

## Advanced

1. How would you design Slurm for a heterogeneous GPU cluster?
2. How would you manage NVIDIA and AMD nodes using the same cluster?
3. How would you troubleshoot a ROCm GPU that is visible at PCIe level but unavailable to PyTorch?
4. How would you design monitoring for NVIDIA, AMD, and Intel accelerators?
5. What parts of an accelerator architecture are vendor-specific and what parts are common?
6. How would you design an HPC-AI platform that avoids hard dependency on a single accelerator vendor?

---

# 5.40 Accelerator Command Cheat Sheet

## NVIDIA

```bash
nvidia-smi
```

```bash
nvidia-smi -L
```

```bash
nvidia-smi topo -m
```

```bash
nvcc --version
```

---

## AMD

Depending on ROCm version:

```bash
rocm-smi
```

Useful ROCm tools can also include:

```bash
rocminfo
```

---

## Intel

Depending on the accelerator and installed software:

```bash
xpu-smi discovery
```

```bash
xpu-smi stats
```

---

## Intel Gaudi

Gaudi systems use Intel/Habana-specific administration and diagnostic tools. The exact commands depend on the Gaudi generation and SynapseAI release.

A common diagnostic tool is:

```bash
hl-smi
```

---

# 5.41 Chapter Summary

Modern HPC-AI infrastructure is no longer limited to a single accelerator architecture.

The major ecosystems include:

```text
NVIDIA
  └── CUDA

AMD
  └── ROCm

Intel GPU
  └── oneAPI / SYCL

Intel Gaudi
  └── SynapseAI

Google
  └── TPU / XLA

AWS
  └── Neuron / Trainium / Inferentia
```

For an HPC-AI Infrastructure Engineer, the most important skill is not memorizing every vendor command.

The important skill is understanding the common infrastructure model:

```text
Hardware
   ↓
Firmware
   ↓
Kernel Driver
   ↓
Vendor Runtime
   ↓
Communication Library
   ↓
Framework
   ↓
Scheduler
   ↓
Application
```

This allows you to transition between accelerator platforms without relearning infrastructure engineering from zero.

---

# Chapter 5 – Final Learning Checklist

You should now understand:

- NVIDIA GPU architecture
- NVIDIA drivers
- CUDA
- NVML
- `nvidia-smi`
- GPU monitoring
- MIG
- Slurm GPU scheduling
- GRES
- GPU containers
- GPUDirect RDMA relationship
- AMD Instinct
- ROCm
- HIP
- Intel Data Center GPUs
- oneAPI
- SYCL
- Intel Gaudi
- Habana Labs
- SynapseAI
- Google TPU
- AWS Trainium
- AWS Inferentia
- Multi-vendor accelerator architecture
- Multi-vendor troubleshooting

---

# HPC-AI Engineer Perspective

The goal is to move from:

```text
"I know NVIDIA commands."
```

to:

```text
"I understand accelerator infrastructure."
```

That distinction is important for an **HPC-AI Infrastructure Engineer**.

You should be able to walk into a heterogeneous cluster and answer:

```text
What accelerator is installed?
        ↓
Which driver manages it?
        ↓
Which runtime does it use?
        ↓
How does the framework access it?
        ↓
How does Slurm allocate it?
        ↓
How does it communicate with other accelerators?
        ↓
How is it monitored?
        ↓
How do I troubleshoot it?
```

That is the foundation of vendor-neutral HPC-AI infrastructure engineering.

---

# Next Chapter

**Chapter 6 – xCAT**

Topics:

- xCAT Architecture
- xCAT Database
- DHCP
- DNS
- Node Discovery
- PXE / Network Boot
- Stateless Provisioning
- Stateful Provisioning
- OS Images
- Compute Node Provisioning
- Post-Provisioning
- Production Troubleshooting
- xCAT Commands
- Interview Questions
