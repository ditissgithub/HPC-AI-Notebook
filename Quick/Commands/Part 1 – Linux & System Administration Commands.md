## Part 1 – Linux & System Administration Commands

* [16.1 System Information](#161-system-information)
* [16.2 CPU and Memory](#162-cpu-and-memory)
* [16.3 Processes](#163-processes)
* [16.4 Disk and Filesystem](#164-disk-and-filesystem)
* [16.5 Files and Directories](#165-files-and-directories)
* [16.6 Services and Systemd](#166-services-and-systemd)
* [16.7 Users and Groups](#167-users-and-groups)
* [16.8 Logs](#168-logs)
* [16.9 Network Basics](#169-network-basics)
* [16.10 HPC Daily Health Check](#1610-hpc-daily-health-check)
* [16.11 Quick Revision](#1611-quick-revision)

# 16.1 System Information

### OS Information

```bash
cat /etc/os-release
```

### Kernel

```bash
uname -r
```

```bash
uname -a
```

### Hostname

```bash
hostname
hostnamectl
```

### CPU Architecture

```bash
uname -m
```

### Uptime

```bash
uptime
```

### System Load

```bash
cat /proc/loadavg
```

---

# 16.2 CPU and Memory

## CPU

```bash
lscpu
```

Quick check:

```bash
nproc
```

CPU information:

```bash
grep -E 'model name|processor' /proc/cpuinfo
```

## Memory

```bash
free -h
```

Detailed:

```bash
cat /proc/meminfo
```

## NUMA

Important on large HPC servers:

```bash
numactl --hardware
```

Check NUMA topology:

```bash
lscpu | grep -i numa
```

---

# 16.3 Processes

## Process List

```bash
ps aux
```

Better for detailed investigation:

```bash
ps -ef
```

## Interactive Monitoring

```bash
top
```

If installed:

```bash
htop
```

## Find a Process

```bash
pgrep -a <process>
```

Example:

```bash
pgrep -a slurmd
```

## Process Details

```bash
ps -fp <PID>
```

## Kill Process

```bash
kill <PID>
```

Force only when necessary:

```bash
kill -9 <PID>
```

## Process State

```bash
ps -eo pid,ppid,state,comm
```

Important states:

```text
R → Running
S → Sleeping
D → Uninterruptible sleep
Z → Zombie
T → Stopped
```

---

# 16.4 Disk and Filesystem

## Disk Usage

```bash
df -h
```

Filesystem type:

```bash
df -hT
```

## Directory Usage

```bash
du -sh /path
```

Top-level usage:

```bash
du -xh --max-depth=1 /path
```

## Block Devices

```bash
lsblk
```

Detailed:

```bash
lsblk -f
```

## Mounted Filesystems

```bash
mount
```

or:

```bash
findmnt
```

## Disk I/O

```bash
iostat -xz 1
```

Useful for identifying:

```text
High utilization
High latency
Low throughput
Busy devices
```

---

# 16.5 Files and Directories

## List

```bash
ls -lah
```

## Find Files

```bash
find /path -name "*.log"
```

Find large files:

```bash
find /var -type f -size +1G
```

Find recently modified files:

```bash
find /path -type f -mtime -1
```

## File Information

```bash
stat file
```

## Permissions

```bash
ls -l file
```

Change permissions:

```bash
chmod 640 file
```

Change ownership:

```bash
chown user01:hpcusers file
```

## Search Text

```bash
grep "ERROR" application.log
```

Recursive:

```bash
grep -R "ERROR" /var/log/
```

---

# 16.6 Services and Systemd

## Service Status

```bash
systemctl status <service>
```

Example:

```bash
systemctl status slurmd
```

## Start

```bash
systemctl start <service>
```

## Stop

```bash
systemctl stop <service>
```

## Restart

```bash
systemctl restart <service>
```

## Enable at Boot

```bash
systemctl enable <service>
```

## Check Failed Services

```bash
systemctl --failed
```

## Service Logs

```bash
journalctl -u <service>
```

Follow:

```bash
journalctl -fu <service>
```

---

# 16.7 Users and Groups

## Current User

```bash
whoami
```

## User Identity

```bash
id user01
```

## Logged-in Users

```bash
who
```

```bash
w
```

## User Lookup

```bash
getent passwd user01
```

## Group Lookup

```bash
getent group hpcusers
```

These are particularly important in an LDAP-integrated HPC environment.

---

# 16.8 Logs

## System Journal

```bash
journalctl
```

Recent messages:

```bash
journalctl -n 100
```

Follow logs:

```bash
journalctl -f
```

Since boot:

```bash
journalctl -b
```

Previous boot:

```bash
journalctl -b -1
```

Kernel messages:

```bash
journalctl -k
```

Traditional kernel command:

```bash
dmesg
```

Search errors:

```bash
journalctl -p err
```

---

# 16.9 Network Basics

## Interfaces

```bash
ip addr
```

Short form:

```bash
ip -br addr
```

## Link State

```bash
ip link
```

## Routes

```bash
ip route
```

## Connectivity

```bash
ping <host>
```

## DNS

```bash
getent hosts <hostname>
```

```bash
dig <hostname>
```

## Listening Ports

```bash
ss -lntup
```

## Established Connections

```bash
ss -tn
```

## Network Statistics

```bash
ip -s link
```

---

# 16.10 HPC Daily Health Check

A simple manual health check on a compute node:

```bash
hostname
uptime
free -h
lscpu
df -hT
ip -br addr
ip route
systemctl --failed
```

Then check important HPC services:

```bash
systemctl status slurmd
```

For an InfiniBand node:

```bash
ibstat
```

For NVIDIA GPU nodes:

```bash
nvidia-smi
```

For Lustre clients:

```bash
mount | grep lustre
lfs df -h
```

For LDAP-integrated systems:

```bash
id user01
getent passwd user01
```

---

# 16.11 Quick Revision

## Linux

```bash
hostnamectl
uname -r
uptime
lscpu
free -h
df -hT
lsblk
iostat -xz 1
```

## Process

```bash
ps -ef
top
pgrep
kill
```

## Services

```bash
systemctl status
systemctl --failed
journalctl -u
```

## Network

```bash
ip -br addr
ip route
ss -lntup
ip -s link
```

## Identity

```bash
id
getent passwd
getent group
```

## HPC

```bash
sinfo
squeue
ibstat
nvidia-smi
lfs df -h
```

---

# HPC Engineer Mental Model

When a node has a problem, quickly move through the layers:

```text
Node
 │
 ├── OS / Kernel
 │
 ├── CPU / Memory
 │
 ├── Disk / Filesystem
 │
 ├── Network
 │
 ├── Services
 │
 └── HPC Components
       ├── Slurm
       ├── InfiniBand
       ├── NVIDIA GPU
       ├── Lustre
       └── LDAP
```

> **Daily operations principle:** Start with simple, high-signal commands and narrow the problem layer by layer instead of immediately changing configuration.

# End of Chapter 16 – Part 1
