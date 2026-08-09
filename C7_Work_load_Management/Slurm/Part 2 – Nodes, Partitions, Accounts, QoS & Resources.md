## Part 2 – Nodes, Partitions, Accounts, QoS & Resources

* [7.7 Nodes](#77-nodes)
* [7.8 Node States](#78-node-states)
* [7.9 Partitions](#79-partitions)
* [7.10 Accounts](#710-accounts)
* [7.11 Quality of Service](#711-quality-of-service)
* [7.12 Resources](#712-resources)
* [7.13 CPU and Memory Allocation](#713-cpu-and-memory-allocation)
* [7.14 GPU Resources](#714-gpu-resources)
* [7.15 Resource Allocation Example](#715-resource-allocation-example)
* [7.16 Part 2 Quick Revision](#716-part-2-quick-revision)

---

# 7.7 Nodes

In Slurm, a **node** represents a compute resource managed by the scheduler.

A node can contain:

```text
CPU
Memory
GPU
Local Storage
Network Interfaces
```

Example:

```text id="n1q9x0"
gpu001
 ├── 2 × CPU
 ├── 512 GB RAM
 ├── 4 × NVIDIA GPU
 └── InfiniBand HCA
```

Slurm uses the node definition to determine what resources are available.

---

## View Nodes

```bash
sinfo
```

Detailed information:

```bash
scontrol show node gpu001
```

Example:

```text
NodeName=gpu001
CPUTot=128
RealMemory=512000
Gres=gpu:4
State=IDLE
```

---

# 7.8 Node States

Slurm maintains a state for each node.

Common states include:

| State       | Meaning                    |
| ----------- | -------------------------- |
| `IDLE`      | Available for jobs         |
| `ALLOCATED` | Resources are being used   |
| `MIXED`     | Some resources allocated   |
| `DOWN`      | Node unavailable           |
| `DRAIN`     | No new jobs accepted       |
| `DRAINING`  | Existing jobs finishing    |
| `RESERVED`  | Reserved for specific use  |
| `UNKNOWN`   | State cannot be determined |

Check:

```bash
sinfo -N
```

Detailed:

```bash
scontrol show node compute001
```

---

## DRAIN vs DOWN

This distinction is important.

### DRAIN

Used when the node should stop accepting new jobs.

Example:

```bash
scontrol update NodeName=compute001 State=DRAIN Reason="Maintenance"
```

Existing jobs may finish.

### DOWN

Indicates that the node is unavailable.

```bash
scontrol update NodeName=compute001 State=DOWN Reason="Hardware failure"
```

Think:

```text
DRAIN → Planned / controlled removal

DOWN  → Unavailable / failure
```

---

# 7.9 Partitions

A **partition** is a logical group of nodes.

It can be thought of as a queue.

Example:

```text id="7p4h2s"
Cluster
   │
   ├── cpu
   │    ├── cpu001
   │    ├── cpu002
   │    └── cpu003
   │
   ├── gpu
   │    ├── gpu001
   │    └── gpu002
   │
   └── debug
        └── test001
```

---

## View Partitions

```bash
sinfo
```

Detailed:

```bash
scontrol show partition
```

Example:

```text
PartitionName=gpu
Nodes=gpu[001-020]
Default=NO
MaxTime=7-00:00:00
State=UP
```

---

## Why Use Partitions?

Different workloads need different policies.

Example:

| Partition | Purpose               |
| --------- | --------------------- |
| `cpu`     | General CPU workloads |
| `gpu`     | GPU workloads         |
| `debug`   | Short testing         |
| `large`   | Large parallel jobs   |

---

# 7.10 Accounts

Slurm accounts are used to organize users and resource usage.

Conceptually:

```text id="5b7q2s"
Research Group
      │
      ├── User A
      ├── User B
      └── User C
```

The account can be associated with:

* Users
* Partitions
* QoS
* Usage
* Fair-share

View accounts:

```bash
sacctmgr show account
```

View users:

```bash
sacctmgr show user
```

---

## Example

```text id="6b4p1k"
Account: ai_research

Users:
satish
user01
user02
```

Submit:

```bash
sbatch --account=ai_research job.sh
```

---

# 7.11 Quality of Service

**QoS** controls additional scheduling and resource policies.

QoS can define limits such as:

* Maximum jobs
* Maximum CPUs
* Maximum GPUs
* Maximum runtime
* Priority
* Fair-share behavior

Conceptually:

```text id="i8zqv0"
User
  ↓
Account
  ↓
QoS
  ↓
Partition
  ↓
Resources
```

View QoS:

```bash
sacctmgr show qos
```

---

## Example QoS

```text
normal
debug
high
gpu
```

A job can request:

```bash
sbatch --qos=debug job.sh
```

A common HPC policy might be:

```text
debug
 └── Short runtime

normal
 └── Standard workload

high
 └── Higher priority / restricted access
```

---

# 7.12 Resources

Slurm allocates resources rather than simply "starting programs."

Typical resources:

```text
CPU
Memory
GPU
Node
Time
```

Example request:

```bash
srun --nodes=2 --ntasks=64 --mem=128G hostname
```

This means the job requests resources according to the specified allocation parameters.

---

# 7.13 CPU and Memory Allocation

## CPUs

Request CPUs:

```bash
sbatch --cpus-per-task=8 job.sh
```

Request tasks:

```bash
sbatch --ntasks=16 job.sh
```

For distributed MPI jobs:

```bash
sbatch --nodes=4 --ntasks-per-node=32 job.sh
```

Conceptually:

```text id="6xq6mb"
4 Nodes
 │
 ├── 32 Tasks
 ├── 32 Tasks
 ├── 32 Tasks
 └── 32 Tasks
```

---

## Memory

Request memory:

```bash
sbatch --mem=64G job.sh
```

Per CPU:

```bash
sbatch --mem-per-cpu=4G job.sh
```

The exact effective allocation depends on the cluster configuration and consumable-resource settings.

---

# 7.14 GPU Resources

Modern HPC-AI clusters require Slurm to manage GPUs.

GPU resources are commonly represented using **GRES**.

Example:

```text id="x2z6la"
Node
 ├── CPU
 ├── Memory
 └── GPU × 4
```

Check:

```bash
scontrol show node gpu001
```

You may see:

```text
Gres=gpu:4
```

---

## Request One GPU

```bash
srun --gpus=1 nvidia-smi
```

Batch:

```bash
sbatch --gpus=1 gpu_job.sh
```

Request four GPUs:

```bash
sbatch --gpus=4 gpu_job.sh
```

---

## GPU Type

Clusters may expose typed GPU resources.

Example:

```bash
sbatch --gpus=a100:2 gpu_job.sh
```

The exact GPU resource names depend on the site's `gres.conf` and Slurm configuration.

---

## GPU Allocation Flow

```text id="pjc3vv"
User
 ↓
sbatch --gpus=1
 ↓
Slurm Scheduler
 ↓
Find GPU Node
 ↓
Allocate GPU
 ↓
Job Starts
 ↓
Application Uses GPU
```

Slurm controls the allocation; the application/framework uses the GPU.

---

# 7.15 Resource Allocation Example

Suppose a user wants:

```text
2 Nodes
8 Tasks per Node
4 GPUs per Node
64 GB Memory per Node
```

A request could be:

```bash
sbatch \
  --nodes=2 \
  --ntasks-per-node=8 \
  --gpus-per-node=4 \
  --mem=64G \
  job.sh
```

The resulting conceptual allocation is:

```text id="9m0v2k"
             Job
              │
       ┌──────┴──────┐
       ▼             ▼
    Node 01        Node 02
       │             │
   8 Tasks        8 Tasks
   4 GPUs         4 GPUs
   64 GB          64 GB
```

---

# 7.16 Part 2 Quick Revision

### Node

```text
Node → Compute resource
```

### Partition

```text
Partition → Group/queue of nodes
```

### Account

```text
Account → Organizational/resource usage identity
```

### QoS

```text
QoS → Scheduling and resource policy
```

### Resources

```text
CPU
Memory
GPU
Nodes
Time
```

---

## Important Commands

```bash
sinfo
sinfo -N
scontrol show node <node>
scontrol show partition
scontrol update NodeName=<node> State=DRAIN
sacctmgr show account
sacctmgr show user
sacctmgr show qos
```

### Resource Requests

```bash
sbatch --cpus-per-task=8 job.sh
sbatch --mem=64G job.sh
sbatch --nodes=2 job.sh
sbatch --gpus=1 job.sh
```

---

## Engineer Mental Model

Remember the relationship:

```text id="8x7b3n"
                  Slurm
                    │
          ┌─────────┼─────────┐
          ▼         ▼         ▼
        Nodes   Partitions   Accounts
          │         │          │
          └─────────┼──────────┘
                    ▼
                   QoS
                    │
                    ▼
                Resources
                    │
          ┌─────────┼─────────┐
          ▼         ▼         ▼
         CPU       RAM       GPU
                    │
                    ▼
                   Job
```

The key idea is:

> **Slurm decides which resources a job receives, when it runs, and where it runs according to cluster policy.**

---

# End of Part 2
