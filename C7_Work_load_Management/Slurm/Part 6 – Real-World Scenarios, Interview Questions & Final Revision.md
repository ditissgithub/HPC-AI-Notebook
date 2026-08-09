## Part 6 – Real-World Scenarios, Interview Questions & Final Revision

* [7.51 Real-World HPC Scenario 1 – Node Not Available](#751-real-world-hpc-scenario-1--node-not-available)
* [7.52 Real-World HPC Scenario 2 – Job Stuck in Pending](#752-real-world-hpc-scenario-2--job-stuck-in-pending)
* [7.53 Real-World HPC Scenario 3 – GPU Job Failure](#753-real-world-hpc-scenario-3--gpu-job-failure)
* [7.54 Real-World HPC Scenario 4 – Low Cluster Utilization](#754-real-world-hpc-scenario-4--low-cluster-utilization)
* [7.55 Real-World HPC Scenario 5 – Node Configuration Mismatch](#755-real-world-hpc-scenario-5--node-configuration-mismatch)
* [7.56 Slurm Production Checklist](#756-slurm-production-checklist)
* [7.57 Slurm Interview Questions](#757-slurm-interview-questions)
* [7.58 Essential Slurm Commands](#758-essential-slurm-commands)
* [7.59 Slurm Architecture – Final View](#759-slurm-architecture--final-view)
* [7.60 Chapter 7 Final Revision](#760-chapter-7-final-revision)

---

# 7.51 Real-World HPC Scenario 1 – Node Not Available

### Situation

A compute node suddenly appears as `DOWN`.

```bash
sinfo -N
```

Example:

```text
compute021    down
```

### Step 1 – Check Slurm

```bash
scontrol show node compute021
```

Look for:

```text
State=
Reason=
```

### Step 2 – Check the node

```bash
ssh compute021
```

```bash
systemctl status slurmd
```

### Step 3 – Check system

```bash
uptime
free -h
df -h
```

### Step 4 – Check network

```bash
ping <slurm-controller>
hostname -f
```

### Step 5 – Check HPC hardware

```bash
ibstat
```

For GPU nodes:

```bash
nvidia-smi
```

### Recovery

After fixing the actual problem:

```bash
scontrol update NodeName=compute021 State=RESUME
```

### Engineering approach

```text
DOWN
 ↓
Reason
 ↓
slurmd
 ↓
Linux
 ↓
Network
 ↓
Hardware
 ↓
Fix
 ↓
RESUME
```

---

# 7.52 Real-World HPC Scenario 2 – Job Stuck in Pending

### Situation

A user reports:

> "My job has been pending for several hours."

Check:

```bash
squeue -j <jobid>
```

Then:

```bash
scontrol show job <jobid>
```

Look at:

```text
Reason=
```

### Case 1 – Resources

```text
Reason=Resources
```

Check:

```text
Available CPUs
Available memory
Available GPUs
Available nodes
Requested time
```

### Case 2 – Priority

```text
Reason=Priority
```

Check:

```bash
sprio
```

### Case 3 – QoS

```text
Reason=QOS
```

Check:

```bash
sacctmgr show qos
```

### Case 4 – Dependency

The job may be waiting for another job.

Check:

```bash
scontrol show job <jobid>
```

### Engineering principle

Never tell the user simply:

> "Slurm is busy."

Identify the exact scheduling reason.

---

# 7.53 Real-World HPC Scenario 3 – GPU Job Failure

### Situation

A user submits:

```bash
sbatch --partition=gpu --gpus=1 train.sh
```

The job starts, but the application reports:

```text
CUDA device not found
```

Troubleshoot in layers.

### 1. Did Slurm allocate a GPU?

```bash
scontrol show job <jobid>
```

### 2. Is GPU visible?

```bash
nvidia-smi
```

### 3. Check visibility

```bash
echo $CUDA_VISIBLE_DEVICES
```

### 4. Check driver

```bash
nvidia-smi
```

### 5. Check hardware

```bash
lspci | grep -i nvidia
```

### 6. Check CUDA/application

```bash
python -c "import torch; print(torch.cuda.is_available())"
```

if PyTorch is installed.

### Troubleshooting model

```text
Slurm
 ↓
GRES
 ↓
GPU allocation
 ↓
CUDA_VISIBLE_DEVICES
 ↓
NVIDIA driver
 ↓
CUDA
 ↓
Framework
 ↓
Application
```

---

# 7.54 Real-World HPC Scenario 4 – Low Cluster Utilization

### Situation

A 1,000-node cluster is available, but CPU utilization is only 30%.

First check:

```bash
sinfo
```

Then:

```bash
squeue
```

Look at pending jobs:

```bash
squeue -t PD
```

Check scheduling priorities:

```bash
sprio
```

Check fair-share:

```bash
sshare
```

Possible causes:

```text
Large resource requests
Long requested time limits
Partition restrictions
QoS limits
Insufficient backfill opportunities
User behavior
Fragmented resources
```

Example:

```text
Job requests:
128 CPUs
7 days

Available:
64 CPUs
12 hours
```

The available resources may remain unused because they cannot satisfy the job's requested allocation.

### Engineering goal

Optimize:

```text
Resource Utilization
        +
Fairness
        +
Job Throughput
```

Not simply CPU utilization.

---

# 7.55 Real-World HPC Scenario 5 – Node Configuration Mismatch

### Situation

A node has:

```text
128 CPUs
512 GB RAM
4 GPUs
```

But Slurm reports different resources.

Check:

```bash
scontrol show node gpu001
```

On the node:

```bash
slurmd -C
```

Compare the detected hardware with the configured node definition.

Also inspect:

```text
slurm.conf
gres.conf
```

Potential causes:

* Hardware replacement
* Configuration drift
* Incorrect CPU topology
* Incorrect memory configuration
* GPU configuration mismatch
* Configuration not distributed to the node

### Engineering principle

The scheduler's resource model must match the physical node.

---

# 7.56 Slurm Production Checklist

## Controller

* [ ] `slurmctld` running
* [ ] Configuration valid
* [ ] Logs monitored
* [ ] HA/recovery strategy defined

## Compute Nodes

* [ ] `slurmd` running
* [ ] Nodes registered
* [ ] Correct CPU count
* [ ] Correct memory
* [ ] GPU nodes correctly configured
* [ ] InfiniBand healthy

## Scheduler

* [ ] Partitions configured
* [ ] QoS configured
* [ ] Priority policy understood
* [ ] Backfill functioning
* [ ] Fair-share configured where required

## Accounting

* [ ] `slurmdbd` running
* [ ] Database reachable
* [ ] Job accounting working
* [ ] Accounts/users configured

## GPU

* [ ] NVIDIA driver
* [ ] CUDA environment
* [ ] GRES configuration
* [ ] GPU allocation tested
* [ ] GPU health monitored

## Operations

* [ ] Maintenance uses DRAIN
* [ ] Configuration managed centrally
* [ ] Logs retained
* [ ] Troubleshooting procedures documented

---

# 7.57 Slurm Interview Questions

## Basic

### 1. What is Slurm?

A workload manager and scheduler used to allocate HPC resources and execute jobs according to scheduling policies.

### 2. What is `slurmctld`?

The central Slurm controller responsible for scheduling and cluster state management.

### 3. What is `slurmd`?

The daemon running on compute nodes that executes jobs and reports node status.

### 4. What is `slurmdbd`?

The Slurm database daemon responsible for accounting integration.

### 5. Difference between `sbatch` and `srun`?

```text
sbatch → Submit batch job
srun   → Launch job step / interactive workload
```

---

## Intermediate

### 6. What is a partition?

A logical group of nodes with a common scheduling policy.

### 7. What is QoS?

A mechanism for applying resource and scheduling policies to jobs/users/accounts.

### 8. What is fair-share?

A scheduling mechanism that helps distribute resources according to configured usage and shares.

### 9. What is backfill?

A scheduling technique that runs suitable short jobs in available resource gaps without delaying higher-priority jobs.

### 10. What is a node in Slurm?

A compute resource managed by Slurm.

---

## Production

### 11. A job is pending. How do you troubleshoot?

```bash
squeue -j <jobid>
scontrol show job <jobid>
```

Check the `Reason=` field, then investigate resources, priority, QoS, dependency, partition, or node availability.

### 12. A node is DOWN. What do you do?

```bash
scontrol show node <node>
systemctl status slurmd
journalctl -u slurmd
```

Then check Linux, network, GPU, InfiniBand and hardware.

### 13. How do you safely perform maintenance?

Drain the node:

```bash
scontrol update NodeName=<node> State=DRAIN Reason="Maintenance"
```

Perform maintenance and resume it afterward:

```bash
scontrol update NodeName=<node> State=RESUME
```

### 14. How do you troubleshoot a GPU job?

Check:

```text
Slurm allocation
 → GRES
 → GPU visibility
 → nvidia-smi
 → Driver
 → CUDA
 → Application
```

### 15. How do you improve cluster utilization?

Investigate:

```text
Scheduling
Backfill
Fair-share
QoS
Partition design
Resource requests
Job sizes
Time limits
```

---

# 7.58 Essential Slurm Commands

## Daily User Commands

```bash
sinfo
squeue
sbatch
srun
scancel
sacct
```

## Job Investigation

```bash
scontrol show job <jobid>
sacct -j <jobid>
sprio
```

## Node Investigation

```bash
scontrol show node <node>
sinfo -N
```

## Partition

```bash
scontrol show partition
```

## Account / QoS

```bash
sacctmgr show account
sacctmgr show user
sacctmgr show qos
sshare
```

## Services

```bash
systemctl status slurmctld
systemctl status slurmd
systemctl status slurmdbd
```

## Logs

```bash
journalctl -u slurmctld
journalctl -u slurmd
journalctl -u slurmdbd
```

## GPU

```bash
nvidia-smi
```

## InfiniBand

```bash
ibstat
rdma link
```

---

# 7.59 Slurm Architecture – Final View

The complete HPC-AI workload path can be remembered as:

```text
                    USER
                     │
                     ▼
             sbatch / srun
                     │
                     ▼
                slurmctld
                     │
        ┌────────────┼────────────┐
        ▼            ▼            ▼
    Scheduler      Nodes       Accounting
        │            │            │
        │         slurmd       slurmdbd
        │            │            │
        ▼            ▼            ▼
    Priority      Compute      Database
    Backfill        Node
    Fair-share       │
                     │
            ┌────────┼────────┐
            ▼        ▼        ▼
           CPU      GPU      RAM
            │        │        │
            └────────┼────────┘
                     ▼
                 HPC / AI Job
```

---

# 7.60 Chapter 7 Final Revision

## Slurm in One Sentence

> **Slurm allocates cluster resources and schedules HPC-AI workloads according to resource requirements and scheduling policies.**

## Three Core Services

```text
slurmctld → Scheduling / Control
slurmd    → Node / Job Execution
slurmdbd  → Accounting
```

## Three Core Commands

```text
sbatch  → Submit
squeue  → Monitor
sacct   → History
```

## Three Core Resources

```text
CPU
Memory
GPU
```

## Three Important Scheduling Concepts

```text
Priority
Backfill
Fair-share
```

## Three Common Problems

```text
Job Pending
Node DOWN/DRAIN
GPU Not Available
```

## Universal Troubleshooting Pattern

```text
Problem
   ↓
Identify State
   ↓
Check Reason
   ↓
Check Slurm
   ↓
Check Linux
   ↓
Check Network
   ↓
Check GPU / IB / Storage
   ↓
Fix
   ↓
Validate
```

---

# Chapter 7 – Final Mental Model

As an HPC-AI Infrastructure Engineer, think of Slurm as the **resource control layer** between users and the physical cluster:

```text
                  HPC / AI Users
                        │
                        ▼
                     Slurm
                        │
        ┌───────────────┼───────────────┐
        ▼               ▼               ▼
    Scheduling       Allocation      Accounting
        │               │               │
        ▼               ▼               ▼
    Priority        CPU/GPU/RAM       Usage
    Backfill        Nodes             History
    Fair-share      Time              Reports
                        │
                        ▼
                Compute Infrastructure
                        │
        ┌───────────────┼───────────────┐
        ▼               ▼               ▼
       CPU             GPU             Network
        │               │               │
        └───────────────┼───────────────┘
                        ▼
                   HPC / AI Workload
```

**Core principle:**

> **Provision the infrastructure with xCAT, configure it with automation, expose resources through Slurm, and let Slurm control how those resources are consumed.**

---

# Chapter 7 Checklist

* [x] Slurm architecture
* [x] `slurmctld`
* [x] `slurmd`
* [x] `slurmdbd`
* [x] Nodes
* [x] Node states
* [x] Partitions
* [x] Accounts
* [x] QoS
* [x] CPU allocation
* [x] Memory allocation
* [x] GPU allocation
* [x] Interactive jobs
* [x] Batch jobs
* [x] Job arrays
* [x] Job dependencies
* [x] Scheduling
* [x] Priority
* [x] Backfill
* [x] Fair-share
* [x] Accounting
* [x] Production operations
* [x] Troubleshooting
* [x] HPC-AI scenarios
* [x] Interview preparation
* [x] Essential commands

---

**Chapter 7 – Slurm Complete**
