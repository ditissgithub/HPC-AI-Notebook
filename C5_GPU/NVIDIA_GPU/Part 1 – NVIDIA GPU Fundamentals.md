## Part 1 – NVIDIA GPU Fundamentals

- [5.1 Why GPUs are Important in HPC & AI](#51-why-gpus-are-important-in-hpc--ai)
- [5.2 What is a GPU?](#52-what-is-a-gpu)
- [5.3 GPU Architecture](#53-gpu-architecture)
- [5.4 CPU vs GPU](#54-cpu-vs-gpu)
- [5.5 CUDA Programming Model](#55-cuda-programming-model)
- [5.6 NVIDIA Software Stack](#56-nvidia-software-stack)
- [5.7 GPU Servers in HPC](#57-gpu-servers-in-hpc)
- [Key Takeaways](#key-takeaways)

---

# 5.1 Why GPUs are Important in HPC & AI

Modern High Performance Computing (HPC) and Artificial Intelligence (AI) workloads require enormous computational power.

Traditional CPUs are excellent for general-purpose computing, but workloads such as:

- Deep Learning
- Machine Learning
- Molecular Dynamics
- Weather Forecasting
- CFD Simulations
- Scientific Computing

can execute much faster on GPUs because thousands of operations can run simultaneously.

```
            CPU
             │
   Few Powerful Cores

----------------------------

            GPU
             │
 Thousands of Smaller Cores
```

Today, nearly every AI supercomputer uses NVIDIA GPUs as compute accelerators.

---

# 5.2 What is a GPU?

A **Graphics Processing Unit (GPU)** is a specialized processor designed for highly parallel computation.

Originally developed for graphics rendering, GPUs are now widely used for:

- AI Training
- AI Inference
- HPC Simulations
- Image Processing
- Video Processing
- Scientific Research

Unlike CPUs, GPUs execute the same operation across many data elements simultaneously.

---

## Popular NVIDIA Data Center GPUs

| GPU | Typical Use |
|------|-------------|
| Tesla P100 | HPC |
| V100 | AI & HPC |
| A100 | AI Training |
| H100 | Large AI Models |
| B200 (Blackwell) | Next-generation AI |

---

# 5.3 GPU Architecture

A simplified GPU consists of multiple Streaming Multiprocessors (SMs), each containing many CUDA Cores.

```
              GPU

      +-----------------+

      | Memory Controller|

      +-----------------+

      |      L2 Cache   |

      +-----------------+

      |  SM | SM | SM  |

      |  SM | SM | SM  |

      +-----------------+

         High Bandwidth
            Memory
```

---

## Major Components

### CUDA Cores

Perform arithmetic and logical operations.

---

### Streaming Multiprocessor (SM)

A group of CUDA cores responsible for executing parallel workloads.

---

### GPU Memory

Stores:

- Training Data
- Model Parameters
- Intermediate Results

Modern GPUs use High Bandwidth Memory (HBM) for faster data access.

---

# 5.4 CPU vs GPU

| Feature | CPU | GPU |
|----------|-----|-----|
| Cores | Few | Thousands |
| Purpose | General Computing | Parallel Computing |
| Latency | Low | Higher |
| Throughput | Moderate | Very High |
| HPC | Control Logic | Heavy Computation |
| AI | Scheduling | Model Training |

---

## HPC Perspective

```
CPU

↓

Operating System

↓

Slurm

↓

Application Launch

↓

GPU

↓

Heavy Computation
```

The CPU manages execution, while the GPU accelerates computationally intensive tasks.

---

# 5.5 CUDA Programming Model

**CUDA (Compute Unified Device Architecture)** is NVIDIA's parallel computing platform.

It enables developers to execute applications directly on NVIDIA GPUs.

Basic workflow:

```
Application

↓

CUDA Runtime

↓

CUDA Driver

↓

GPU

↓

Parallel Execution
```

---

## CUDA Components

- CUDA Toolkit
- CUDA Runtime
- CUDA Compiler (`nvcc`)
- CUDA Libraries
- NVIDIA Driver

---

## Verify CUDA Compiler

```bash
nvcc --version
```

Example Output

```
Cuda compilation tools,
release 12.x
```

---

# 5.6 NVIDIA Software Stack

The software stack required for GPU computing consists of several layers.

```
Application

↓

CUDA Library

↓

CUDA Runtime

↓

NVIDIA Driver

↓

GPU Hardware
```

---

## Major Components

| Component | Purpose |
|------------|----------|
| NVIDIA Driver | Controls GPU hardware |
| CUDA Toolkit | Development environment |
| CUDA Runtime | Executes GPU applications |
| cuBLAS | Linear Algebra |
| cuDNN | Deep Learning |
| NCCL | Multi-GPU Communication |

---

## HPC Note

The **NVIDIA Driver version** and **CUDA Toolkit version** must be compatible. An incompatibility may prevent GPU applications from running correctly.

---

# 5.7 GPU Servers in HPC

GPU servers combine multiple CPUs and GPUs to deliver accelerated computing.

Typical architecture:

```
            Login Node
                 │
            Slurm Scheduler
                 │
      -----------------------
      │                     │
      ▼                     ▼
 GPU Compute          GPU Compute
    Node                 Node
      │                     │
      └──────────┬──────────┘
                 ▼
            Shared Storage
```

Each compute node typically contains:

- Multi-core CPUs
- NVIDIA GPUs
- High-speed Memory
- InfiniBand HCA
- Local NVMe Storage

---

## Production Example

A research team submits a deep learning job.

Workflow:

```
User

↓

Slurm

↓

GPU Node Allocated

↓

CUDA Application Starts

↓

GPU Memory Allocated

↓

Training Begins

↓

Results Stored on Lustre
```

---

# Key Takeaways

- GPUs accelerate highly parallel workloads common in HPC and AI.
- NVIDIA GPUs dominate modern AI and supercomputing environments.
- CUDA provides the programming platform for GPU computing.
- The NVIDIA software stack includes drivers, CUDA, and optimized libraries.
- GPU servers combine CPUs, GPUs, high-speed networking, and storage to execute large-scale workloads.
- Understanding GPU architecture is the foundation for GPU administration and troubleshooting.

---

## Next Part

**Chapter 5 – Part 2**

Topics:

- NVIDIA Driver
- CUDA Installation
- `nvidia-smi`
- NVML
- GPU Monitoring
- GPU Utilization
- GPU Health Checks
- Essential GPU Commands
