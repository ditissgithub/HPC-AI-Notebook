## Part 3 – Jobs, Job Scripts & Essential Commands

* [7.17 Interactive Jobs](#717-interactive-jobs)
* [7.18 Batch Jobs](#718-batch-jobs)
* [7.19 Common Job Options](#719-common-job-options)
* [7.20 Job Arrays](#720-job-arrays)
* [7.21 Job Dependencies](#721-job-dependencies)
* [7.22 Job Monitoring](#722-job-monitoring)
* [7.23 Job Cancellation](#723-job-cancellation)
* [7.24 Important User Commands](#724-important-user-commands)
* [7.25 Important Administrator Commands](#725-important-administrator-commands)
* [7.26 Basic Job Script](#726-basic-job-script)
* [7.27 GPU Job Script](#727-gpu-job-script)
* [7.28 MPI Job Example](#728-mpi-job-example)
* [7.29 Part 3 Quick Revision](#729-part-3-quick-revision)

---

# 7.17 Interactive Jobs

Interactive jobs are useful for:

* Testing
* Debugging
* Development
* Checking GPU availability
* Running short experiments

Example:

```bash
srun --partition=cpu --nodes=1 --ntasks=1 --cpus-per-task=4 --mem=8G --time=00:30:00 --pty bash
```

Slurm allocates the requested resources and starts a shell.

```text
User
 ↓
srun
 ↓
Slurm Scheduler
 ↓
Compute Node
 ↓
Interactive Shell
```

For GPU testing:

```bash
srun --partition=gpu --gpus=1 --time=00:15:00 --pty bash
```

Then:

```bash
nvidia-smi
```

---

# 7.18 Batch Jobs

Batch jobs are the standard way to run production workloads.

Instead of keeping a terminal open:

```text
Job Script
    ↓
sbatch
    ↓
Queue
    ↓
Scheduler
    ↓
Compute Node
    ↓
Application
```

Submit:

```bash
sbatch job.sh
```

Example response:

```text
Submitted batch job 12345
```

---

# 7.19 Common Job Options

Common `sbatch` options include:

| Option              | Purpose                 |
| ------------------- | ----------------------- |
| `--job-name`        | Job name                |
| `--partition`       | Partition               |
| `--nodes`           | Number of nodes         |
| `--ntasks`          | Number of tasks         |
| `--ntasks-per-node` | Tasks per node          |
| `--cpus-per-task`   | CPUs per task           |
| `--mem`             | Memory                  |
| `--time`            | Time limit              |
| `--output`          | Output file             |
| `--error`           | Error file              |
| `--account`         | Account                 |
| `--qos`             | QoS                     |
| `--gpus`            | GPUs                    |
| `--constraint`      | Node feature/constraint |

Example:

```bash
sbatch \
  --job-name=test \
  --partition=cpu \
  --nodes=1 \
  --ntasks=4 \
  --cpus-per-task=2 \
  --mem=16G \
  --time=01:00:00 \
  job.sh
```

---

# 7.20 Job Arrays

Job arrays are useful when the same application must run many times with different inputs.

Example:

```bash
sbatch --array=1-100 job.sh
```

This creates 100 array tasks.

Inside the script:

```bash
echo "Task ID: $SLURM_ARRAY_TASK_ID"
```

Conceptually:

```text
Array Job
 │
 ├── Task 1
 ├── Task 2
 ├── Task 3
 ├── ...
 └── Task 100
```

Typical use cases:

* Parameter sweeps
* ML experiments
* Dataset processing
* Monte Carlo simulations
* Hyperparameter searches

---

# 7.21 Job Dependencies

Dependencies allow jobs to execute in a specific order.

Example:

```text
Job A
 ↓
Job B
 ↓
Job C
```

Submit Job A:

```bash
jid=$(sbatch --parsable preprocess.sh)
```

Submit Job B after A completes:

```bash
sbatch --dependency=afterok:$jid train.sh
```

This is useful for AI pipelines:

```text
Data Preparation
      ↓
Training
      ↓
Evaluation
      ↓
Report
```

---

# 7.22 Job Monitoring

## View Queue

```bash
squeue
```

Specific user:

```bash
squeue -u $USER
```

Specific job:

```bash
squeue -j 12345
```

Useful compact view:

```bash
squeue -o "%.18i %.12P %.20j %.8u %.2t %.10M %.6D %R"
```

---

## Job Details

```bash
scontrol show job 12345
```

This is especially useful when a job is pending.

---

## Job History

```bash
sacct -j 12345
```

More detailed:

```bash
sacct -j 12345 --format=JobID,JobName,State,Elapsed,AllocCPUS,MaxRSS,ExitCode
```

---

## Job Efficiency

After completion:

```bash
seff 12345
```

If `seff` is installed, it provides a quick view of CPU and memory utilization.

---

# 7.23 Job Cancellation

Cancel a job:

```bash
scancel 12345
```

Cancel all jobs belonging to the current user:

```bash
scancel -u $USER
```

Cancel an array:

```bash
scancel 12345
```

Use cancellation carefully in production environments.

---

# 7.24 Important User Commands

### Cluster Information

```bash
sinfo
```

### Queue

```bash
squeue
```

### Submit

```bash
sbatch job.sh
```

### Interactive

```bash
srun --pty bash
```

### Job Details

```bash
scontrol show job <jobid>
```

### Job History

```bash
sacct -j <jobid>
```

### Cancel

```bash
scancel <jobid>
```

### Node Information

```bash
scontrol show node <node>
```

### Partition Information

```bash
scontrol show partition
```

---

# 7.25 Important Administrator Commands

Administrators frequently use:

```bash
scontrol show node
```

```bash
scontrol show partition
```

```bash
scontrol show config
```

```bash
scontrol show job <jobid>
```

For accounting:

```bash
sacctmgr show account
```

```bash
sacctmgr show user
```

```bash
sacctmgr show qos
```

---

## Node State Management

Drain:

```bash
scontrol update NodeName=compute001 State=DRAIN Reason="Maintenance"
```

Resume:

```bash
scontrol update NodeName=compute001 State=RESUME
```

Mark down:

```bash
scontrol update NodeName=compute001 State=DOWN Reason="Hardware failure"
```

---

# 7.26 Basic Job Script

A simple batch script:

```bash
#!/bin/bash
#SBATCH --job-name=test
#SBATCH --partition=cpu
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=4
#SBATCH --mem=8G
#SBATCH --time=00:30:00
#SBATCH --output=job-%j.out

echo "Job ID: $SLURM_JOB_ID"
echo "Node: $SLURMD_NODENAME"

hostname
date

./application
```

Submit:

```bash
sbatch job.sh
```

---

# 7.27 GPU Job Script

Example:

```bash
#!/bin/bash
#SBATCH --job-name=gpu-test
#SBATCH --partition=gpu
#SBATCH --nodes=1
#SBATCH --gpus=1
#SBATCH --cpus-per-task=8
#SBATCH --mem=32G
#SBATCH --time=01:00:00
#SBATCH --output=gpu-%j.out

echo "Job ID: $SLURM_JOB_ID"
echo "Node: $SLURMD_NODENAME"

nvidia-smi

python train.py
```

The allocation flow is:

```text
sbatch
  ↓
Slurm
  ↓
GPU Node
  ↓
GPU Allocation
  ↓
nvidia-smi
  ↓
Training
```

---

# 7.28 MPI Job Example

For distributed MPI workloads:

```bash
#!/bin/bash
#SBATCH --job-name=mpi-test
#SBATCH --partition=cpu
#SBATCH --nodes=4
#SBATCH --ntasks-per-node=32
#SBATCH --time=01:00:00
#SBATCH --output=mpi-%j.out

srun ./mpi_application
```

Conceptually:

```text
             MPI Job
                │
      ┌─────────┼─────────┐
      ▼         ▼         ▼
    Node1     Node2     Node3 ... Node4
      │         │         │
      └─────────┼─────────┘
                ▼
          MPI Communication
```

In HPC environments, the actual MPI launch mechanism should match the cluster's configured MPI stack and Slurm integration.

---

# 7.29 Part 3 Quick Revision

## Interactive

```bash
srun --pty bash
```

Use for:

```text
Testing
Debugging
Development
```

---

## Batch

```bash
sbatch job.sh
```

Use for:

```text
Production Workloads
Long Jobs
Reproducible Execution
```

---

## Monitor

```bash
squeue
scontrol show job <jobid>
sacct -j <jobid>
```

---

## Cancel

```bash
scancel <jobid>
```

---

## GPU

```bash
sbatch --gpus=1 gpu_job.sh
```

---

## Job Array

```bash
sbatch --array=1-100 job.sh
```

---

## Dependency

```bash
sbatch --dependency=afterok:<jobid> job.sh
```

---

## Engineer Mental Model

```text
                User
                  │
       ┌──────────┴──────────┐
       ▼                     ▼
     srun                   sbatch
       │                     │
       ▼                     ▼
 Interactive              Batch Job
       │                     │
       └──────────┬──────────┘
                  ▼
              slurmctld
                  │
                  ▼
              Scheduler
                  │
                  ▼
             Compute Node
                  │
       ┌──────────┼──────────┐
       ▼          ▼          ▼
      CPU        RAM        GPU
                  │
                  ▼
              Workload
```

### Most important commands to remember

```text
sinfo       → Cluster/partition information
squeue      → Running/pending jobs
sbatch      → Submit batch job
srun        → Run job/interactive workload
scontrol    → Administrative control/details
sacct       → Job accounting/history
scancel     → Cancel job
sacctmgr    → Account/QoS management
```

---

# Part 3 Learning Checklist

Before moving forward, you should be comfortable with:

* Interactive jobs
* Batch jobs
* Job scripts
* Resource requests
* GPU jobs
* MPI jobs
* Job arrays
* Dependencies
* Queue monitoring
* Job history
* Node state management
* Basic administrator commands

---

# End of Part 3
