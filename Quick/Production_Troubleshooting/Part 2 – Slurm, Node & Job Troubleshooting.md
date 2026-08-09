## Part 2 – Slurm, Node & Job Troubleshooting

> **Notebook focus:** Quickly diagnose common production problems involving Slurm controllers, compute nodes, jobs, partitions, QoS and node states.

* [17.13 Slurm Troubleshooting Model](#1713-slurm-troubleshooting-model)
* [17.14 Node Down or Drain](#1714-node-down-or-drain)
* [17.15 slurmd Not Running](#1715-slurmd-not-running)
* [17.16 Node Not Registering](#1716-node-not-registering)
* [17.17 Job Pending](#1717-job-pending)
* [17.18 Job Failed](#1718-job-failed)
* [17.19 Job Out of Memory](#1719-job-out-of-memory)
* [17.20 Job Timeout](#1720-job-timeout)
* [17.21 Partition Problems](#1721-partition-problems)
* [17.22 QoS and Account Problems](#1722-qos-and-account-problems)
* [17.23 Slurm Controller Troubleshooting](#1723-slurm-controller-troubleshooting)
* [17.24 Production Recovery Workflow](#1724-production-recovery-workflow)
* [17.25 Quick Revision](#1725-quick-revision)

# 17.13 Slurm Troubleshooting Model

A Slurm problem should be investigated through multiple layers:

```text
User
 │
 ▼
Job Submission
 │
 ▼
Slurmctld
 │
 ├── Account / Association
 ├── Partition
 ├── QoS
 ├── Priority
 └── Scheduler
 │
 ▼
Compute Node
 │
 └── Slurmd
      │
      ├── CPU
      ├── Memory
      ├── GPU
      ├── Network
      └── Storage
```

Do not troubleshoot only from `squeue`.

---

# 17.14 Node Down or Drain

Check:

```bash
sinfo -N -l
```

```bash
sinfo -R
```

Detailed node information:

```bash
scontrol show node <node>
```

Look for:

```text
State
Reason
CPUAlloc
CPUTot
RealMemory
AllocMem
Gres
CfgTRES
AllocTRES
```

Example:

```bash
scontrol show node compute01
```

Typical states:

```text
IDLE
ALLOCATED
MIXED
DOWN
DRAIN
DRAINING
FAIL
```

---

## Investigate the Reason

Example:

```text
State=DOWN Reason=Not responding
```

Check the node:

```bash
ssh compute01 hostname
```

Then:

```bash
systemctl status slurmd
```

```bash
journalctl -u slurmd -n 100
```

Also check:

```bash
uptime
free -h
df -hT
ip -br addr
```

For GPU nodes:

```bash
nvidia-smi
```

For IB:

```bash
ibstat
```

---

# 17.15 slurmd Not Running

Check:

```bash
systemctl status slurmd
```

If failed:

```bash
journalctl -u slurmd -n 100
```

Check configuration:

```bash
slurmd -C
```

This displays detected hardware configuration useful for comparing against Slurm configuration.

Check configuration file:

```bash
grep -v '^#' /etc/slurm/slurm.conf
```

Check whether the node can communicate with the controller:

```bash
getent hosts <slurmctld-host>
```

Check connectivity:

```bash
ping <slurmctld-host>
```

Check Slurm ports:

```bash
ss -lntp
```

---

## Recovery

After correcting the root cause:

```bash
systemctl restart slurmd
```

Then:

```bash
systemctl status slurmd
```

Check controller:

```bash
scontrol show node <node>
```

Do not return a node to production until its actual health has been validated.

---

# 17.16 Node Not Registering

Typical symptoms:

```text
Node DOWN
Node UNKNOWN
Not responding
Registration failure
```

Check:

```bash
scontrol show node <node>
```

On compute node:

```bash
systemctl status slurmd
journalctl -u slurmd
```

Check hostname:

```bash
hostname
hostname -f
```

Check DNS:

```bash
getent hosts <controller>
getent hosts <node>
```

Check time synchronization:

```bash
timedatectl
```

Check network:

```bash
ip -br addr
ip route
```

Potential causes:

```text
Wrong hostname
DNS failure
Network failure
Firewall
slurmd failure
Configuration mismatch
Time synchronization issue
Controller connectivity
```

---

# 17.17 Job Pending

Start with:

```bash
squeue -j <JOBID>
```

Show reason:

```bash
squeue -j <JOBID> \
-o "%.18i %.9T %.30R"
```

Detailed:

```bash
scontrol show job <JOBID>
```

Common reasons:

```text
Priority
Resources
Dependency
QOS
Partition
Association
Node state
```

---

## Example

```text
JOBID  STATE    NODELIST(REASON)
12345  PENDING  (Resources)
```

Meaning:

> The scheduler currently cannot satisfy the requested resources.

Check:

```bash
sinfo
```

Then:

```bash
scontrol show partition <partition>
```

Check requested resources:

```bash
scontrol show job 12345
```

Look for:

```text
ReqTRES
MinMemory
NumNodes
CPUs
Gres
Partition
QOS
```

---

# 17.18 Job Failed

Check:

```bash
sacct -j <JOBID>
```

Detailed:

```bash
sacct -j <JOBID> \
--format=JobID,State,ExitCode,Elapsed,MaxRSS,NodeList
```

Check job output:

```bash
cat slurm-<JOBID>.out
```

Check job script:

```bash
cat job.sh
```

Check node:

```bash
scontrol show node <node>
```

Possible causes:

```text
Application error
Wrong environment
Missing library
Permission problem
Filesystem failure
GPU failure
Memory exhaustion
Network failure
Node failure
```

---

# 17.19 Job Out of Memory

Slurm may report:

```text
OUT_OF_MEMORY
```

Check:

```bash
sacct -j <JOBID> \
--format=JobID,State,MaxRSS,ReqMem,Elapsed
```

Check node logs:

```bash
journalctl -k | grep -i oom
```

Check memory:

```bash
free -h
```

Potential causes:

```text
Application memory leak
Insufficient --mem
Unexpected workload size
Node memory pressure
```

For a job requesting memory:

```bash
#SBATCH --mem=64G
```

Do not simply increase memory without understanding why the job exceeded its allocation.

---

# 17.20 Job Timeout

Typical state:

```text
TIMEOUT
```

Check:

```bash
sacct -j <JOBID> \
--format=JobID,State,Elapsed,Timelimit
```

Inspect:

```bash
scontrol show job <JOBID>
```

Possible causes:

```text
Underestimated runtime
Application inefficiency
Filesystem slowdown
Network/RDMA problem
GPU performance issue
CPU contention
```

For debugging, compare:

```text
Requested Time
      vs
Actual Runtime
```

---

# 17.21 Partition Problems

Check:

```bash
scontrol show partition
```

or:

```bash
sinfo
```

Look for:

```text
State
Nodes
Default
MaxTime
AllowAccounts
AllowQos
```

Typical issue:

```text
Job requests GPU partition
but GPU nodes are unavailable.
```

Check:

```bash
sinfo -p <partition> -N -l
```

Then inspect individual nodes:

```bash
scontrol show node <node>
```

---

# 17.22 QoS and Account Problems

A job may remain pending because of:

```text
Association
Account
QoS
Limits
Priority
```

Check job:

```bash
scontrol show job <JOBID>
```

Check associations:

```bash
sacctmgr show assoc
```

Check QoS:

```bash
sacctmgr show qos
```

Check user/account:

```bash
sacctmgr show user <username> withassoc
```

Example diagnostic chain:

```text
Job Pending
    ↓
Reason = QOS
    ↓
scontrol show job
    ↓
sacctmgr show qos
    ↓
Check limits
    ↓
Check user association
```

Avoid modifying accounting configuration during an incident unless the evidence clearly indicates an accounting/configuration problem.

---

# 17.23 Slurm Controller Troubleshooting

On the controller:

```bash
systemctl status slurmctld
```

Logs:

```bash
journalctl -u slurmctld -n 100
```

Check cluster:

```bash
sinfo
```

Configuration:

```bash
scontrol show config
```

Check controller connectivity from compute node:

```bash
getent hosts <slurmctld>
```

```bash
ss -ntp
```

Potential causes:

```text
slurmctld stopped
Configuration error
DNS problem
Network problem
Firewall
Database/accounting issue
Resource exhaustion
```

---

# 17.24 Production Recovery Workflow

For a failed compute node:

```text
             Node Failure
                  │
                  ▼
          Check Slurm State
                  │
                  ▼
        scontrol show node
                  │
                  ▼
          Check Node Access
                  │
          ┌───────┴───────┐
          ▼               ▼
       SSH works       SSH fails
          │               │
          ▼               ▼
     Check slurmd      Network/OS
          │
          ▼
    Check Linux health
          │
    ┌─────┼──────┬──────┐
    ▼     ▼      ▼      ▼
   GPU    IB    Lustre  Disk
    │     │      │      │
    └─────┴──────┴──────┘
              │
              ▼
             Fix
              │
              ▼
           Validate
              │
              ▼
        Return to Slurm
```

After fixing:

```bash
scontrol update NodeName=<node> State=RESUME
```

Then verify:

```bash
sinfo -N -l
```

**Only use `RESUME` after confirming the underlying fault is resolved.**

---

# 17.25 Quick Revision

### Node problem

```bash
sinfo -R
scontrol show node <node>
systemctl status slurmd
journalctl -u slurmd
```

### Job problem

```bash
squeue -j <jobid>
scontrol show job <jobid>
sacct -j <jobid>
```

### Pending job

```bash
squeue -j <jobid> -o "%.18i %.9T %.30R"
```

### Account/QoS

```bash
sacctmgr show assoc
sacctmgr show qos
```

### Controller

```bash
systemctl status slurmctld
journalctl -u slurmctld
```

### Recovery

```bash
scontrol update NodeName=<node> State=RESUME
```

> **Production rule:** A Slurm node marked `DOWN` or `DRAIN` is a **symptom**, not necessarily the root cause. Always investigate Linux, networking, GPU, InfiniBand, storage and configuration before returning the node to service.

# End of Chapter 17 – Part 2
