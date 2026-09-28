# Phase 1: Bridge the Critical Gaps
## AI Infrastructure Engineer — Practical Notebook
**Author:** Soni Gupta | HPC-AI Infrastructure Engineer  
**Base:** 4.5+ years HPC, 20 PetaFLOPS clusters, OpenCHAI, xCAT/SLURM, InfiniBand/RDMA  
**Goal:** Extend proven HPC infrastructure expertise into modern AI Infrastructure engineering  
**Timeframe:** Months 1–3

---

## Table of Contents

1. [Conceptual Bridge: HPC → AI Infrastructure](#1-conceptual-bridge)
2. [Module 1: Kubeflow on GPU Clusters](#2-module-1-kubeflow-on-gpu-clusters)
3. [Module 2: vLLM — LLM Inference Serving](#3-module-2-vllm-llm-inference-serving)
4. [Module 3: Triton Inference Server](#4-module-3-triton-inference-server)
5. [Module 4: Cloud Infrastructure Hands-On (AWS/GCP)](#5-module-4-cloud-infrastructure)
6. [Module 5: MinIO Object Storage for AI Workloads](#6-module-5-minio-object-storage)
7. [Integration: Full AI Infrastructure Stack](#7-integration-full-stack)
8. [Validation Checklist](#8-validation-checklist)

---

## 1. Conceptual Bridge

> Before writing a single `kubectl` command, understand how your existing skills map to the AI infrastructure world. Nothing here is foreign — it's a paradigm shift, not a technology replacement.

### HPC → AI Infrastructure Terminology Map

| HPC Concept (You Know) | AI Infrastructure Equivalent | Key Difference |
|------------------------|-------------------------------|----------------|
| SLURM job scheduler | Kubernetes + Kubeflow Pipelines | K8s is always-on; SLURM is batch-first |
| xCAT node provisioning | Kubernetes Node Pools / cluster-autoscaler | Dynamic scaling vs static provisioning |
| MPI job (`srun --ntasks`) | PyTorch DDP / FSDP / DeepSpeed | Framework-managed collective comms |
| `module load cuda/12.2` | Container image with CUDA base (`nvcr.io/nvidia/cuda:12.2`) | Immutable image vs runtime modules |
| Lustre parallel filesystem | Object store (S3/MinIO) + shared PVC | Pull-on-demand vs POSIX mount |
| InfiniBand (IB) fabric | InfiniBand or RoCEv2 for NCCL all-reduce | Same hardware, different upper-layer |
| `sacct` / `squeue` monitoring | Prometheus + Grafana + DCGM Exporter | Pull-based metrics vs push logs |
| SLURM QoS policies | Kubernetes ResourceQuota + LimitRange | Namespace-scoped vs partition-scoped |
| HPL benchmark | MLPerf Training / NCCL tests | FP64 peak vs AI collective throughput |
| User home dirs (NFS/LDAP) | PersistentVolumeClaims + RBAC | Declarative vs imperative access |
| Multi-rack IB topology | K8s topology-aware scheduling | `topologySpreadConstraints` |
| Bare-metal PXE boot | CoreOS/RHCOS node image + Ignition | Immutable OS vs traditional Linux |

### Why Kubernetes Won in AI Infrastructure

SLURM was designed for batch HPC. Kubernetes was designed for persistent services. Modern AI workloads are a hybrid: training jobs (batch) + inference servers (persistent) + data pipelines (streaming). Kubernetes unified these under one control plane.

Your advantage: you already understand low-level node management, network fabric, and GPU topology — the hard parts that most Kubernetes engineers don't know. Kubernetes is just the orchestration layer on top of infrastructure you already master.

---

## 2. Module 1: Kubeflow on GPU Clusters

### 2.1 Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    Kubeflow Platform                         │
│                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐  │
│  │  Kubeflow    │  │   Kubeflow   │  │     Katib         │  │
│  │  Notebooks   │  │  Pipelines   │  │  (HPO / AutoML)  │  │
│  │  (JupyterHub)│  │  (Argo WF)   │  │                  │  │
│  └──────────────┘  └──────────────┘  └──────────────────┘  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐  │
│  │  Training    │  │    KServe    │  │   Central        │  │
│  │  Operators   │  │  (Inference) │  │   Dashboard      │  │
│  │  (PyTorch,TF)│  │              │  │                  │  │
│  └──────────────┘  └──────────────┘  └──────────────────┘  │
│                                                             │
│              Kubernetes Control Plane                       │
│  ┌───────────────────────────────────────────────────────┐  │
│  │  API Server │ Scheduler │ etcd │ Controller Manager   │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                             │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │ GPU Node │  │ GPU Node │  │ GPU Node │  │ CPU Node │   │
│  │ 8x A100  │  │ 8x A100  │  │ 8x H100  │  │ (system) │   │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘   │
└─────────────────────────────────────────────────────────────┘
```

### 2.2 Prerequisites

Your existing cluster should have:
- Kubernetes ≥ 1.27 (via kubeadm — you already do this)
- NVIDIA GPU Operator installed
- kubectl configured
- At least one GPU node with CUDA 12.x

Verify your GPU nodes are visible to Kubernetes:

```bash
# Check GPU nodes are labeled and allocatable
kubectl get nodes -o custom-columns=\
NAME:.metadata.name,\
GPU:.status.allocatable.'nvidia\.com/gpu',\
STATUS:.status.conditions[-1].type

# Expected output:
# NAME           GPU   STATUS
# gpu-node-01    8     Ready
# gpu-node-02    8     Ready

# Check NVIDIA device plugin is running
kubectl get pods -n kube-system | grep nvidia-device-plugin

# Verify GPU is schedulable
kubectl describe node gpu-node-01 | grep -A5 "Allocatable"
```

### 2.3 Install Kubeflow (Manifests Method)

Kubeflow provides a kustomize-based manifest deployment. This gives you full control — no opaque Helm charts.

```bash
# Install kustomize (if not already installed)
curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh" | bash
sudo mv kustomize /usr/local/bin/

# Clone manifests (pin to a stable release)
export KUBEFLOW_VERSION=v1.8.0
git clone --branch ${KUBEFLOW_VERSION} \
  https://github.com/kubeflow/manifests.git
cd manifests

# Deploy the full Kubeflow stack
# This deploys ~60 components — takes 5–10 minutes
while ! kustomize build example | kubectl apply -f -; do
  echo "Retrying in 10s (CRDs may not be ready yet)..."
  sleep 10
done

# Watch rollout — all pods must reach Running
kubectl get pods -n kubeflow --watch
```

**HPC Parallel:** Installing Kubeflow is like deploying a management middleware stack on your xCAT head node. The `kustomize build | apply` loop is needed because Kubernetes CRDs (Custom Resource Definitions) must be installed before the controllers that use them — similar to how xCAT plugins require the base database schema before you can add node definitions.

### 2.4 Expose the Dashboard

```bash
# Port-forward to access the Kubeflow dashboard locally
kubectl port-forward svc/istio-ingressgateway \
  -n istio-system 8080:80 &

# OR — for cluster-wide access, patch to NodePort
kubectl patch svc istio-ingressgateway \
  -n istio-system \
  -p '{"spec": {"type": "NodePort"}}'

# Get the assigned NodePort
kubectl get svc istio-ingressgateway -n istio-system \
  -o jsonpath='{.spec.ports[?(@.name=="http2")].nodePort}'
```

Access: `http://<node-ip>:<nodeport>` — default credentials: `user@example.com` / `12341234`

### 2.5 Configure GPU Resource Quotas per Namespace

In SLURM you use QoS and partition limits. In Kubernetes you use `ResourceQuota` and `LimitRange` per namespace. Create a profile for AI training teams:

```yaml
# ai-team-namespace.yaml
---
apiVersion: v1
kind: Namespace
metadata:
  name: ai-team-alpha
  labels:
    app.kubernetes.io/part-of: kubeflow-profile
---
apiVersion: v1
kind: ResourceQuota
metadata:
  name: gpu-quota
  namespace: ai-team-alpha
spec:
  hard:
    requests.cpu: "128"
    requests.memory: 512Gi
    requests.nvidia.com/gpu: "16"    # Max GPUs this team can request
    limits.nvidia.com/gpu: "16"
    persistentvolumeclaims: "20"
    requests.storage: "10Ti"
---
apiVersion: v1
kind: LimitRange
metadata:
  name: gpu-limit-range
  namespace: ai-team-alpha
spec:
  limits:
  - type: Container
    default:
      nvidia.com/gpu: "1"            # Default GPU per container if not specified
    defaultRequest:
      nvidia.com/gpu: "1"
    max:
      nvidia.com/gpu: "8"            # Max GPUs per single container
```

```bash
kubectl apply -f ai-team-namespace.yaml

# Verify quotas
kubectl describe resourcequota gpu-quota -n ai-team-alpha
```

### 2.6 Run a Distributed PyTorch Training Job with PyTorchJob

This is where your SLURM `srun` knowledge directly translates. A `PyTorchJob` is the Kubernetes equivalent of an MPI/SLURM job — it launches a master + N worker pods with coordinated environment variables for `RANK`, `WORLD_SIZE`, `MASTER_ADDR`.

```yaml
# pytorch-distributed-training.yaml
apiVersion: kubeflow.org/v1
kind: PyTorchJob
metadata:
  name: resnet-distributed-train
  namespace: ai-team-alpha
spec:
  pytorchReplicaSpecs:
    Master:
      replicas: 1
      restartPolicy: OnFailure
      template:
        spec:
          nodeSelector:
            accelerator: nvidia-a100        # Target your GPU nodes
          containers:
          - name: pytorch
            image: nvcr.io/nvidia/pytorch:24.01-py3
            command:
            - python
            - -m
            - torch.distributed.run
            - --nproc_per_node=8            # 8 GPUs per node
            - --nnodes=$(WORLD_SIZE)
            - --node_rank=$(RANK)
            - --master_addr=$(MASTER_ADDR)
            - --master_port=23456
            - /workspace/train.py
            - --epochs=10
            - --batch-size=256
            resources:
              limits:
                nvidia.com/gpu: "8"
                memory: "480Gi"
              requests:
                nvidia.com/gpu: "8"
                memory: "480Gi"
            env:
            - name: NCCL_DEBUG
              value: "INFO"
            - name: NCCL_IB_DISABLE
              value: "0"                    # Enable InfiniBand (you have this!)
            - name: NCCL_IB_HCA
              value: "mlx5_0,mlx5_1"       # Your IB HCA ports — adjust per node
            - name: NCCL_SOCKET_IFNAME
              value: "eth0"
            - name: CUDA_VISIBLE_DEVICES
              value: "0,1,2,3,4,5,6,7"
    Worker:
      replicas: 3                           # 3 additional workers = 4 nodes total
      restartPolicy: OnFailure
      template:
        spec:
          nodeSelector:
            accelerator: nvidia-a100
          containers:
          - name: pytorch
            image: nvcr.io/nvidia/pytorch:24.01-py3
            command:
            - python
            - -m
            - torch.distributed.run
            - --nproc_per_node=8
            - --nnodes=$(WORLD_SIZE)
            - --node_rank=$(RANK)
            - --master_addr=$(MASTER_ADDR)
            - --master_port=23456
            - /workspace/train.py
            - --epochs=10
            - --batch-size=256
            resources:
              limits:
                nvidia.com/gpu: "8"
                memory: "480Gi"
              requests:
                nvidia.com/gpu: "8"
                memory: "480Gi"
            env:
            - name: NCCL_DEBUG
              value: "INFO"
            - name: NCCL_IB_DISABLE
              value: "0"
            - name: NCCL_IB_HCA
              value: "mlx5_0,mlx5_1"
```

```bash
kubectl apply -f pytorch-distributed-training.yaml

# Monitor — equivalent to `squeue` in SLURM
kubectl get pytorchjob -n ai-team-alpha
kubectl get pods -n ai-team-alpha -l training.kubeflow.org/job-name=resnet-distributed-train

# Watch NCCL initialization logs
kubectl logs -n ai-team-alpha \
  resnet-distributed-train-master-0 -f | grep -E "NCCL|rank|allreduce"
```

**Key insight:** The environment variables `RANK`, `WORLD_SIZE`, and `MASTER_ADDR` are injected automatically by the PyTorch operator — exactly like OpenMPI sets `OMPI_COMM_WORLD_RANK`. Your NCCL tuning experience transfers directly here.

### 2.7 NCCL Tuning for InfiniBand (Leverage Your IB Expertise)

This is where your InfiniBand knowledge gives you an immediate edge over typical Kubernetes engineers:

```bash
# Run NCCL all-reduce bandwidth test as a K8s job
cat <<EOF | kubectl apply -f -
apiVersion: batch/v1
kind: Job
metadata:
  name: nccl-allreduce-test
  namespace: ai-team-alpha
spec:
  template:
    spec:
      restartPolicy: Never
      nodeSelector:
        accelerator: nvidia-a100
      containers:
      - name: nccl-test
        image: nvcr.io/nvidia/cuda:12.2.0-devel-ubuntu22.04
        command: ["/bin/bash", "-c"]
        args:
        - |
          # Install NCCL tests
          git clone https://github.com/NVIDIA/nccl-tests.git && cd nccl-tests
          make MPI=0 CUDA_HOME=/usr/local/cuda NCCL_HOME=/usr
          
          # Run all-reduce over all 8 GPUs
          ./build/all_reduce_perf -b 512M -e 8G -f 2 -g 8 \
            -z 0 --op sum
        resources:
          limits:
            nvidia.com/gpu: "8"
        env:
        - name: NCCL_IB_DISABLE
          value: "0"
        - name: NCCL_IB_GID_INDEX
          value: "3"        # RoCEv2 GID index — match your IB config
        - name: NCCL_NET_GDR_LEVEL
          value: "5"        # GPUDirect RDMA — critical for A100 NVLink+IB
        - name: NCCL_ALGO
          value: "Ring"     # Ring vs Tree — test both
EOF

kubectl logs -n ai-team-alpha job/nccl-allreduce-test -f
```

Expected output (8x A100 SXM5 over IB HDR):
```
#                                                              out-of-place                       in-place
#       size         count      type   redop    root     time   algbw   busbw #wrong     time   algbw   busbw #wrong
     536870912     134217728     float     sum      -1    4.97  107.99  202.48      0     4.95  108.42  203.29      0
    1073741824     268435456     float     sum      -1    9.88  108.68  203.77      0     9.85  109.01  204.39      0
```

Target: **busbw > 180 GB/s** per all-reduce on IB HDR (200Gbps). If you see < 100 GB/s, check `NCCL_IB_GID_INDEX` (must match your RDMA GID configuration from `ibv_devinfo`).

### 2.8 Kubeflow Pipelines: Your First ML Pipeline

A Kubeflow Pipeline is a DAG of containerized steps — think of it as a scriptable, versioned replacement for multi-step shell scripts that span multiple nodes.

```python
# pipeline_definition.py
# Install: pip install kfp==2.7.0

import kfp
from kfp import dsl
from kfp.dsl import Dataset, Input, Model, Output

# --- Step 1: Data preprocessing component ---
@dsl.component(
    base_image="nvcr.io/nvidia/pytorch:24.01-py3",
    packages_to_install=["datasets", "transformers"]
)
def preprocess_data(
    dataset_name: str,
    output_dataset: Output[Dataset]
):
    """Download and tokenize dataset. Analogous to a pre-processing
    SLURM step that stages data to Lustre before the MPI job."""
    from datasets import load_dataset
    from transformers import AutoTokenizer
    import json, os

    tokenizer = AutoTokenizer.from_pretrained("bert-base-uncased")
    dataset = load_dataset(dataset_name, split="train[:1%]")
    
    tokenized = dataset.map(
        lambda x: tokenizer(x["text"], truncation=True, max_length=512),
        batched=True
    )
    
    os.makedirs(output_dataset.path, exist_ok=True)
    tokenized.save_to_disk(output_dataset.path)
    print(f"Preprocessed {len(tokenized)} samples to {output_dataset.path}")


# --- Step 2: Training component ---
@dsl.component(
    base_image="nvcr.io/nvidia/pytorch:24.01-py3",
    packages_to_install=["transformers", "accelerate"]
)
def train_model(
    input_dataset: Input[Dataset],
    num_epochs: int,
    output_model: Output[Model]
):
    """Fine-tune a BERT model. This component runs on GPU nodes."""
    import torch
    from transformers import AutoModelForSequenceClassification, TrainingArguments, Trainer
    from datasets import load_from_disk
    import os

    print(f"CUDA available: {torch.cuda.is_available()}")
    print(f"GPU count: {torch.cuda.device_count()}")

    dataset = load_from_disk(input_dataset.path)
    model = AutoModelForSequenceClassification.from_pretrained(
        "bert-base-uncased", num_labels=2
    )

    training_args = TrainingArguments(
        output_dir=output_model.path,
        num_train_epochs=num_epochs,
        per_device_train_batch_size=32,
        fp16=True,                        # Mixed precision — like HPL's optimized BLAS
        dataloader_num_workers=4,
        logging_steps=50,
        save_strategy="epoch"
    )

    trainer = Trainer(
        model=model,
        args=training_args,
        train_dataset=dataset
    )
    trainer.train()
    trainer.save_model(output_model.path)
    print(f"Model saved to {output_model.path}")


# --- Step 3: Evaluation component ---
@dsl.component(
    base_image="nvcr.io/nvidia/pytorch:24.01-py3",
    packages_to_install=["transformers", "scikit-learn"]
)
def evaluate_model(
    input_model: Input[Model],
    input_dataset: Input[Dataset],
    accuracy_threshold: float
) -> float:
    from transformers import pipeline
    from datasets import load_from_disk
    from sklearn.metrics import accuracy_score
    import json

    dataset = load_from_disk(input_dataset.path)
    classifier = pipeline("text-classification", model=input_model.path, device=0)
    
    predictions = classifier(dataset["text"][:100], truncation=True)
    predicted_labels = [1 if p["label"] == "POSITIVE" else 0 for p in predictions]
    
    # Dummy labels for example
    true_labels = [1] * 50 + [0] * 50
    acc = accuracy_score(true_labels, predicted_labels)
    
    print(f"Accuracy: {acc:.4f}")
    assert acc >= accuracy_threshold, f"Model below threshold: {acc} < {accuracy_threshold}"
    return acc


# --- Pipeline definition ---
@dsl.pipeline(
    name="bert-fine-tuning-pipeline",
    description="End-to-end BERT fine-tuning on GPU nodes"
)
def bert_pipeline(
    dataset_name: str = "ag_news",
    num_epochs: int = 3,
    accuracy_threshold: float = 0.80
):
    preprocess_task = preprocess_data(dataset_name=dataset_name)
    
    train_task = train_model(
        input_dataset=preprocess_task.outputs["output_dataset"],
        num_epochs=num_epochs
    )
    # Request GPU for training step
    train_task.set_accelerator_type("nvidia.com/gpu").set_accelerator_limit(1)
    train_task.set_memory_limit("32G")
    
    evaluate_model(
        input_model=train_task.outputs["output_model"],
        input_dataset=preprocess_task.outputs["output_dataset"],
        accuracy_threshold=accuracy_threshold
    )


# --- Compile and submit ---
if __name__ == "__main__":
    from kfp import compiler
    
    # Compile to YAML IR
    compiler.Compiler().compile(bert_pipeline, "bert_pipeline.yaml")
    print("Pipeline compiled to bert_pipeline.yaml")
    
    # Submit to Kubeflow
    client = kfp.Client(host="http://localhost:8080")
    run = client.create_run_from_pipeline_func(
        bert_pipeline,
        arguments={
            "dataset_name": "ag_news",
            "num_epochs": 3,
            "accuracy_threshold": 0.80
        },
        run_name="bert-training-run-v1"
    )
    print(f"Run submitted: {run.run_id}")
```

---

## 3. Module 2: vLLM — LLM Inference Serving

### 3.1 What vLLM Solves

Traditional inference (loading a model, running `model.generate()`) is GPU-inefficient: the KV-cache grows unboundedly, batching is naive, and GPU utilization during decoding drops to 30–40%. vLLM solves this with **PagedAttention** — a memory management algorithm that treats the KV-cache like an OS virtual memory system with pages and page tables, enabling:

- **Continuous batching:** New requests join in-flight batches, not at batch boundaries
- **Near-100% GPU memory utilization:** No fragmentation, no padding waste
- **2–24× higher throughput** vs naive HuggingFace inference

Your HPC parallel: PagedAttention is to GPU memory what SLURM's `--mem-per-cpu` + cgroups is to RAM — fine-grained, non-wasteful allocation. The difference is it operates within a single GPU job.

### 3.2 Architecture

```
        Client Requests (HTTP/gRPC)
               │
               ▼
    ┌─────────────────────┐
    │   vLLM API Server   │  ← OpenAI-compatible REST API
    │  (AsyncIO engine)   │
    └──────────┬──────────┘
               │ Scheduler
               ▼
    ┌─────────────────────┐
    │  Continuous Batcher │  ← Merges waiting + running requests
    │  (request queues)   │
    └──────────┬──────────┘
               │
               ▼
    ┌─────────────────────┐
    │  PagedAttention     │  ← KV-cache pages (like OS pages in DRAM)
    │  KV Cache Manager   │    Block table per sequence
    └──────────┬──────────┘
               │
               ▼
    ┌─────────────────────┐
    │  LLM Model Engine   │  ← Tensor parallel across N GPUs
    │  (GPU Workers)      │    NCCL all-reduce between ranks
    └─────────────────────┘
```

### 3.3 Installation

```bash
# On a GPU node with CUDA 12.1+
pip install vllm  # Pulls in torch, transformers, ray

# Verify GPU access
python -c "import vllm; print(vllm.__version__)"
python -c "import torch; print(torch.cuda.device_count())"
```

### 3.4 Single-GPU Inference Server

Start with a smaller model to validate your setup before scaling:

```bash
# Serve Llama-3.1-8B (requires HuggingFace token for gated models)
export HF_TOKEN=hf_your_token_here

python -m vllm.entrypoints.openai.api_server \
  --model meta-llama/Meta-Llama-3.1-8B-Instruct \
  --dtype bfloat16 \               # bf16 is preferred on A100/H100
  --max-model-len 8192 \
  --gpu-memory-utilization 0.90 \  # Leave 10% headroom
  --tensor-parallel-size 1 \       # Single GPU
  --port 8000 \
  --host 0.0.0.0

# Test with curl — OpenAI-compatible endpoint
curl http://localhost:8000/v1/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "meta-llama/Meta-Llama-3.1-8B-Instruct",
    "prompt": "Explain GPU memory hierarchy in HPC systems:",
    "max_tokens": 256,
    "temperature": 0.7
  }'
```

### 3.5 Multi-GPU Tensor Parallelism (Your InfiniBand Pays Off)

For 70B+ parameter models, you need tensor parallelism (TP) across multiple GPUs. vLLM uses NCCL for inter-GPU communication — your InfiniBand fabric reduces all-reduce latency dramatically here.

```bash
# Serve Llama-3.1-70B across 8 GPUs (TP=8)
# With IB: all-reduce latency ~2–5μs vs ~50μs over PCIe
NCCL_IB_DISABLE=0 \
NCCL_IB_HCA=mlx5_0,mlx5_1 \
NCCL_NET_GDR_LEVEL=5 \
python -m vllm.entrypoints.openai.api_server \
  --model meta-llama/Meta-Llama-3.1-70B-Instruct \
  --dtype bfloat16 \
  --tensor-parallel-size 8 \       # Split across 8 GPUs
  --max-model-len 16384 \
  --gpu-memory-utilization 0.92 \
  --port 8000 \
  --host 0.0.0.0

# For 2-node serving (pipeline parallel + tensor parallel)
# TP=8 PP=2 = 16 GPUs total
python -m vllm.entrypoints.openai.api_server \
  --model meta-llama/Meta-Llama-3.1-70B-Instruct \
  --tensor-parallel-size 8 \
  --pipeline-parallel-size 2 \
  --dtype bfloat16 \
  --port 8000
```

### 3.6 Benchmark vLLM Throughput

This is your **HPL equivalent for LLM inference** — run this and put the numbers in your portfolio:

```bash
# Clone vLLM for benchmark scripts
git clone https://github.com/vllm-project/vllm.git
cd vllm/benchmarks

# Throughput benchmark — measures tokens/second
python benchmark_throughput.py \
  --model meta-llama/Meta-Llama-3.1-8B-Instruct \
  --dtype bfloat16 \
  --input-len 512 \
  --output-len 256 \
  --num-prompts 1000 \
  --gpu-memory-utilization 0.90

# Latency benchmark — measures Time-to-First-Token (TTFT) and Time-per-Output-Token (TPOT)
python benchmark_latency.py \
  --model meta-llama/Meta-Llama-3.1-8B-Instruct \
  --dtype bfloat16 \
  --input-len 512 \
  --output-len 128 \
  --num-iters 100 \
  --batch-size 1

# Online serving benchmark — simulates concurrent users with Poisson arrival
python benchmark_serving.py \
  --backend vllm \
  --model meta-llama/Meta-Llama-3.1-8B-Instruct \
  --dataset-name random \
  --num-prompts 500 \
  --request-rate 10 \             # 10 requests/second
  --port 8000
```

**Key metrics to record for your portfolio:**

| Metric | Description | Target (A100 80GB, 8B model) |
|--------|-------------|-------------------------------|
| Throughput | Output tokens/sec (total) | > 2,000 tok/s |
| TTFT | Time to first token (ms) | < 100 ms at p50 |
| TPOT | Time per output token (ms) | < 20 ms at p50 |
| GPU Util | GPU SM utilization % | > 75% |
| Memory util | VRAM used / total | ~90% (target) |

### 3.7 Deploy vLLM as a Kubernetes Service

```yaml
# vllm-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: vllm-llama3-8b
  namespace: ai-inference
spec:
  replicas: 1
  selector:
    matchLabels:
      app: vllm-llama3-8b
  template:
    metadata:
      labels:
        app: vllm-llama3-8b
    spec:
      nodeSelector:
        accelerator: nvidia-a100
      containers:
      - name: vllm-server
        image: vllm/vllm-openai:latest
        command: ["python", "-m", "vllm.entrypoints.openai.api_server"]
        args:
        - "--model=meta-llama/Meta-Llama-3.1-8B-Instruct"
        - "--dtype=bfloat16"
        - "--tensor-parallel-size=1"
        - "--gpu-memory-utilization=0.90"
        - "--max-model-len=8192"
        - "--port=8000"
        - "--host=0.0.0.0"
        ports:
        - containerPort: 8000
          name: http
        env:
        - name: HF_TOKEN
          valueFrom:
            secretKeyRef:
              name: hf-credentials
              key: token
        - name: NCCL_IB_DISABLE
          value: "0"
        - name: NCCL_IB_HCA
          value: "mlx5_0,mlx5_1"
        resources:
          limits:
            nvidia.com/gpu: "1"
            memory: "80Gi"
          requests:
            nvidia.com/gpu: "1"
            memory: "80Gi"
        volumeMounts:
        - name: model-cache
          mountPath: /root/.cache/huggingface
        livenessProbe:
          httpGet:
            path: /health
            port: 8000
          initialDelaySeconds: 120
          periodSeconds: 30
        readinessProbe:
          httpGet:
            path: /health
            port: 8000
          initialDelaySeconds: 90
          periodSeconds: 10
      volumes:
      - name: model-cache
        persistentVolumeClaim:
          claimName: model-cache-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: vllm-llama3-8b-svc
  namespace: ai-inference
spec:
  selector:
    app: vllm-llama3-8b
  ports:
  - port: 80
    targetPort: 8000
    name: http
  type: ClusterIP
```

```bash
# Create HuggingFace token secret
kubectl create secret generic hf-credentials \
  --from-literal=token=$HF_TOKEN \
  -n ai-inference

kubectl apply -f vllm-deployment.yaml

# Check startup (model loading takes 2–5 min)
kubectl logs -n ai-inference deploy/vllm-llama3-8b -f | \
  grep -E "Warming up|engine initialized|GPU blocks"
```

---

## 4. Module 3: Triton Inference Server

### 4.1 Triton vs vLLM — When to Use Each

| Criteria | vLLM | Triton Inference Server |
|----------|------|-------------------------|
| Primary use | LLM serving | Multi-framework model serving |
| Model types | LLMs (transformer decoder) | Any ML model (ONNX, TensorRT, PyTorch, TF) |
| Batching | Dynamic continuous batching | Static / sequence batching |
| Customization | Python backend extensions | C++ backends, custom ops |
| LLM throughput | Best-in-class | Requires TRT-LLM backend |
| GPU ensemble | Built-in tensor parallel | Ensemble scheduling |
| Use when | Serving LLMs in production | Serving vision models, ONNX pipelines, multi-model |

In an HPC-AI stack, you'll likely use **both**: vLLM for language models, Triton for everything else (image classifiers, custom CUDA kernels, ONNX pipelines).

### 4.2 Triton Model Repository Structure

Triton uses a filesystem-based model registry — conceptually similar to how you structure software packages in xCAT's osimages, but for ML models:

```
model_repository/
├── resnet50_onnx/
│   ├── config.pbtxt          ← Model configuration
│   └── 1/                    ← Version 1
│       └── model.onnx
├── bert_tensorrt/
│   ├── config.pbtxt
│   └── 1/
│       └── model.plan        ← TensorRT serialized engine
├── llama3_vllm/
│   ├── config.pbtxt
│   └── 1/
│       └── model.py          ← Python backend
└── preprocessing_pipeline/
    ├── config.pbtxt           ← Ensemble config
    └── (no model file — ensemble)
```

### 4.3 Deploy a ResNet-50 ONNX Model

```bash
# Pull a pretrained ResNet-50 and export to ONNX
pip install torch torchvision onnx onnxruntime

python3 << 'EOF'
import torch
import torchvision.models as models
import os

# Load pretrained ResNet-50
model = models.resnet50(pretrained=True)
model.eval()

# Dummy input: batch=1, RGB image 224x224
dummy_input = torch.randn(1, 3, 224, 224)

# Export to ONNX
os.makedirs("model_repository/resnet50_onnx/1", exist_ok=True)
torch.onnx.export(
    model, dummy_input,
    "model_repository/resnet50_onnx/1/model.onnx",
    input_names=["input"],
    output_names=["output"],
    dynamic_axes={"input": {0: "batch_size"}, "output": {0: "batch_size"}},
    opset_version=17
)
print("ONNX model exported successfully")
EOF
```

Write the Triton configuration:

```protobuf
# model_repository/resnet50_onnx/config.pbtxt
name: "resnet50_onnx"
backend: "onnxruntime"
max_batch_size: 32

input [
  {
    name: "input"
    data_type: TYPE_FP32
    dims: [ 3, 224, 224 ]   # CHW format (without batch dimension)
  }
]

output [
  {
    name: "output"
    data_type: TYPE_FP32
    dims: [ 1000 ]           # ImageNet classes
  }
]

dynamic_batching {
  preferred_batch_size: [ 4, 8, 16, 32 ]
  max_queue_delay_microseconds: 100    # Wait up to 100μs to form a batch
}

instance_group [
  {
    count: 1
    kind: KIND_GPU
    gpus: [ 0 ]
  }
]

optimization {
  execution_accelerators {
    gpu_execution_accelerator: [
      {
        name: "tensorrt"       # Auto-convert ONNX to TRT at load time
        parameters {
          key: "precision_mode"
          value: "FP16"
        }
      }
    ]
  }
}
```

### 4.4 Launch Triton Server

```bash
# Pull NVIDIA Triton container
docker pull nvcr.io/nvidia/tritonserver:24.01-py3

# Launch with GPU
docker run --gpus all \
  -p 8000:8000 \   # HTTP
  -p 8001:8001 \   # gRPC
  -p 8002:8002 \   # Metrics (Prometheus format!)
  -v $(pwd)/model_repository:/models \
  nvcr.io/nvidia/tritonserver:24.01-py3 \
  tritonserver \
    --model-repository=/models \
    --log-verbose=1 \
    --metrics-port=8002

# OR as a Kubernetes deployment:
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: triton-server
  namespace: ai-inference
spec:
  replicas: 1
  selector:
    matchLabels:
      app: triton-server
  template:
    metadata:
      labels:
        app: triton-server
      annotations:
        prometheus.io/scrape: "true"     # Auto-scrape by Prometheus
        prometheus.io/port: "8002"
        prometheus.io/path: "/metrics"
    spec:
      containers:
      - name: triton
        image: nvcr.io/nvidia/tritonserver:24.01-py3
        args:
        - tritonserver
        - --model-repository=/models
        - --log-verbose=1
        ports:
        - containerPort: 8000
        - containerPort: 8001
        - containerPort: 8002
        resources:
          limits:
            nvidia.com/gpu: "1"
        volumeMounts:
        - name: model-repo
          mountPath: /models
      volumes:
      - name: model-repo
        nfs:                             # Mount your NFS/Lustre model store
          server: nfs-server.cluster.local
          path: /models
EOF
```

### 4.5 Send Inference Requests

```python
# triton_client.py
import tritonclient.http as httpclient
import numpy as np
from PIL import Image

# Connect
client = httpclient.InferenceServerClient(url="localhost:8000")

# Check server health
assert client.is_server_live()
assert client.is_model_ready("resnet50_onnx")

# Prepare input (ResNet-50 expects normalized ImageNet input)
def preprocess_image(image_path):
    img = Image.open(image_path).resize((224, 224))
    img_array = np.array(img).astype(np.float32)
    img_array = img_array.transpose(2, 0, 1)  # HWC → CHW
    # ImageNet normalization
    mean = np.array([0.485, 0.456, 0.406]).reshape(3, 1, 1)
    std  = np.array([0.229, 0.224, 0.225]).reshape(3, 1, 1)
    img_array = (img_array / 255.0 - mean) / std
    return img_array[np.newaxis, :]  # Add batch dim

# Batch inference (simulating production load)
batch_size = 8
inputs = np.vstack([preprocess_image("test.jpg")] * batch_size)

infer_input = httpclient.InferInput("input", inputs.shape, "FP32")
infer_input.set_data_from_numpy(inputs)

result = client.infer(
    model_name="resnet50_onnx",
    inputs=[infer_input],
    outputs=[httpclient.InferRequestedOutput("output")]
)

output = result.as_numpy("output")  # Shape: (8, 1000)
top5 = np.argsort(output[0])[-5:][::-1]
print(f"Top-5 class IDs: {top5}")
print(f"Throughput: {batch_size} images per request")
```

### 4.6 Triton Prometheus Metrics — Integrate with Your Grafana Stack

Since you already run Prometheus + Grafana for SLURM/HPC monitoring, Triton's Prometheus-compatible `/metrics` endpoint plugs directly in:

```yaml
# Add to your Prometheus scrape config
# /etc/prometheus/prometheus.yml
scrape_configs:
  - job_name: 'triton'
    static_configs:
      - targets: ['triton-server-svc.ai-inference.svc:8002']
    metrics_path: '/metrics'

# Key Triton metrics to dashboard:
# nv_inference_request_success         — total successful requests
# nv_inference_request_duration_us     — request latency (microseconds)
# nv_inference_queue_duration_us       — time in queue
# nv_gpu_utilization                   — GPU SM utilization
# nv_gpu_memory_used_bytes             — VRAM used
# nv_inference_count                   — inference count per model
```

---

## 5. Module 4: Cloud Infrastructure

> **Goal:** Understand how bare-metal HPC maps to cloud primitives. You don't need to abandon on-prem — you need to fluently speak both languages.

### 5.1 HPC Concepts → AWS Primitives

| HPC Component | AWS Equivalent | Notes |
|---------------|----------------|-------|
| Head node | EC2 + Elastic IP | Always-on, small instance type |
| GPU compute nodes | EC2 p3/p4/p5 instances | p4d.24xlarge = 8x A100 |
| InfiniBand fabric | Elastic Fabric Adapter (EFA) | AWS's RDMA-like network, 400Gbps |
| SLURM scheduler | AWS ParallelCluster or EKS | ParallelCluster = SLURM on AWS |
| Lustre filesystem | Amazon FSx for Lustre | Managed Lustre, S3-integrated |
| NFS home dirs | Amazon EFS | Elastic NFS, multi-AZ |
| xCAT provisioning | EC2 AMI + Launch Templates | Immutable image-based provisioning |
| IB Placement groups | EC2 Placement Groups (cluster) | Same rack for low-latency |
| Job accounting | AWS Cost Explorer + tags | Tag EC2 by project, user |

### 5.2 AWS CLI Setup

```bash
# Install AWS CLI v2
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip && sudo ./aws/install

# Configure credentials (use IAM role in production, not root keys)
aws configure
# AWS Access Key ID: <your-key>
# AWS Secret Access Key: <your-secret>
# Default region: us-east-1     ← p4d instances available here
# Default output format: json

# Verify
aws sts get-caller-identity
```

### 5.3 Launch a GPU Instance

```bash
# Find the latest Deep Learning AMI (DLAMI) — pre-installed CUDA, PyTorch, etc.
aws ec2 describe-images \
  --owners amazon \
  --filters \
    "Name=name,Values=Deep Learning OSS Nvidia Driver AMI GPU PyTorch*" \
    "Name=state,Values=available" \
  --query "sort_by(Images, &CreationDate)[-1].[ImageId,Name]" \
  --output table

# Create a key pair
aws ec2 create-key-pair \
  --key-name ai-infra-key \
  --query 'KeyMaterial' \
  --output text > ai-infra-key.pem
chmod 400 ai-infra-key.pem

# Launch a p3.2xlarge (1x V100, cost-effective for learning)
INSTANCE_ID=$(aws ec2 run-instances \
  --image-id ami-XXXXXXXXX \         # Replace with DLAMI ID from above
  --instance-type p3.2xlarge \
  --key-name ai-infra-key \
  --security-group-ids sg-XXXXXXXXX \
  --subnet-id subnet-XXXXXXXXX \
  --block-device-mappings '[{"DeviceName":"/dev/sda1","Ebs":{"VolumeSize":200,"VolumeType":"gp3","Iops":3000}}]' \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=ai-infra-test},{Key=Project,Value=openchai}]' \
  --query 'Instances[0].InstanceId' \
  --output text)

echo "Launched: $INSTANCE_ID"

# Wait for running state
aws ec2 wait instance-running --instance-ids $INSTANCE_ID

# Get public IP
PUBLIC_IP=$(aws ec2 describe-instances \
  --instance-ids $INSTANCE_ID \
  --query 'Reservations[0].Instances[0].PublicIpAddress' \
  --output text)

# SSH in
ssh -i ai-infra-key.pem ubuntu@$PUBLIC_IP

# ALWAYS stop/terminate when done to avoid charges
aws ec2 stop-instances --instance-ids $INSTANCE_ID
# aws ec2 terminate-instances --instance-ids $INSTANCE_ID  # When fully done
```

### 5.4 S3 as AI Dataset Store

S3 is the standard object store for AI training datasets. Think of it as an infinitely scalable, HTTP-accessible replacement for your Lustre staging area.

```bash
# Create a bucket for AI datasets
aws s3 mb s3://openchai-ai-datasets-$(date +%Y%m) \
  --region us-east-1

# Upload a dataset (streaming, no need to download first)
aws s3 cp \
  ./my_training_data/ \
  s3://openchai-ai-datasets-202501/imagenet-1k/ \
  --recursive \
  --storage-class INTELLIGENT_TIERING \  # Auto-tier between S3 classes
  --sse AES256 \                          # Server-side encryption
  --no-progress

# Sync (like rsync — only sends changed files)
aws s3 sync \
  ./checkpoints/ \
  s3://openchai-ai-datasets-202501/checkpoints/ \
  --delete

# Stream directly into PyTorch without downloading
python3 << 'EOF'
import boto3
import io
import torch
import numpy as np

s3 = boto3.client("s3")

# Stream a checkpoint file from S3 without full download
obj = s3.get_object(Bucket="openchai-ai-datasets-202501", Key="checkpoints/model_epoch_10.pt")
buffer = io.BytesIO(obj["Body"].read())
checkpoint = torch.load(buffer, map_location="cpu")
print(f"Loaded checkpoint: epoch={checkpoint.get('epoch')}")
EOF
```

### 5.5 AWS ParallelCluster — Run SLURM on AWS

Since you're a SLURM expert, ParallelCluster lets you deploy a managed SLURM cluster on AWS in minutes:

```yaml
# parallelcluster-config.yaml
Region: us-east-1
Image:
  Os: alinux2

HeadNode:
  InstanceType: c5.2xlarge
  Networking:
    SubnetId: subnet-XXXXXXXXX
  Ssh:
    KeyName: ai-infra-key

Scheduling:
  Scheduler: slurm
  SlurmQueues:
  - Name: gpu-queue
    ComputeResources:
    - Name: gpu-nodes
      InstanceType: p3.8xlarge    # 4x V100 per node
      MinCount: 0                  # Scale to zero when idle
      MaxCount: 10
      Efa:
        Enabled: false             # Enable for p4d instances with EFA
    Networking:
      SubnetIds:
        - subnet-XXXXXXXXX
      PlacementGroup:
        Enabled: true              # Same placement group = low latency

SharedStorage:
  - MountDir: /lustre
    Name: lustre-storage
    StorageType: FsxLustre
    FsxLustreSettings:
      StorageCapacity: 1200        # GB, must be multiple of 1200
      ImportPath: s3://openchai-ai-datasets-202501
      DeploymentType: SCRATCH_2
```

```bash
# Install ParallelCluster
pip install aws-parallelcluster

# Create cluster
pcluster create-cluster \
  --cluster-name openchai-aws \
  --cluster-configuration parallelcluster-config.yaml

# SSH to head node
pcluster ssh --cluster-name openchai-aws -i ai-infra-key.pem

# On head node — submit a GPU job (same SLURM you know)
sbatch << 'EOF'
#!/bin/bash
#SBATCH --job-name=gpu-test
#SBATCH --partition=gpu-queue
#SBATCH --nodes=2
#SBATCH --ntasks-per-node=4
#SBATCH --gres=gpu:4
#SBATCH --time=00:30:00

srun python3 -c "import torch; print(f'GPU: {torch.cuda.get_device_name(0)}')"
EOF

# DELETE when done (you pay for head node even when idle)
pcluster delete-cluster --cluster-name openchai-aws
```

---

## 6. Module 5: MinIO Object Storage for AI Workloads

### 6.1 Why MinIO for OpenCHAI

You already have Lustre for parallel I/O in HPC jobs. Adding MinIO gives you:

- **S3-compatible API:** Every AI framework (PyTorch, HuggingFace, Spark) has S3 connectors
- **Model registry:** Store and version checkpoints like Docker stores image layers
- **Dataset staging:** Pre-stage training data closer to compute (like Lustre staging from tape)
- **Multi-tenant:** Each AI team gets a bucket with access policies — like LDAP groups for Lustre

### 6.2 Architecture with OpenCHAI

```
┌─────────────────────────────────────────────────────────┐
│                    OpenCHAI Cluster                      │
│                                                          │
│  ┌──────────────────────────────────────────────────┐   │
│  │              MinIO Distributed Cluster            │   │
│  │   (4 nodes × 2 drives = 8 drives, erasure coded) │   │
│  │   Accessible via S3 API at minio.cluster.local   │   │
│  └──────────────────┬───────────────────────────────┘   │
│                     │ S3 API (port 9000)                 │
│          ┌──────────┴──────────┐                        │
│          │                     │                        │
│  ┌───────▼──────┐    ┌────────▼──────┐                  │
│  │  Training    │    │  Inference    │                   │
│  │  Jobs        │    │  Servers      │                   │
│  │  (PyTorch    │    │  (vLLM /      │                   │
│  │   SLURM/K8s) │    │   Triton)     │                   │
│  └──────────────┘    └───────────────┘                   │
│                                                          │
│  ┌──────────────────────────────────────────────────┐   │
│  │       Existing Lustre Filesystem                  │   │
│  │       (High-bandwidth parallel I/O for HPC)       │   │
│  └──────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

### 6.3 Deploy MinIO on Kubernetes (Distributed Mode)

```bash
# Install MinIO Operator
kubectl apply -k github.com/minio/operator

# Create a MinIO Tenant (distributed cluster)
cat <<EOF | kubectl apply -f -
apiVersion: minio.min.io/v2
kind: Tenant
metadata:
  name: openchai-minio
  namespace: minio-tenant
spec:
  image: minio/minio:RELEASE.2024-01-16T16-07-38Z
  
  pools:
  - name: pool-0
    servers: 4                          # 4 MinIO server pods
    volumesPerServer: 2                 # 2 PVCs per pod
    volumeClaimTemplate:
      spec:
        storageClassName: local-storage # Use your local SSDs
        resources:
          requests:
            storage: 10Ti              # 10 TiB per PVC
  
  mountPath: /data
  subPath: /data
  
  requestAutoCert: true                # TLS certificate from MinIO operator
  
  env:
  - name: MINIO_BROWSER
    value: "on"
  - name: MINIO_STORAGE_CLASS_STANDARD
    value: "EC:2"                      # Erasure code — 2 parity drives (like RAID-6)
  
  users:
  - name: openchai-minio-admin
  
  features:
    bucketDNS: false
    domains: {}

---
apiVersion: v1
kind: Secret
metadata:
  name: openchai-minio-admin
  namespace: minio-tenant
data:
  accesskey: b3BlbmNoYWktYWRtaW4=     # "openchai-admin" base64
  secretkey: c2VjcmV0a2V5MTIzNDU2Nzg= # "secretkey12345678" base64
EOF

kubectl get tenant -n minio-tenant
kubectl get pods -n minio-tenant
```

### 6.4 Configure MinIO Buckets for AI Workloads

```bash
# Install MinIO client
wget https://dl.min.io/client/mc/release/linux-amd64/mc
chmod +x mc && sudo mv mc /usr/local/bin/

# Configure alias
mc alias set openchai \
  http://minio.minio-tenant.svc.cluster.local:80 \
  openchai-admin \
  secretkey12345678

# Create bucket structure for AI workloads
mc mb openchai/ai-datasets          # Raw training datasets
mc mb openchai/ai-checkpoints       # Model checkpoints during training
mc mb openchai/ai-models            # Final production models
mc mb openchai/ai-logs              # Training metrics and logs
mc mb openchai/openchai-cache       # Provisioning artifacts, ISOs

# Set versioning on checkpoint bucket (like Git for model weights)
mc version enable openchai/ai-checkpoints

# Set lifecycle policy — auto-delete old checkpoints after 30 days
mc ilm rule add \
  --expire-days 30 \
  openchai/ai-checkpoints

# Create per-team access policy
cat <<POLICY > ai-team-alpha-policy.json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:PutObject", "s3:DeleteObject", "s3:ListBucket"],
      "Resource": [
        "arn:aws:s3:::ai-datasets/*",
        "arn:aws:s3:::ai-checkpoints/team-alpha/*",
        "arn:aws:s3:::ai-models/team-alpha/*"
      ]
    }
  ]
}
POLICY

mc admin policy create openchai ai-team-alpha ai-team-alpha-policy.json
mc admin user add openchai team-alpha-user team-alpha-secret-key
mc admin policy attach openchai ai-team-alpha --user team-alpha-user
```

### 6.5 PyTorch Training with MinIO S3 Storage

This snippet shows how a training job in your OpenCHAI cluster reads data from MinIO and writes checkpoints back — replacing manual file staging to Lustre:

```python
# train_with_minio.py
import torch
import torch.nn as nn
import boto3
from torch.utils.data import DataLoader, Dataset
import io, os, json
from datetime import datetime

# ─── MinIO / S3 Configuration ──────────────────────────────────────────────
MINIO_ENDPOINT   = os.environ.get("MINIO_ENDPOINT", "http://minio.minio-tenant.svc.cluster.local:80")
MINIO_ACCESS_KEY = os.environ.get("MINIO_ACCESS_KEY", "openchai-admin")
MINIO_SECRET_KEY = os.environ.get("MINIO_SECRET_KEY", "secretkey12345678")

s3_client = boto3.client(
    "s3",
    endpoint_url=MINIO_ENDPOINT,
    aws_access_key_id=MINIO_ACCESS_KEY,
    aws_secret_access_key=MINIO_SECRET_KEY,
    region_name="us-east-1"  # MinIO ignores this but boto3 requires it
)

# ─── Dataset that streams from MinIO ───────────────────────────────────────
class MinIODataset(Dataset):
    """Streams dataset shards from MinIO on demand.
    Compare to: opening files from Lustre in your SLURM job."""
    
    def __init__(self, bucket: str, prefix: str, transform=None):
        self.bucket = bucket
        self.prefix = prefix
        self.transform = transform
        
        # List all objects in prefix (like `ls` on Lustre path)
        response = s3_client.list_objects_v2(Bucket=bucket, Prefix=prefix)
        self.keys = [obj["Key"] for obj in response.get("Contents", [])]
        print(f"Found {len(self.keys)} objects in s3://{bucket}/{prefix}")
    
    def __len__(self):
        return len(self.keys)
    
    def __getitem__(self, idx):
        # Stream individual sample from MinIO
        obj = s3_client.get_object(Bucket=self.bucket, Key=self.keys[idx])
        data = torch.load(io.BytesIO(obj["Body"].read()))
        
        if self.transform:
            data["input"] = self.transform(data["input"])
        
        return data["input"], data["label"]

# ─── Checkpoint save/load to MinIO ─────────────────────────────────────────
def save_checkpoint_to_minio(model, optimizer, epoch, loss, bucket="ai-checkpoints", prefix="team-alpha"):
    """Save checkpoint to MinIO — replaces writing to Lustre scratch."""
    checkpoint = {
        "epoch": epoch,
        "model_state_dict": model.state_dict(),
        "optimizer_state_dict": optimizer.state_dict(),
        "loss": loss,
        "timestamp": datetime.utcnow().isoformat()
    }
    
    buffer = io.BytesIO()
    torch.save(checkpoint, buffer)
    buffer.seek(0)
    
    key = f"{prefix}/checkpoint_epoch_{epoch:04d}.pt"
    s3_client.put_object(
        Bucket=bucket,
        Key=key,
        Body=buffer.getvalue(),
        ContentType="application/octet-stream"
    )
    print(f"Checkpoint saved to s3://{bucket}/{key}")

def load_latest_checkpoint_from_minio(model, optimizer, bucket="ai-checkpoints", prefix="team-alpha"):
    """Resume training from latest MinIO checkpoint."""
    response = s3_client.list_objects_v2(Bucket=bucket, Prefix=prefix)
    if "Contents" not in response:
        print("No checkpoint found, starting from scratch.")
        return 0, float("inf")
    
    # Get latest checkpoint by key name (epoch number in name)
    checkpoints = sorted([o["Key"] for o in response["Contents"]])
    latest_key = checkpoints[-1]
    
    obj = s3_client.get_object(Bucket=bucket, Key=latest_key)
    checkpoint = torch.load(io.BytesIO(obj["Body"].read()), map_location="cpu")
    
    model.load_state_dict(checkpoint["model_state_dict"])
    optimizer.load_state_dict(checkpoint["optimizer_state_dict"])
    
    print(f"Resumed from s3://{bucket}/{latest_key} (epoch {checkpoint['epoch']})")
    return checkpoint["epoch"], checkpoint["loss"]

# ─── Training Loop ──────────────────────────────────────────────────────────
def train():
    device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
    print(f"Training on: {device}")

    model = nn.Sequential(
        nn.Linear(784, 512), nn.ReLU(),
        nn.Linear(512, 256), nn.ReLU(),
        nn.Linear(256, 10)
    ).to(device)

    optimizer = torch.optim.Adam(model.parameters(), lr=1e-3)
    criterion = nn.CrossEntropyLoss()

    # Resume from MinIO checkpoint if exists
    start_epoch, best_loss = load_latest_checkpoint_from_minio(model, optimizer)

    dataset = MinIODataset(bucket="ai-datasets", prefix="mnist-shards/train/")
    loader = DataLoader(dataset, batch_size=256, shuffle=True, num_workers=4)

    for epoch in range(start_epoch, 20):
        model.train()
        total_loss = 0
        for batch_idx, (data, target) in enumerate(loader):
            data, target = data.to(device), target.to(device)
            optimizer.zero_grad()
            output = model(data)
            loss = criterion(output, target)
            loss.backward()
            optimizer.step()
            total_loss += loss.item()

        avg_loss = total_loss / len(loader)
        print(f"Epoch {epoch}: loss={avg_loss:.4f}")

        # Save checkpoint every 5 epochs
        if epoch % 5 == 0:
            save_checkpoint_to_minio(model, optimizer, epoch, avg_loss)

if __name__ == "__main__":
    train()
```

### 6.6 MinIO as OpenCHAI Extension

Add a MinIO provisioning module to OpenCHAI so that every new cluster deployment automatically gets an S3 endpoint:

```bash
# openchai/modules/storage/minio_deploy.sh
#!/bin/bash
# Part of OpenCHAI — deploys MinIO on a freshly provisioned cluster
# Usage: ./minio_deploy.sh <cluster_name> <storage_nodes> <drives_per_node>

CLUSTER_NAME=${1:-"openchai-cluster"}
STORAGE_NODES=${2:-4}
DRIVES_PER_NODE=${3:-2}

echo "[OpenCHAI] Deploying MinIO for cluster: ${CLUSTER_NAME}"
echo "[OpenCHAI] Config: ${STORAGE_NODES} nodes × ${DRIVES_PER_NODE} drives"

# Label storage nodes
for i in $(seq 1 $STORAGE_NODES); do
    kubectl label node storage-node-0${i} \
      openchai/role=object-storage \
      openchai/cluster=${CLUSTER_NAME}
done

# Generate tenant manifest from template
cat > /tmp/minio-tenant-${CLUSTER_NAME}.yaml << YAML
apiVersion: minio.min.io/v2
kind: Tenant
metadata:
  name: ${CLUSTER_NAME}-minio
  namespace: minio-tenant
  labels:
    openchai.io/cluster: ${CLUSTER_NAME}
spec:
  pools:
  - name: pool-0
    servers: ${STORAGE_NODES}
    volumesPerServer: ${DRIVES_PER_NODE}
    nodeSelector:
      openchai/cluster: ${CLUSTER_NAME}
    volumeClaimTemplate:
      spec:
        storageClassName: local-storage
        resources:
          requests:
            storage: 10Ti
YAML

kubectl apply -f /tmp/minio-tenant-${CLUSTER_NAME}.yaml

# Create standard AI bucket structure
echo "[OpenCHAI] Waiting for MinIO to be ready..."
kubectl wait --for=condition=ready pod \
  -l app=minio -n minio-tenant \
  --timeout=300s

MINIO_IP=$(kubectl get svc -n minio-tenant \
  -l v1.min.io/tenant=${CLUSTER_NAME}-minio \
  -o jsonpath='{.items[0].spec.clusterIP}')

mc alias set ${CLUSTER_NAME} http://${MINIO_IP}:80 openchai-admin secretkey
mc mb ${CLUSTER_NAME}/ai-datasets
mc mb ${CLUSTER_NAME}/ai-checkpoints
mc mb ${CLUSTER_NAME}/ai-models

echo "[OpenCHAI] MinIO deployed at http://${MINIO_IP}:80"
echo "[OpenCHAI] Bucket structure created for AI workloads"
```

---

## 7. Integration: Full AI Infrastructure Stack

After completing all four modules, your cluster looks like this:

```
┌─────────────────────────────────────────────────────────────────────┐
│                   OpenCHAI AI Infrastructure Stack                   │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                     Kubeflow Platform                        │    │
│  │  [Notebooks] [Pipelines] [Training Operators] [KServe]      │    │
│  └──────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  ┌──────────────────────┐  ┌──────────────────────────────────┐     │
│  │   vLLM Inference     │  │   Triton Inference Server        │     │
│  │   (LLMs: Llama, etc) │  │   (ONNX, TRT, Vision models)    │     │
│  │   TP=8, IB-connected │  │   Prometheus metrics → Grafana  │     │
│  └──────────────────────┘  └──────────────────────────────────┘     │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │              MinIO Object Store (S3-compatible)              │    │
│  │   [ai-datasets] [ai-checkpoints] [ai-models] [ai-logs]      │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │            Kubernetes + GPU Operator                         │    │
│  │   DCGM Exporter → Prometheus → Grafana (you already have)   │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  ┌───────────────┐  ┌────────────────┐  ┌───────────────────────┐  │
│  │ InfiniBand /  │  │ Lustre FS      │  │ AWS (overflow /       │  │
│  │ RoCEv2 Fabric │  │ (HPC workloads)│  │  burst / cold data)   │  │
│  └───────────────┘  └────────────────┘  └───────────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
```

### End-to-End Test: Train → Save → Serve

```bash
# 1. Start a training job via Kubeflow
kubectl apply -f pytorch-distributed-training.yaml -n ai-team-alpha

# 2. Monitor training — check NCCL bandwidth
kubectl logs -n ai-team-alpha resnet-distributed-train-master-0 | \
  grep -E "loss|epoch|allreduce"

# 3. After training, model is saved to MinIO
mc ls openchai/ai-models/team-alpha/

# 4. Deploy the trained model with vLLM (if LLM) or Triton (if classification)
kubectl apply -f triton-deployment.yaml -n ai-inference

# 5. Verify inference endpoint
curl http://triton-server-svc.ai-inference.svc/v2/models/resnet50_onnx/ready
# Expected: {"name":"resnet50_onnx","ready":true}

# 6. Check GPU utilization across the full stack
kubectl exec -n ai-inference deploy/triton-server -- nvidia-smi dmon -s u
```

---

## 8. Validation Checklist

Use this checklist to confirm Phase 1 is complete before moving to Phase 2.

### Module 1 — Kubeflow
- [ ] Kubeflow fully deployed, all pods Running in `kubeflow` namespace
- [ ] Dashboard accessible, user profile created
- [ ] `PyTorchJob` ran successfully with ≥2 workers
- [ ] NCCL all-reduce test shows > 150 GB/s busbw on IB fabric
- [ ] First Kubeflow Pipeline (bert_pipeline) compiled and run successfully
- [ ] ResourceQuota applied to a namespace — GPU limit enforced

### Module 2 — vLLM
- [ ] vLLM serves Llama-3.1-8B on single GPU
- [ ] Benchmark run: throughput > 1,500 tok/s (A100), latency TTFT < 200ms
- [ ] Multi-GPU TP=4 or TP=8 serving confirmed with NCCL IB enabled
- [ ] vLLM deployed as Kubernetes Deployment with health checks passing
- [ ] Benchmark results documented with GPU model, batch size, concurrency

### Module 3 — Triton
- [ ] ResNet-50 ONNX model loaded and serving on GPU
- [ ] TensorRT optimization confirmed (check Triton logs for "TensorRT execution")
- [ ] Prometheus metrics endpoint `/metrics` returning `nv_inference_*` metrics
- [ ] Grafana dashboard updated with Triton metrics alongside existing HPC metrics
- [ ] Python client sending batched requests and receiving correct predictions

### Module 4 — Cloud (AWS/GCP)
- [ ] AWS CLI configured and `get-caller-identity` returns your account
- [ ] GPU instance (p3.2xlarge) launched, SSH'd in, `nvidia-smi` confirmed
- [ ] S3 bucket created, files uploaded and downloaded successfully
- [ ] ParallelCluster deployed with SLURM, GPU job submitted and completed
- [ ] All cloud resources terminated (no ongoing charges)

### Module 5 — MinIO
- [ ] MinIO Operator and Tenant deployed in Kubernetes
- [ ] 4 buckets created: datasets, checkpoints, models, logs
- [ ] PyTorch training job reads from MinIO and saves checkpoint to MinIO
- [ ] Checkpoint resume confirmed (delete local checkpoint, re-run — resumes from MinIO)
- [ ] MinIO integrated into OpenCHAI as a provisioning module

### Portfolio Evidence
- [ ] GitHub: Push all YAML manifests, scripts, and configs to `ditissgithub`
- [ ] OpenCHAI: Open PR adding MinIO deployment module
- [ ] Benchmarks: Document vLLM and NCCL test results in your repo's `benchmarks/` directory
- [ ] Resume: Add "vLLM, Triton Inference Server, Kubeflow, MinIO/S3" to Technical Skills section

---

*Notebook by Soni Gupta · HPC-AI Infrastructure Engineer · sonig@nvidia.com*  
*Phase 1 of AI Infrastructure Engineer Transition Roadmap*
