## Part 4 – Command Cheat Sheet & HPC Diagnostic Patterns

> **Notebook focus:** A compact daily-reference section for Linux, Slurm, networking, InfiniBand, GPU, Lustre, xCAT and LDAP operations.

* [16.35 Linux Quick Reference](#1635-linux-quick-reference)
* [16.36 Network Quick Reference](#1636-network-quick-reference)
* [16.37 Slurm Quick Reference](#1637-slurm-quick-reference)
* [16.38 InfiniBand Quick Reference](#1638-infiniband-quick-reference)
* [16.39 NVIDIA GPU Quick Reference](#1639-nvidia-gpu-quick-reference)
* [16.40 Lustre Quick Reference](#1640-lustre-quick-reference)
* [16.41 xCAT Quick Reference](#1641-xcat-quick-reference)
* [16.42 LDAP/SSSD Quick Reference](#1642-ldapsssd-quick-reference)
* [16.43 Common Diagnostic Patterns](#1643-common-diagnostic-patterns)
* [16.44 Final Daily Checklist](#1644-final-daily-checklist)


# 16.35 Linux Quick Reference

| Task            | Command                      |
| --------------- | ---------------------------- |
| OS              | `cat /etc/os-release`        |
| Kernel          | `uname -r`                   |
| CPU             | `lscpu`                      |
| Memory          | `free -h`                    |
| Load            | `uptime`                     |
| Processes       | `ps -ef`                     |
| Disk            | `df -hT`                     |
| Directory size  | `du -sh <path>`              |
| Block devices   | `lsblk`                      |
| I/O             | `iostat -xz 1`               |
| Services        | `systemctl status <service>` |
| Failed services | `systemctl --failed`         |
| Logs            | `journalctl -u <service>`    |
| Find files      | `find`                       |
| Search text     | `grep`                       |

---

# 16.36 Network Quick Reference

```bash id="g4j6nq"
ip -br addr
```

```bash id="s5xqjv"
ip link
ip route
```

```bash id="4ypo6x"
ip -s link
```

```bash id="d5cxk2"
ss -lntup
```

```bash id="m5g4cc"
ping <host>
```

```bash id="0w2q9v"
getent hosts <hostname>
```

```bash id="yp7d4v"
dig <hostname>
```

### Diagnostic order

```text id="j09b7u"
Interface
   ↓
Link
   ↓
IP
   ↓
Route
   ↓
DNS
   ↓
Application
```

---

# 16.37 Slurm Quick Reference

### Cluster

```bash id="y4k0qg"
sinfo
sinfo -Nel
scontrol show config
```

### Nodes

```bash id="1x9zv1"
sinfo -N -l
sinfo -R
scontrol show node <node>
```

### Jobs

```bash id="q31m3x"
squeue
squeue -u $USER
scontrol show job <jobid>
sacct -j <jobid>
```

### Job execution

```bash id="k5skca"
sbatch job.sh
srun hostname
salloc -N 1
```

### Job control

```bash id="90s7j0"
scancel <jobid>
```

### Diagnostic pattern

```text id="9z4x5q"
Job Pending
    ↓
squeue
    ↓
scontrol show job
    ↓
Check Reason
    ↓
Partition / QoS / Account / Resources
```

---

# 16.38 InfiniBand Quick Reference

```bash id="0q3u4x"
ibstat
```

```bash id="p2a5t6"
ibstatus
```

```bash id="5h2w6b"
ibdev2netdev
```

```bash id="6kg1v5"
ibv_devinfo
ibv_devices
```

```bash id="0t2lqu"
rdma link
rdma resource show
```

### Diagnostic pattern

```text id="g8r20v"
HCA
 ↓
Port
 ↓
Link
 ↓
Subnet Manager
 ↓
IPoIB/RDMA
 ↓
Application
```

If the port is not `ACTIVE`, investigate the fabric before testing the application.

---

# 16.39 NVIDIA GPU Quick Reference

### Status

```bash id="wq8y01"
nvidia-smi
```

### Detailed

```bash id="72xjcl"
nvidia-smi -q
```

### GPU monitoring

```bash id="yd3a1c"
nvidia-smi pmon -s um
```

### PCIe detection

```bash id="h8quc6"
lspci | grep -i nvidia
```

### Driver module

```bash id="0b4m51"
lsmod | grep nvidia
```

### CUDA

```bash id="v9i5y4"
nvcc --version
```

### Diagnostic pattern

```text id="x2q4hv"
PCIe
 ↓
GPU detected
 ↓
NVIDIA driver
 ↓
nvidia-smi
 ↓
CUDA/runtime
 ↓
Application
```

---

# 16.40 Lustre Quick Reference

### Mount

```bash id="v4a1tp"
mount | grep lustre
```

### Capacity

```bash id="e9q4bq"
lfs df -h
```

### Stripe

```bash id="y9y6pw"
lfs getstripe <path>
```

### Set stripe

```bash id="j7y8b3"
lfs setstripe -c 4 <directory>
```

### User quota

```bash id="oqg8n4"
lfs quota -h -u user01 /home
```

### Logs

```bash id="j6d6xw"
dmesg | grep -i lustre
```

### Diagnostic pattern

```text id="3l1z5u"
Application
 ↓
Client
 ↓
Network
 ↓
MDT / OST
 ↓
Backend Storage
```

---

# 16.41 xCAT Quick Reference

```bash id="4m5j0z"
lsdef <node>
```

```bash id="m2q2pv"
nodestat <node>
```

```bash id="2gqf4x"
xdsh <node> hostname
```

```bash id="d4c2z9"
xcp file <node>:/path/
```

```bash id="1xw6w9"
tabdump
```

### Provisioning mental model

```text id="v5f6tq"
Node Definition
      ↓
Network
      ↓
Discovery
      ↓
Boot
      ↓
OS Image
      ↓
Post-install
      ↓
Configured Node
```

---

# 16.42 LDAP/SSSD Quick Reference

### Identity

```bash id="m4f5cy"
id user01
```

```bash id="p5y9qf"
getent passwd user01
```

```bash id="q4m5y5"
getent group hpcusers
```

### SSSD

```bash id="4h8e9p"
systemctl status sssd
```

```bash id="5r7c4x"
sssctl domain-list
```

```bash id="v6x4cb"
sssctl domain-status <domain>
```

### LDAP

```bash id="1f8y6r"
ldapsearch -x \
-H ldap://ldap01 \
-b "dc=nsm,dc=in" \
"(uid=user01)"
```

### Diagnostic pattern

```text id="4g5o0s"
User
 ↓
NSS
 ↓
SSSD
 ↓
LDAP
 ↓
Directory
```

---

# 16.43 Common Diagnostic Patterns

## Node Not Joining Slurm

```text id="4t3g5c"
slurmd
  ↓
systemctl status slurmd
  ↓
journalctl -u slurmd
  ↓
scontrol show node
  ↓
Check hostname/configuration
  ↓
Check network
```

---

## GPU Node Problem

```text id="q0qgq8"
lspci
  ↓
lsmod
  ↓
nvidia-smi
  ↓
dmesg
  ↓
CUDA/runtime
  ↓
Slurm GRES
```

---

## Lustre Problem

```text id="0i4g2p"
mount
  ↓
lfs df
  ↓
lfs quota
  ↓
lfs getstripe
  ↓
Network
  ↓
Client logs
```

---

## Authentication Problem

```text id="6u2m5n"
id user
  ↓
getent passwd
  ↓
systemctl status sssd
  ↓
sssctl domain-status
  ↓
LDAP query
  ↓
PAM / SSH logs
```

---

## Network Problem

```text id="3y5j9v"
ip link
  ↓
ip addr
  ↓
ip route
  ↓
ping
  ↓
DNS
  ↓
ss
  ↓
Application
```

---

# 16.44 Final Daily Checklist

Before declaring a compute node healthy:

```text id="2k8h1y"
☐ Hostname correct
☐ OS/kernel healthy
☐ CPU available
☐ Memory healthy
☐ Filesystems mounted
☐ Disk usage acceptable
☐ Network interfaces UP
☐ Routes correct
☐ InfiniBand ACTIVE
☐ GPU detected
☐ NVIDIA driver healthy
☐ Lustre accessible
☐ LDAP lookup working
☐ SSSD healthy
☐ slurmd healthy
☐ Node registered in Slurm
☐ No unexpected errors in logs
```

A compact command sequence:

```bash id="w9k3a1"
hostname
uptime
free -h
df -hT
ip -br addr
systemctl --failed
systemctl is-active slurmd
ibstat
nvidia-smi
mount | grep lustre
id user01
```

---

# Daily Operations Principle

```text id="7q5v9h"
                Observe
                   ↓
              Collect Data
                   ↓
               Compare
                   ↓
              Find Layer
                   ↓
               Diagnose
                   ↓
                Fix
                   ↓
              Validate
```

> **Final notebook rule:** Use this chapter as a **daily operational reference**, not as a replacement for detailed component documentation. For production incidents, collect evidence before making changes.

# End of Chapter 16 – Part 4
