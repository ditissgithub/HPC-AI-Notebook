# AI Infrastructure — Topic 1
## From a GPU to a Model Server

### 1. A Model Is a Set of Files

A trained AI model is represented by model files/checkpoints containing learned parameters (weights), along with configuration, tokenizer, and other metadata files.

> A model is not necessarily one large file; it can be a collection of files.

---

### 2. Why Not Run the Model on a Normal Computer?

The first constraint is **hardware**.

A CPU generally has fewer, more powerful general-purpose cores, while a GPU has many parallel compute units optimized for massively parallel workloads.

```text
CPU
├── Fewer powerful cores
└── General-purpose workloads

GPU
├── Many parallel compute units
└── Highly parallel workloads such as AI
```

> **Important:** Do not describe GPU cores simply as "small CPU cores." GPU execution units are architecturally different from CPU cores.

---

### 3. Feeding the GPU

GPU performance depends heavily on getting data to the compute units efficiently.

```text
CPU
 │
 └── System RAM (DDR)
          │
        PCIe
          │
          ▼
        GPU
          │
          └── VRAM / HBM
```

Modern AI GPUs such as NVIDIA H100/H200 use **HBM (High Bandwidth Memory)**.

HBM provides very high memory bandwidth, allowing the GPU compute hardware to access data much faster than typical system-memory paths.

#### Remember the difference

- **Memory capacity** → how much data can fit.
- **Memory bandwidth** → how quickly data can be transferred.

> Do not use a fixed statement such as "system RAM is 50× slower." The actual difference depends on the hardware, memory technology, and access path.

---

### 4. What Defines a GPU / GPU Card?

For AI infrastructure, important characteristics include:

```text
GPU
├── Compute capability
├── Memory capacity
├── Memory bandwidth
├── Interconnect
│   ├── PCIe
│   └── NVLink
└── Power / thermal characteristics
```

AI performance is not determined by compute capacity alone. A workload can be limited by:

- GPU compute
- GPU memory capacity
- GPU memory bandwidth
- GPU ↔ GPU communication
- CPU ↔ GPU communication
- Storage / data pipeline
- Software and configuration

---

### 5. The GPU Needs Software

GPU hardware cannot run a PyTorch model by itself.

A simplified NVIDIA software stack is:

```text
AI Model
   ↓
PyTorch / TensorFlow / JAX / Runtime
   ↓
CUDA Libraries
   ↓
NVIDIA Driver
   ↓
GPU Hardware
```

For distributed AI workloads:

```text
PyTorch
   ↓
NCCL
   ↓
NVLink / NVSwitch / InfiniBand
   ↓
Other GPUs
```

The important idea is:

> **AI infrastructure = hardware + drivers + libraries + runtime + orchestration + application + operations.**

---

### 6. GPU Compute: SMs and Tensor Cores

A GPU contains many **Streaming Multiprocessors (SMs)**.

Conceptually:

```text
GPU
├── SM 0
├── SM 1
├── SM 2
├── ...
└── Many SMs
```

SMs contain execution resources such as CUDA Cores and Tensor Cores.

#### CUDA Cores

General GPU compute resources used for a wide range of operations.

#### Tensor Cores

Specialized hardware designed to accelerate matrix/tensor operations common in AI workloads.

```text
AI workload
    ↓
Matrix operations
    ↓
Tensor Cores
    ↓
High AI compute throughput
```

---

### 7. HBM: Capacity vs Bandwidth

HBM is the GPU's high-bandwidth memory used to store data such as:

- Model weights
- Activations
- Gradients
- Intermediate tensors
- KV cache

Two concepts must be separated:

```text
HBM Capacity
= How much data can fit

HBM Bandwidth
= How quickly data can move between
  GPU compute and memory
```

A GPU can have enough memory capacity but still become **memory-bandwidth bound**.

---

### 8. Compute-Bound vs Memory-Bound

#### Compute-bound

The GPU's compute resources are the limiting factor.

```text
GPU Compute
    │
    ▼
CUDA / Tensor Cores
    │
    ▼
High utilization
```

#### Memory-bound

The GPU spends significant time waiting for data movement.

```text
GPU Compute
    ↑
    │ waiting
    │
   HBM
```

Therefore:

> **High GPU utilization does not automatically mean the workload is performing well.**

A workload can show high utilization while being limited by memory bandwidth, communication, inefficient kernels, synchronization, or other factors.

---

### 9. GPU-to-GPU Communication

Multi-GPU AI workloads frequently require GPUs to exchange data.

Within a server:

```text
GPU 0 ── NVLink / NVSwitch ── GPU 1
 │                              │
 └────────── GPU fabric ────────┘
```

Across servers:

```text
GPU
 │
 ▼
NIC
 │
 ▼
InfiniBand / Ethernet
 │
 ▼
NIC
 │
 ▼
GPU
```

#### NVLink

High-speed GPU-to-GPU interconnect used for communication within supported GPU systems.

#### NVSwitch

A high-bandwidth switching fabric used to connect multiple GPUs in supported NVIDIA systems.

#### InfiniBand

Commonly used for high-performance communication between GPU servers.

---

### 10. Where NCCL Fits

NCCL (NVIDIA Collective Communications Library) provides optimized collective communication for multi-GPU and multi-node workloads.

Simplified:

```text
PyTorch
   ↓
NCCL
   ├── NVLink / NVSwitch
   └── InfiniBand / RDMA
```

NCCL is important for operations such as:

- All-reduce
- All-gather
- Reduce-scatter
- Broadcast

This is why GPU topology and the network fabric can directly affect distributed-training performance.

---

### 11. CPU ↔ GPU ↔ NUMA

GPU performance can also depend on CPU and NUMA placement.

Example:

```text
NUMA Node 0                 NUMA Node 1
     │                           │
    CPU0                        CPU1
     │                           │
    PCIe                        PCIe
     │                           │
   GPU0                        GPU1
```

If a process is poorly aligned with the GPU's CPU/NUMA locality, additional latency and lower performance can occur.

Useful commands:

```bash
lscpu
numactl -H
nvidia-smi topo -m
```

This is especially important for HPC and distributed AI workloads.

---

### 12. A Script Is Not Automatically a Service

A simple inference script might be:

```bash
python inference.py
```

This is an application/process, not necessarily a production service.

A production model-serving architecture looks more like:

```text
Model
  ↓
Inference Runtime
  ↓
Model Server
  ↓
API Endpoint
  ↓
Users / Applications
```

A production model server may additionally provide:

- Request handling
- Batching
- Concurrency management
- GPU utilization
- Health checks
- Metrics
- Logging
- Scaling
- High availability
- Load balancing

### Key distinction

```text
Script
= Runs a task

Service
= Continuously provides a capability
  to clients with operational controls
```

Examples of model-serving technologies include NVIDIA Triton Inference Server, vLLM, and other inference runtimes/servers.

---

## 13. From Model File to Production Model Server

The complete progression is:

```text
Model Files
    ↓
GPU Infrastructure
    ↓
CUDA + Driver + Libraries
    ↓
Inference Runtime
    ↓
Model Server
    ↓
API Endpoint
    ↓
Users / Applications
```

Production infrastructure adds:

```text
Load Balancing
Monitoring & Metrics
Logging
Health Checks
Auto Scaling
High Availability
Security
```

---

## 14. Production AI Infrastructure Mental Model

```text
                 AI MODEL
              weights + config
                     │
                     ▼
          AI Framework / Runtime
          PyTorch / vLLM / etc.
                     │
                     ▼
                CUDA + Driver
                     │
                     ▼
               GPU Hardware
          ┌──────────┼──────────┐
          │          │          │
       Compute      HBM     Interconnect
          │          │          │
          └──────────┼──────────┘
                     ▼
                Model Server
                     │
                     ▼
                  API
                     │
                     ▼
              Users / Clients
```

---

## 15. Tier-2 Troubleshooting Perspective

When an AI workload is slow or failing, do not immediately assume the GPU is faulty.

Think through the layers:

```text
Application
    ↓
Framework
    ↓
CUDA / Libraries
    ↓
NCCL / Communication
    ↓
GPU
    ↓
HBM
    ↓
NVLink / NVSwitch
    ↓
CPU / NUMA / PCIe
    ↓
InfiniBand / Network
    ↓
Storage / Data Pipeline
```

Example:

> **GPU utilization = 95% does not prove that the entire AI system is healthy.**

Possible bottlenecks can still exist in:

- HBM bandwidth
- GPU-to-GPU communication
- CPU/data-loader pipeline
- storage
- network
- kernel efficiency
- synchronization

---

## 16. Key Takeaways

1. A trained model is usually a **set of model files and metadata**, not necessarily one file.
2. GPUs are designed for **massively parallel workloads**.
3. GPU execution units are architecturally different from CPU cores.
4. **HBM capacity** and **HBM bandwidth** are different concepts.
5. GPU performance depends on more than TFLOPS.
6. **SMs** are fundamental GPU compute units.
7. **Tensor Cores** accelerate AI-oriented matrix operations.
8. **NVLink/NVSwitch** enable high-speed GPU communication within supported systems.
9. **InfiniBand/RDMA** is commonly used for communication between GPU servers.
10. **NCCL** provides optimized collective communication for distributed GPU workloads.
11. CPU/NUMA affinity can affect GPU performance.
12. A Python inference script is not automatically a production model service.
13. A production model server adds APIs, request handling, monitoring, health checks, scaling, and reliability.
14. AI infrastructure combines **compute + memory + networking + software + orchestration + operations**.

---

## 17. One-Line Summary

> **AI infrastructure transforms trained model files into a reliable, scalable service by combining GPU compute, high-bandwidth memory, high-speed interconnects, software runtimes, orchestration, and production operations.**
