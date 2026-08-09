## Part 4 – Scheduling, Accounting & Production Operations

* [7.30 How Slurm Schedules Jobs](#730-how-slurm-schedules-jobs)
* [7.31 Job Priority](#731-job-priority)
* [7.32 Backfill Scheduling](#732-backfill-scheduling)
* [7.33 Fair-Share](#733-fair-share)
* [7.34 Slurm Accounting](#734-slurm-accounting)
* [7.35 slurmdbd](#735-slurmdbd)
* [7.36 Monitoring Cluster Health](#736-monitoring-cluster-health)
* [7.37 Slurm and xCAT](#737-slurm-and-xcat)
* [7.38 Slurm and GPU Infrastructure](#738-slurm-and-gpu-infrastructure)
* [7.39 Production Operations](#739-production-operations)
* [7.40 Part 4 Quick Revision](#740-part-4-quick-revision)

---

# 7.30 How Slurm Schedules Jobs

Slurm continuously evaluates:

```text id="q1x3jz"
Pending Jobs
     │
     ▼
Available Resources
     │
     ▼
Scheduling Policy
     │
     ├── Priority
     ├── Fair-share
     ├── Partition
     ├── QoS
     ├── Resource requirements
     └── Time limits
     │
     ▼
Resource Allocation
     │
     ▼
Job Starts
```

The scheduler does not simply run jobs in submission order.

A job may remain pending because:

* Required resources are unavailable
* Higher-priority jobs are ahead
* Requested GPU type is unavailable
* Account/QoS limits are reached
* Partition limits are reached
* Job dependency is not satisfied
* Requested time/resources cannot currently be scheduled

---

# 7.31 Job Priority

Slurm assigns priorities to pending jobs.

Check:

```bash
sprio
```

Example:

```text id="2m2d4b"
JOBID   PRIORITY
1001    150000
1002    120000
1003     85000
```

A simplified mental model:

```text id="g3l4m8"
Priority
   │
   ├── Age
   ├── Fair-share
   ├── Job size
   ├── QoS
   └── Other configured factors
```

The exact priority calculation depends on the cluster's Slurm configuration.

---

# 7.32 Backfill Scheduling

Backfill allows smaller jobs to run while a higher-priority job waits for its reserved resources.

Example:

```text id="s3h2tq"
Time ───────────────────────────────>

Large Job
████████████████████████

Small Job
      █████

Another Small Job
             █████
```

The scheduler can run short jobs in available gaps if doing so does not delay higher-priority reservations.

This improves cluster utilization.

### Why it matters

Without backfill:

```text id="x4h4t1"
Large Job Waiting
      ↓
Idle Resources
      ↓
Poor Utilization
```

With backfill:

```text id="7q7l0j"
Large Job Waiting
      ↓
Available Gap
      ↓
Small Job Runs
      ↓
Better Utilization
```

---

# 7.33 Fair-Share

Fair-share helps distribute cluster resources among users or groups.

Example:

```text id="l9v0gz"
Research Group A → Heavy recent usage
Research Group B → Low recent usage
```

The scheduler can reduce the relative priority of A and favor B according to configured fair-share policy.

Check:

```bash id="ivb7wq"
sshare
```

Example:

```text id="6f3qcv"
Account       FairShare
research_a    0.50
research_b    1.00
```

The exact values and interpretation depend on the site's configuration.

### Goal

```text id="l9n6ph"
Fair resource distribution
        +
High cluster utilization
        =
Efficient HPC environment
```

---

# 7.34 Slurm Accounting

Accounting records what happened to jobs.

Useful information includes:

```text id="c5d7kf"
Job ID
User
Account
Partition
Start Time
End Time
State
CPU usage
Memory usage
Exit Code
GPU resources
```

Basic command:

```bash id="u5p5qs"
sacct
```

Specific job:

```bash id="8is3fc"
sacct -j 12345
```

Useful format:

```bash id="u1opj9"
sacct -j 12345 \
  --format=JobID,JobName,User,Account,Partition,State,Elapsed,AllocCPUS,MaxRSS,ExitCode
```

---

# 7.35 slurmdbd

`slurmdbd` provides the interface between Slurm and the accounting database.

Architecture:

```text id="3m8o5v"
             slurmctld
                 │
                 ▼
              slurmdbd
                 │
                 ▼
          Accounting Database
```

Check:

```bash id="1q6g9w"
systemctl status slurmdbd
```

Logs:

```bash id="9sk0e7"
journalctl -u slurmdbd
```

The accounting database is useful for:

* Historical job records
* Usage analysis
* Account management
* Fair-share information
* Resource utilization reports

---

# 7.36 Monitoring Cluster Health

An HPC administrator should continuously monitor:

```text id="e0v7xn"
Controller
Compute Nodes
Partitions
Jobs
CPU
Memory
GPU
Network
Storage
```

### Cluster overview

```bash id="7h3v5v"
sinfo
```

### Nodes

```bash id="p9cnm5"
sinfo -N
```

### Jobs

```bash id="7lkn9v"
squeue
```

### Controller

```bash id="8d3t7a"
systemctl status slurmctld
```

### Compute node

```bash id="4v6x1f"
systemctl status slurmd
```

---

# 7.37 Slurm and xCAT

In a production HPC cluster, xCAT and Slurm have different responsibilities.

```text id="v0z5bh"
                  xCAT
                   │
             Provision Node
                   │
                   ▼
              Linux Node
                   │
           Configure Software
                   │
                   ▼
                 slurmd
                   │
                   ▼
               slurmctld
                   │
                   ▼
              Schedule Jobs
```

### Remember

> **xCAT manages node provisioning; Slurm manages workload execution.**

Example lifecycle:

```text id="t5m8h7"
New Hardware
     ↓
xCAT Discovery
     ↓
OS Provisioning
     ↓
Ansible Configuration
     ↓
HPC Software
     ↓
slurmd
     ↓
Slurm Registration
     ↓
Node Available
```

---

# 7.38 Slurm and GPU Infrastructure

For AI clusters, Slurm becomes a critical GPU resource manager.

Example:

```text id="l8l8ip"
                 Slurm
                   │
             GPU Allocation
                   │
        ┌──────────┼──────────┐
        ▼          ▼          ▼
      GPU01      GPU02      GPU03
        │          │          │
        └──────────┼──────────┘
                   ▼
              AI Workload
```

A user may request:

```bash id="2b7g9x"
sbatch --partition=gpu --gpus=4 train.sh
```

Slurm determines where the GPUs are available and allocates them according to cluster policy.

Check node GPU configuration:

```bash id="n3kqbd"
scontrol show node gpu001
```

Check from the allocated node:

```bash id="v5a7o8"
nvidia-smi
```

---

## GPU Troubleshooting Relationship

If a GPU job fails:

```text id="8i6n5x"
Slurm Allocation
      ↓
Node
      ↓
GPU Visibility
      ↓
NVIDIA Driver
      ↓
CUDA
      ↓
Application
```

Check:

```bash id="4h7x4j"
echo $CUDA_VISIBLE_DEVICES
```

and:

```bash id="q4g5b8"
nvidia-smi
```

The exact GPU visibility behavior depends on Slurm's configured GRES/device-management integration.

---

# 7.39 Production Operations

## Draining a Node

Before maintenance:

```bash id="m6o6f8"
scontrol update NodeName=compute001 \
State=DRAIN \
Reason="Scheduled maintenance"
```

Check:

```bash id="q0y3xv"
sinfo -N
```

After maintenance:

```bash id="hj7d8f"
scontrol update NodeName=compute001 State=RESUME
```

---

## Checking Why a Job Is Pending

First:

```bash id="k3d0r8"
squeue -j 12345
```

Then:

```bash id="s4n8u9"
scontrol show job 12345
```

Look for:

```text id="3h9w1n"
Reason=
```

Common reasons include:

```text
Resources
Priority
QOS
Dependency
Partition
AssocMaxJobsLimit
ReqNodeNotAvail
```

---

## Checking a Node Problem

```bash id="5z2l9d"
scontrol show node compute001
```

Then on the node:

```bash id="j8f5nm"
systemctl status slurmd
```

Check logs:

```bash id="o4p8h7"
journalctl -u slurmd
```

Then validate:

```bash id="m6f6jm"
hostname
uptime
free -h
lscpu
```

For GPU nodes:

```bash id="1q5q4d"
nvidia-smi
```

For InfiniBand:

```bash id="q1k7pv"
ibstat
```

---

# 7.40 Part 4 Quick Revision

## Scheduling

```text id="3n6n9k"
Pending Jobs
     ↓
Priority / Policy
     ↓
Available Resources
     ↓
Scheduler
     ↓
Job Allocation
```

---

## Important Scheduling Concepts

| Concept    | Purpose                          |
| ---------- | -------------------------------- |
| Priority   | Determines scheduling order      |
| Backfill   | Uses available gaps efficiently  |
| Fair-share | Balances resource usage          |
| QoS        | Applies resource/policy limits   |
| Partition  | Defines resource pool and policy |

---

## Accounting

```text id="x9x9b1"
Job
 ↓
slurmctld
 ↓
slurmdbd
 ↓
Database
 ↓
Historical Usage
```

---

## Production Mental Model

When a job is not running:

```text id="z7v4y3"
Job Pending
    │
    ▼
Check squeue
    │
    ▼
Check Reason
    │
    ▼
scontrol show job
    │
    ├── Priority?
    ├── Resources?
    ├── QoS?
    ├── Account?
    ├── Dependency?
    └── Node?
```

When a node is unhealthy:

```text id="8b8l6g"
Slurm Node
    ↓
slurmd
    ↓
Linux
    ↓
Network
    ↓
Storage
    ↓
GPU / IB
```

---

## Essential Commands

```bash
sinfo
squeue
scontrol show job <jobid>
scontrol show node <node>
scontrol show partition
sprio
sshare
sacct
sacctmgr
scontrol update
systemctl status slurmctld
systemctl status slurmd
systemctl status slurmdbd
```

---

## HPC-AI Engineer Perspective

For Slurm, remember three layers:

```text id="2p5y2g"
                    Slurm
                      │
          ┌───────────┼───────────┐
          ▼           ▼           ▼
      Scheduling   Resources   Accounting
          │           │           │
          ▼           ▼           ▼
       Priority    CPU/GPU      Usage
       Backfill    Memory       History
       Fair-share  Nodes         Reports
```

Your goal as an HPC-AI Infrastructure Engineer is not merely to know `sbatch`.

You should be able to answer:

> **Why is this job pending?**

> **Why is this node DOWN/DRAIN?**

> **Why is the GPU job not getting a GPU?**

> **Why is cluster utilization low?**

> **Why is one research group consuming disproportionate resources?**

Those questions represent real Slurm administration work.

---

# End of Part 4
