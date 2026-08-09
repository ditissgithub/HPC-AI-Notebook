## Part 2 – HPC, Slurm, InfiniBand, GPU & Lustre Commands

> **Notebook focus:** High-value commands used daily while operating and troubleshooting HPC compute, GPU, network and storage infrastructure.

* [16.12 Slurm Commands](#1612-slurm-commands)
* [16.13 Slurm Node Health](#1613-slurm-node-health)
* [16.14 Slurm Job Troubleshooting](#1614-slurm-job-troubleshooting)
* [16.15 InfiniBand Commands](#1615-infiniband-commands)
* [16.16 RDMA Commands](#1616-rdma-commands)
* [16.17 NVIDIA GPU Commands](#1617-nvidia-gpu-commands)
* [16.18 CUDA Checks](#1618-cuda-checks)
* [16.19 Lustre Commands](#1619-lustre-commands)
* [16.20 xCAT Commands](#1620-xcat-commands)
* [16.21 LDAP/SSSD Commands](#1621-ldapsssd-commands)
* [16.22 Quick HPC Health Check](#1622-quick-hpc-health-check)
* [16.23 Quick Revision](#1623-quick-revision)


# 16.12 Slurm Commands

## Cluster Status

```bash
sinfo
```

Detailed:

```bash
sinfo -Nel
```

Node information:

```bash
scontrol show nodes
```

Cluster configuration:

```bash
scontrol show config
```

---

## Job Queue

```bash
squeue
```

User jobs:

```bash
squeue -u $USER
```

Detailed job:

```bash
scontrol show job <JOBID>
```

---

## Submit Job

```bash
sbatch job.sh
```

Interactive allocation:

```bash
salloc -N 1 -n 1
```

Run directly:

```bash
srun hostname
```

---

## Cancel Job

```bash
scancel <JOBID>
```

Cancel all jobs for a user:

```bash
scancel -u $USER
```

Use carefully on production clusters.

---

# 16.13 Slurm Node Health

Check node:

```bash
scontrol show node <NODE>
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
sinfo -N -l
```

Identify drained nodes:

```bash
sinfo -R
```

Typical workflow:

```text
Node DOWN/DRAIN
      ↓
scontrol show node
      ↓
Check reason
      ↓
Check Linux
      ↓
Check Slurm
      ↓
Check GPU/IB/Lustre
      ↓
Fix
      ↓
Return node to service
```

---

# 16.14 Slurm Job Troubleshooting

Check job:

```bash
scontrol show job <JOBID>
```

Check completed job:

```bash
sacct -j <JOBID>
```

Useful fields:

```bash
sacct -j <JOBID> \
--format=JobID,JobName,State,ExitCode,Elapsed,MaxRSS,NodeList
```

Common states:

```text
PENDING
RUNNING
COMPLETED
FAILED
CANCELLED
TIMEOUT
OUT_OF_MEMORY
```

Check why a job is pending:

```bash
squeue -j <JOBID> -o "%.18i %.9T %.20R"
```

Typical reasons:

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

# 16.15 InfiniBand Commands

## Device Status

```bash
ibstat
```

Detailed:

```bash
ibstatus
```

## HCA Information

```bash
ibv_devinfo
```

## Devices

```bash
ibdev2netdev
```

Useful mapping:

```text
HCA
 ↓
Port
 ↓
IB interface
```

## Link Information

```bash
ibstat
```

Check for:

```text
State: Active
Physical state: LinkUp
```

---

# 16.16 RDMA Commands

List RDMA devices:

```bash
rdma link
```

Show RDMA resources:

```bash
rdma resource show
```

Check RDMA-capable devices:

```bash
ibv_devices
```

Basic RDMA test tools may include:

```bash
ib_write_bw
ib_read_bw
ib_send_bw
```

Example:

```bash
ib_write_bw
```

Use appropriate server/client invocation according to the installed perftest version.

---

## InfiniBand Troubleshooting Flow

```text
ibstat
   ↓
ibdev2netdev
   ↓
rdma link
   ↓
ip link
   ↓
Check subnet manager
   ↓
Check cables/switch
```

If the port is not `ACTIVE`, application-level RDMA testing is premature.

---

# 16.17 NVIDIA GPU Commands

## GPU Status

```bash
nvidia-smi
```

Continuous monitoring:

```bash
watch -n 1 nvidia-smi
```

## GPU Processes

```bash
nvidia-smi pmon -s um
```

## GPU Information

```bash
nvidia-smi -q
```

Specific GPU:

```bash
nvidia-smi -i 0
```

## Driver Version

```bash
nvidia-smi
```

or:

```bash
cat /proc/driver/nvidia/version
```

## GPU Utilization

```bash
nvidia-smi \
--query-gpu=index,name,temperature.gpu,utilization.gpu,memory.used,memory.total \
--format=csv
```

---

# 16.18 CUDA Checks

Check CUDA compiler:

```bash
nvcc --version
```

Check NVIDIA driver:

```bash
nvidia-smi
```

Remember:

```text
NVIDIA Driver
      │
      ├── Kernel module
      │
      └── CUDA compatibility
```

`nvcc` availability is not required for every CUDA application environment.

Check libraries:

```bash
ldconfig -p | grep -i cuda
```

---

## GPU Troubleshooting

### GPU Not Detected

```bash
lspci | grep -i nvidia
```

Then:

```bash
nvidia-smi
```

Check driver module:

```bash
lsmod | grep nvidia
```

Kernel messages:

```bash
dmesg | grep -i nvidia
```

Flow:

```text
PCIe
 ↓
NVIDIA device
 ↓
Kernel driver
 ↓
nvidia-smi
 ↓
CUDA application
```

---

# 16.19 Lustre Commands

## Mount

```bash
mount | grep lustre
```

Filesystem:

```bash
df -hT | grep lustre
```

## Capacity

```bash
lfs df -h
```

## File Layout

```bash
lfs getstripe <file>
```

Directory layout:

```bash
lfs getstripe <directory>
```

## Set Stripe

Example:

```bash
lfs setstripe -c 4 <directory>
```

Example with stripe size:

```bash
lfs setstripe -S 4M -c 4 <directory>
```

Use workload-specific values rather than assuming that higher stripe counts are always better.

---

## Lustre Quota

Check user quota:

```bash
lfs quota -h -u user01 /home
```

```bash
lfs quota -h -u user01 /scratch
```

Check filesystem:

```bash
lfs df -h
```

---

# 16.20 xCAT Commands

Check node definition:

```bash
lsdef <node>
```

List nodes:

```bash
lsdef -t node
```

Check node status:

```bash
nodestat <node>
```

Run command through xCAT:

```bash
xdsh <node> hostname
```

Run on a group:

```bash
xdsh compute "uptime"
```

Copy files:

```bash
xcp file.txt compute:/tmp/
```

Check xCAT tables:

```bash
tabdump
```

Check a specific table:

```bash
tabdump nodetype
```

---

## xCAT Troubleshooting Mental Model

```text
Node Definition
      ↓
Network/DHCP
      ↓
Boot
      ↓
OS Image
      ↓
Post-install
      ↓
Node Configuration
      ↓
Slurm Registration
```

---

# 16.21 LDAP/SSSD Commands

Check user identity:

```bash
id user01
```

Check NSS:

```bash
getent passwd user01
```

Check group:

```bash
getent group hpcusers
```

Check SSSD:

```bash
systemctl status sssd
```

SSSD logs:

```bash
journalctl -u sssd
```

LDAP connectivity:

```bash
ldapsearch -x \
-H ldap://ldap01 \
-b "dc=nsm,dc=in" \
"(uid=user01)"
```

Check LDAP client configuration:

```bash
sssctl config-check
```

Depending on the SSSD version, additional diagnostics can be obtained with:

```bash
sssctl domain-list
sssctl domain-status <domain>
```

---

# 16.22 Quick HPC Health Check

For a GPU compute node:

```bash
hostname
uptime
free -h
df -hT
ip -br addr
systemctl --failed
```

### Slurm

```bash
systemctl status slurmd
```

### InfiniBand

```bash
ibstat
ibdev2netdev
```

### GPU

```bash
nvidia-smi
```

### Lustre

```bash
mount | grep lustre
lfs df -h
```

### LDAP

```bash
id user01
getent passwd user01
```

---

# 16.23 Quick Revision

## Slurm

```bash
sinfo
squeue
scontrol show node <node>
scontrol show job <jobid>
sacct -j <jobid>
scancel <jobid>
```

## InfiniBand

```bash
ibstat
ibstatus
ibdev2netdev
ibv_devinfo
rdma link
```

## GPU

```bash
nvidia-smi
nvidia-smi -q
nvcc --version
lspci | grep -i nvidia
```

## Lustre

```bash
lfs df -h
lfs getstripe <path>
lfs quota -h -u <user> <filesystem>
```

## xCAT

```bash
lsdef
nodestat
xdsh
xcp
tabdump
```

## LDAP

```bash
id <user>
getent passwd <user>
sssctl domain-status <domain>
ldapsearch
```

---

# HPC Engineer Daily Command Map

```text
                    HPC NODE
                       │
       ┌───────────────┼────────────────┐
       ▼               ▼                ▼
     Linux           Slurm           Hardware
       │               │                │
   systemctl         sinfo          ┌───┴────┐
   journalctl        squeue         ▼        ▼
   ps/top            sacct         GPU       IB
   df/iostat         scontrol      │         │
                                   nvidia-smi ibstat
                                      │
                                      ▼
                                   Lustre
                                      │
                                   lfs df
                                      │
                                      ▼
                                   LDAP
                                      │
                                  id/getent
```

> **HPC Engineer rule:** Do not memorize commands in isolation. Associate each command with the layer it diagnoses: **OS → Scheduler → Network → GPU → Storage → Authentication**.

# End of Chapter 16 – Part 2
