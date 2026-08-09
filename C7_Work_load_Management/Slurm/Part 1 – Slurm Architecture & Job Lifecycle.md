### Part 1 – Slurm Architecture & Job Lifecycle

* [7.1 What is Slurm?](#71-what-is-slurm)
* [7.2 Why HPC Uses Slurm](#72-why-hpc-uses-slurm)
* [7.3 Slurm Architecture](#73-slurm-architecture)
* [7.4 Core Slurm Components](#74-core-slurm-components)
* [7.5 Job Lifecycle](#75-job-lifecycle)
* [7.6 Slurm Configuration Files](#76-slurm-configuration-files)

## 7.1 What is Slurm?

**Slurm** is an open-source workload manager and job scheduler widely used in HPC clusters.

Its primary responsibilities are:

```text
Users
  ↓
Submit Jobs
  ↓
Slurm
  ├── Allocate Resources
  ├── Schedule Jobs
  ├── Start Jobs
  ├── Monitor Jobs
  └── Release Resources
```

Slurm does **not** perform the actual scientific computation.

The application runs on compute nodes; Slurm controls **when, where, and with which resources** it runs.

---

# 7.2 Why HPC Uses Slurm

A cluster may contain:

```text
1000 CPU Nodes
200 GPU Nodes
10,000+ CPUs
Hundreds of GPUs
Petabytes of Storage
```

Without a scheduler, users could compete for resources manually.

Slurm provides:

* Resource allocation
* Job queuing
* Scheduling
* Fairness
* Priority
* Accounting
* Node management
* GPU allocation
* Job monitoring

The basic idea is:

```text
              HPC Cluster
                   │
              ┌────┴────┐
              │  Slurm  │
              └────┬────┘
          ┌─────────┼─────────┐
          ▼         ▼         ▼
       CPU Jobs   GPU Jobs  MPI Jobs
```

---

# 7.3 Slurm Architecture

A simplified architecture:

```text
                    User
                     │
                sbatch/srun
                     │
                     ▼
                slurmctld
               Controller
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
       Node01      Node02     GPU01
       slurmd      slurmd     slurmd
          │          │          │
          └──────────┼──────────┘
                     ▼
                  Jobs
```

For accounting:

```text
                    slurmctld
                        │
                        ▼
                    slurmdbd
                        │
                        ▼
                 Slurm Accounting DB
```

---

# 7.4 Core Slurm Components

## slurmctld

The central Slurm controller.

Responsibilities include:

* Scheduling
* Node state management
* Job state management
* Resource allocation
* Queue management

Check:

```bash
systemctl status slurmctld
```

---

## slurmd

Runs on compute nodes.

Responsibilities include:

* Accepting jobs from `slurmctld`
* Starting job steps
* Reporting node status
* Managing local job execution

Check:

```bash
systemctl status slurmd
```

---

## slurmdbd

Provides accounting integration.

```text
slurmctld
    ↓
slurmdbd
    ↓
Database
```

Useful for:

* Job history
* Usage accounting
* Accounts
* Associations
* Fair-share information

Check:

```bash
systemctl status slurmdbd
```

---

# 7.5 Job Lifecycle

A typical job goes through:

```text
SUBMITTED
    ↓
PENDING
    ↓
ALLOCATED
    ↓
RUNNING
    ↓
COMPLETING
    ↓
COMPLETED
```

If resources are unavailable:

```text
User
 ↓
sbatch
 ↓
PENDING
 ↓
Wait for Resources
 ↓
RUNNING
```

A job can also fail or be cancelled.

---

## Example

```bash
sbatch job.sh
```

Slurm returns a Job ID:

```text
Submitted batch job 12345
```

Check:

```bash
squeue -j 12345
```

After completion:

```bash
sacct -j 12345
```

---

# 7.6 Slurm Configuration Files

The most important configuration file is:

```text
/etc/slurm/slurm.conf
```

It defines cluster-wide configuration such as:

* Nodes
* Partitions
* Controller
* Scheduling
* Resource configuration

GPU configuration commonly uses:

```text
/etc/slurm/gres.conf
```

Accounting configuration commonly involves:

```text
/etc/slurm/slurmdbd.conf
```

A useful mental model:

```text
slurm.conf
    │
    ├── Cluster
    ├── Nodes
    ├── Partitions
    └── Scheduler
```

```text
gres.conf
    │
    └── GPUs / Generic Resources
```

```text
slurmdbd.conf
    │
    └── Accounting Database
```

---

# Part 1 – Quick Revision

Remember:

```text
slurmctld → Controls
slurmd    → Executes
slurmdbd  → Accounts
```

And:

```text
sbatch → Batch Job
srun   → Run Job
squeue → View Queue
sacct  → View History
```

---

# End of Part 1
