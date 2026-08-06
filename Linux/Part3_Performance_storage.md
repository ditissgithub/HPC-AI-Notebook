# Part 3 – Performance & Storage

- [2.20 Linux Memory Management](#220-linux-memory-management)
- [2.21 CPU Scheduling & Performance](#221-cpu-scheduling--performance)
- [2.22 Disk I/O](#222-disk-io)
- [2.23 Linux Filesystems](#223-linux-filesystems)
- [2.24 Logical Volume Manager (LVM)](#224-logical-volume-manager-lvm)
- [2.25 RAID Fundamentals](#225-raid-fundamentals)
- [2.26 Swap Memory](#226-swap-memory)
- [2.27 Performance Monitoring Tools](#227-performance-monitoring-tools)
- [2.28 Performance Troubleshooting Workflow](#228-performance-troubleshooting-workflow)
- [Production Insight](#production-insight)
- [Key Takeaways](#key-takeaways)

---

# 2.20 Linux Memory Management

## Overview

Memory is one of the most critical resources in an HPC system.

Every running application requires memory to store:

- Program code
- Variables
- Buffers
- Shared libraries
- Temporary data

Unlike desktop systems, HPC applications may require hundreds of gigabytes or even terabytes of memory.

---

## Linux Memory Architecture

```
                CPU

                 │

                 ▼

          CPU Cache (L1/L2/L3)

                 │

                 ▼

           Physical Memory (RAM)

                 │

                 ▼

          Swap (Disk Based)
```

---

## Virtual Memory

Linux provides **Virtual Memory**.

Every process believes it owns its own memory space.

```
Application

↓

Virtual Address

↓

Kernel

↓

Physical Memory
```

Benefits:

- Memory isolation
- Security
- Efficient allocation
- Large address space

---

## Memory Layout

```
+----------------------+

| Kernel Space         |

+----------------------+

| Shared Libraries     |

+----------------------+

| Heap                 |

+----------------------+

| Stack                |

+----------------------+

| Program Code         |

+----------------------+
```

---

## Important Commands

View memory

```bash
free -h
```

Detailed memory

```bash
cat /proc/meminfo
```

Virtual memory statistics

```bash
vmstat
```

Interactive

```bash
top
```

---

## Memory Pressure

Symptoms

- Slow applications
- High swap usage
- OOM Killer events
- Long response times

---

## Out of Memory (OOM)

When Linux runs out of memory:

```
Memory Full

↓

Kernel

↓

OOM Killer

↓

Terminates Process
```

View OOM events

```bash
dmesg | grep -i oom
```

or

```bash
journalctl | grep -i oom
```

---

# 2.21 CPU Scheduling & Performance

Linux distributes processes across available CPU cores.

```
Processes

↓

Scheduler

↓

CPU Cores
```

---

## View CPU Information

```bash
lscpu
```

Example output

```
Architecture

CPU(s)

Sockets

Cores

Threads
```

---

## CPU Usage

Overall utilization

```bash
top
```

Per-core usage

```bash
mpstat -P ALL 1
```

or

```bash
sar -P ALL 1
```

---

## CPU Load

View load average

```bash
uptime
```

Example

```
load average: 1.52 2.14 2.48
```

Interpretation

- 1-minute
- 5-minute
- 15-minute

---

## CPU Bottlenecks

Common causes

- Runaway processes
- Too many runnable tasks
- CPU affinity issues
- Interrupt storms
- NUMA imbalance

---

## Useful Commands

```bash
top

htop

pidstat

mpstat

sar

uptime

lscpu
```

---

# 2.22 Disk I/O

Storage performance directly impacts HPC applications.

Large datasets require high throughput.

```
Application

↓

Filesystem

↓

Block Layer

↓

Storage Device
```

---

## Disk Statistics

Overall I/O

```bash
iostat
```

Extended statistics

```bash
iostat -xz 1
```

Disk usage

```bash
df -h
```

Directory usage

```bash
du -sh
```

Block devices

```bash
lsblk
```

---

## Important Metrics

| Metric | Meaning |
|----------|----------|
| %util | Disk utilization |
| await | Average I/O wait |
| svctm | Service time |
| r/s | Read operations |
| w/s | Write operations |

---

## Detect I/O Bottlenecks

Useful commands

```bash
iostat -xz

iotop

vmstat

dstat
```

---

## I/O Wait

High I/O wait indicates the CPU is waiting for storage.

View

```bash
vmstat 1
```

Check

```
wa
```

column.

---

# 2.23 Linux Filesystems

A filesystem organizes data stored on disks.

Common filesystems

- XFS
- EXT4
- Btrfs
- Lustre
- NFS

---

## XFS

Advantages

- Excellent scalability
- Large files
- Parallel workloads
- Enterprise support

Used extensively in HPC environments.

---

## EXT4

Advantages

- Stable
- Mature
- Reliable

Suitable for general-purpose Linux systems.

---

## View Filesystems

```bash
df -Th
```

View mounted devices

```bash
findmnt
```

Filesystem information

```bash
blkid
```

---

## Check Filesystem

EXT4

```bash
fsck.ext4
```

XFS

```bash
xfs_repair
```

---

# 2.24 Logical Volume Manager (LVM)

LVM provides flexible disk management.

Traditional layout

```
Disk

↓

Partition

↓

Filesystem
```

LVM

```
Disk

↓

Physical Volume

↓

Volume Group

↓

Logical Volume

↓

Filesystem
```

---

## Components

### Physical Volume

```
pvcreate
```

---

### Volume Group

```
vgcreate
```

---

### Logical Volume

```
lvcreate
```

---

## Useful Commands

Display PV

```bash
pvs
```

Display VG

```bash
vgs
```

Display LV

```bash
lvs
```

Complete information

```bash
lvdisplay
```

---

## Benefits

- Online expansion
- Flexible storage
- Easier administration
- Snapshot capability

---

# 2.25 RAID Fundamentals

RAID combines multiple disks.

Objectives

- Performance
- Redundancy
- Capacity

---

## RAID Levels

| RAID | Purpose |
|--------|----------|
| RAID 0 | Performance |
| RAID 1 | Mirroring |
| RAID 5 | Parity |
| RAID 6 | Dual Parity |
| RAID 10 | Performance + Redundancy |

---

## RAID Overview

```
Disks

↓

RAID Controller

↓

Logical Device

↓

Filesystem
```

---

## View RAID

Software RAID

```bash
cat /proc/mdstat
```

Detailed information

```bash
mdadm --detail /dev/md0
```

---

# 2.26 Swap Memory

Swap extends RAM onto disk.

```
RAM Full

↓

Swap

↓

Disk
```

---

## Why Swap Exists

Provides

- Temporary memory
- OOM prevention
- Memory balancing

Swap is much slower than RAM.

---

## View Swap

```bash
swapon --show
```

Memory summary

```bash
free -h
```

Disable

```bash
swapoff -a
```

Enable

```bash
swapon -a
```

---

## Swappiness

Current value

```bash
cat /proc/sys/vm/swappiness
```

Temporary

```bash
sysctl vm.swappiness=10
```

---

# 2.27 Performance Monitoring Tools

Production administrators constantly monitor system health.

---

## CPU

```bash
top

htop

mpstat

pidstat
```

---

## Memory

```bash
free

vmstat

sar

numastat
```

---

## Storage

```bash
iostat

iotop

df

du

lsblk
```

---

## Network

```bash
ss

ip

sar -n DEV

ethtool
```

---

## Processes

```bash
ps

pstree

pidstat

pgrep
```

---

## Logs

```bash
journalctl

dmesg
```

---

## Hardware

```bash
lscpu

lspci

lsblk

dmidecode
```

---

# 2.28 Performance Troubleshooting Workflow

A systematic troubleshooting process helps identify bottlenecks quickly.

```
Application Slow

↓

CPU

↓

Memory

↓

Disk

↓

Network

↓

Logs

↓

Hardware

↓

Root Cause
```

---

## Step 1 — CPU

Check

```bash
top

mpstat

uptime
```

---

## Step 2 — Memory

Check

```bash
free

vmstat

cat /proc/meminfo
```

---

## Step 3 — Storage

Check

```bash
iostat

iotop

df
```

---

## Step 4 — Network

Check

```bash
ip addr

ip route

ss

ping
```

---

## Step 5 — Logs

```bash
journalctl

dmesg
```

---

## Step 6 — Hardware

```bash
lscpu

lsblk

lspci
```

---

## Example Production Scenario

### Symptom

A Slurm job is running significantly slower than expected.

### Investigation

1. Check CPU utilization.

```bash
mpstat -P ALL 1
```

2. Verify memory usage.

```bash
free -h
```

3. Check for swap activity.

```bash
vmstat 1
```

4. Inspect disk performance.

```bash
iostat -xz 1
```

5. Review kernel messages.

```bash
dmesg
```

### Possible Root Causes

- CPU oversubscription
- Memory exhaustion
- Excessive swapping
- Storage bottleneck
- Hardware failure

---

# Production Insight

Performance issues in HPC environments rarely originate from a single component. A slow application may be caused by CPU contention, insufficient memory, heavy disk I/O, NUMA imbalance, network congestion, or filesystem latency. Effective administrators gather evidence from multiple subsystems before identifying the root cause. Avoid making configuration changes until you have verified the source of the performance problem.

---

# Key Takeaways

- Linux uses virtual memory to efficiently manage RAM.
- CPU scheduling distributes work across available cores.
- High I/O wait often indicates storage bottlenecks.
- XFS is the preferred filesystem for many enterprise and HPC deployments.
- LVM provides flexible and scalable storage management.
- RAID improves performance, redundancy, or both depending on the level used.
- Swap should supplement RAM, not replace it.
- Performance troubleshooting should follow a structured workflow rather than relying on guesswork.
- Mastering Linux performance tools is essential for diagnosing production HPC systems.

---

## Next Part

**Part 4 – Security & Administration**

Topics covered:

- User & Group Management
- File Permissions
- ACL
- SELinux
- SSH
- Package Management
- Log Management
- Cron & systemd Timers
