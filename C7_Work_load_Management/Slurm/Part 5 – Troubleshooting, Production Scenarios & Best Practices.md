## Part 5 – Troubleshooting, Production Scenarios & Best Practices

* [7.41 Common Node Problems](#741-common-node-problems)
* [7.42 Common Job Problems](#742-common-job-problems)
* [7.43 GPU Scheduling Problems](#743-gpu-scheduling-problems)
* [7.44 Job Pending Troubleshooting](#744-job-pending-troubleshooting)
* [7.45 Node DOWN or DRAIN Troubleshooting](#745-node-down-or-drain-troubleshooting)
* [7.46 slurmd Troubleshooting](#746-slurmd-troubleshooting)
* [7.47 Slurm Controller Troubleshooting](#747-slurm-controller-troubleshooting)
* [7.48 Production Troubleshooting Workflow](#748-production-troubleshooting-workflow)
* [7.49 Slurm Best Practices](#749-slurm-best-practices)
* [7.50 Part 5 Quick Revision](#750-part-5-quick-revision)

---

# 7.41 Common Node Problems

A compute node can fail at different layers.

```text
Hardware
   ↓
Operating System
   ↓
Network
   ↓
slurmd
   ↓
Slurm Controller
   ↓
Job Execution
```

Do not immediately assume Slurm is the problem.

---

## Check Node State

```bash
sinfo -N
```

Detailed:

```bash
scontrol show node compute001
```

Look for:

```text
State=
Reason=
CPUAlloc=
CPUTot=
RealMemory=
AllocMem=
Gres=
```

---

## Check slurmd

On the compute node:

```bash
systemctl status slurmd
```

Logs:

```bash
journalctl -u slurmd
```

Recent logs:

```bash
journalctl -u slurmd --since "30 minutes ago"
```

---

# 7.42 Common Job Problems

## Problem: Job Is Pending

Start with:

```bash
squeue -j <jobid>
```

Then:

```bash
scontrol show job <jobid>
```

Look for:

```text
Reason=
```

Common reasons:

```text
Resources
Priority
QOS
Dependency
Partition
ReqNodeNotAvail
AssocMaxJobsLimit
```

---

## Problem: Job Failed

Check:

```bash
sacct -j <jobid>
```

Useful format:

```bash
sacct -j <jobid> \
--format=JobID,State,Elapsed,ExitCode,AllocCPUS,MaxRSS
```

Then inspect the job output:

```bash
cat slurm-<jobid>.out
```

Possible causes:

* Application error
* Missing input
* Out-of-memory
* Invalid environment
* Permission problem
* Node failure
* GPU failure

---

# 7.43 GPU Scheduling Problems

GPU problems require checking both **Slurm** and the **NVIDIA stack**.

```text
Slurm
  ↓
GRES
  ↓
GPU Allocation
  ↓
NVIDIA Driver
  ↓
CUDA
  ↓
Application
```

---

## GPU Not Allocated

Check:

```bash
scontrol show node gpu001
```

Look for:

```text
Gres=gpu:...
```

Check the partition:

```bash
scontrol show partition gpu
```

Check the job:

```bash
scontrol show job <jobid>
```

---

## GPU Not Visible Inside Job

Check:

```bash
echo $CUDA_VISIBLE_DEVICES
```

Then:

```bash
nvidia-smi
```

If the node itself has a problem:

```bash
lspci | grep -i nvidia
```

```bash
nvidia-smi
```

Also check:

```bash
systemctl status nvidia-persistenced
```

where applicable.

---

# 7.44 Job Pending Troubleshooting

Use this workflow:

```text
Job Pending
     ↓
squeue -j JOBID
     ↓
scontrol show job JOBID
     ↓
Check Reason=
     ↓
Identify Layer
```

### Example

If:

```text
Reason=Resources
```

Investigate:

```text
Available Nodes
Available CPUs
Available Memory
Available GPUs
Requested Time
Partition
```

If:

```text
Reason=Priority
```

Check:

```bash
sprio
```

If:

```text
Reason=QOS
```

Check:

```bash
sacctmgr show qos
```

If:

```text
Reason=Dependency
```

Check the dependency in:

```bash
scontrol show job <jobid>
```

---

# 7.45 Node DOWN or DRAIN Troubleshooting

First:

```bash
scontrol show node compute001
```

Look for:

```text
State=
Reason=
```

Example:

```text
State=DRAIN
Reason=Hardware maintenance
```

This may be intentional.

---

## If Node Is Unexpectedly DOWN

Check the node:

```bash
ssh compute001
```

Then:

```bash
systemctl status slurmd
```

```bash
journalctl -u slurmd
```

Check basic system health:

```bash
uptime
free -h
df -h
```

Network:

```bash
ip addr
ping <slurm-controller>
```

DNS:

```bash
hostname -f
getent hosts <slurm-controller>
```

---

## GPU Node

```bash
nvidia-smi
```

## InfiniBand Node

```bash
ibstat
```

The troubleshooting sequence becomes:

```text
Node DOWN
   ↓
slurmd
   ↓
Linux
   ↓
Network
   ↓
GPU / IB / Storage
   ↓
Correct Problem
   ↓
Resume Node
```

After fixing the problem:

```bash
scontrol update NodeName=compute001 State=RESUME
```

---

# 7.46 slurmd Troubleshooting

`slurmd` is the execution daemon on compute nodes.

Check:

```bash
systemctl status slurmd
```

Logs:

```bash
journalctl -u slurmd
```

Common causes of failure:

* Configuration mismatch
* Authentication problem
* Network connectivity
* Hostname/DNS problem
* Resource mismatch
* Missing directories
* Permission problems
* Time synchronization issues

Check configuration consistency:

```bash
slurmd -C
```

This displays detected hardware information that can help compare the node against the Slurm configuration.

---

# 7.47 Slurm Controller Troubleshooting

The controller is a critical component.

Check:

```bash
systemctl status slurmctld
```

Logs:

```bash
journalctl -u slurmctld
```

Check cluster:

```bash
sinfo
```

Check configuration:

```bash
scontrol show config
```

If `slurmctld` is unavailable:

```text
Users
  ↓
Slurm Commands
  ↓
X
slurmctld
  ↓
Scheduling Stops
```

Compute nodes may continue existing workloads depending on configuration, but new scheduling decisions cannot proceed normally.

---

## Basic Controller Troubleshooting

```text
slurmctld problem
      ↓
Service status
      ↓
Logs
      ↓
Configuration
      ↓
Database/accounting
      ↓
Network
      ↓
Recover
      ↓
Validate
```

---

# 7.48 Production Troubleshooting Workflow

Use a **layered approach**.

## Layer 1 – Slurm

```bash
sinfo
squeue
scontrol show job <jobid>
scontrol show node <node>
```

## Layer 2 – Services

```bash
systemctl status slurmctld
systemctl status slurmd
systemctl status slurmdbd
```

## Layer 3 – Linux

```bash
uptime
free -h
df -h
top
```

## Layer 4 – Network

```bash
ip addr
ip route
ping <host>
ss -lntp
```

## Layer 5 – InfiniBand

```bash
ibstat
ibdev2netdev
rdma link
```

## Layer 6 – GPU

```bash
nvidia-smi
nvidia-smi topo -m
```

## Layer 7 – Storage

```bash
df -h
mount
```

For Lustre:

```bash
lfs df
```

---

# 7.49 Slurm Best Practices

## 1. Use Configuration Management

Keep Slurm configuration under version control.

```text
slurm.conf
gres.conf
cgroup.conf
slurmdbd.conf
```

Use automation such as Ansible to distribute configuration consistently.

---

## 2. Avoid Manual Changes

Do not repeatedly modify compute nodes manually.

Prefer:

```text
Automation
   ↓
Standard Configuration
   ↓
Validation
```

---

## 3. Use DRAIN for Planned Maintenance

Instead of immediately taking a healthy node down:

```bash
scontrol update NodeName=compute001 \
State=DRAIN \
Reason="Maintenance"
```

This prevents new jobs from being scheduled there.

---

## 4. Monitor Node Health

Monitor:

```text
CPU
Memory
GPU
Temperature
Filesystem
InfiniBand
Network
slurmd
```

A node being `IDLE` in Slurm does not automatically mean the hardware is healthy.

---

## 5. Keep Configuration Consistent

Important consistency areas:

```text
slurm.conf
Node definitions
Partition configuration
GRES
Cgroup configuration
Authentication
Time synchronization
```

---

## 6. Use Resource Limits

Avoid allowing a single workload to consume the entire cluster unintentionally.

Use:

```text
Partitions
Accounts
QoS
Fair-share
Time limits
CPU limits
Memory limits
GPU limits
```

---

## 7. Validate GPU Scheduling

For GPU clusters, verify the complete path:

```text
GPU Hardware
   ↓
NVIDIA Driver
   ↓
CUDA
   ↓
Slurm GRES
   ↓
GPU Allocation
   ↓
CUDA Application
```

---

# 7.50 Part 5 Quick Revision

## Job Problem

```text
Job Pending
    ↓
squeue
    ↓
scontrol show job
    ↓
Reason=
    ↓
Priority / Resource / QoS / Dependency
```

---

## Node Problem

```text
Node DOWN/DRAIN
      ↓
scontrol show node
      ↓
Reason=
      ↓
slurmd
      ↓
Linux
      ↓
Network / GPU / IB / Storage
```

---

## GPU Problem

```text
GPU Job
   ↓
Slurm Allocation
   ↓
GRES
   ↓
CUDA_VISIBLE_DEVICES
   ↓
nvidia-smi
   ↓
CUDA
```

---

## Controller Problem

```text
slurmctld
   ↓
systemctl
   ↓
journalctl
   ↓
Configuration
   ↓
Network / DB
```

---

## Most Useful Troubleshooting Commands

```bash
sinfo
squeue
scontrol show job <jobid>
scontrol show node <node>
scontrol show config

sprio
sshare
sacct
sacctmgr

systemctl status slurmctld
systemctl status slurmd
systemctl status slurmdbd

journalctl -u slurmctld
journalctl -u slurmd
journalctl -u slurmdbd

nvidia-smi
ibstat
df -h
free -h
```

---

# HPC-AI Engineer Mental Model

When troubleshooting Slurm, think:

```text
                    User Job
                       │
                       ▼
                    Slurm
                       │
             ┌─────────┴─────────┐
             ▼                   ▼
        Scheduler             Resources
             │                   │
             ▼           ┌───────┼───────┐
         slurmctld        CPU     GPU     RAM
             │
             ▼
           slurmd
             │
      ┌──────┼──────┐
      ▼      ▼      ▼
     Linux   IB    Lustre
```

The key production principle is:

> **Do not troubleshoot Slurm in isolation. Slurm sits on top of Linux, networking, storage, GPU and InfiniBand infrastructure.**

---

# End of Part 5
