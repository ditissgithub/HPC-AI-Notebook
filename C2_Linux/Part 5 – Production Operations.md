# Part 5 – Production Operations

- [2.38 Linux Networking Basics](#238-linux-networking-basics)
- [2.39 Linux Troubleshooting Methodology](#239-linux-troubleshooting-methodology)
- [2.40 Production Troubleshooting Scenarios](#240-production-troubleshooting-scenarios)
- [2.41 HPC Linux Best Practices](#241-hpc-linux-best-practices)
- [2.42 Essential Linux Commands Cheat Sheet](#242-essential-linux-commands-cheat-sheet)
- [2.43 Linux Interview Questions](#243-linux-interview-questions)
- [2.44 Chapter Summary](#244-chapter-summary)

---

# 2.38 Linux Networking Basics

Although the next chapter focuses entirely on networking, every Linux administrator must understand the basic networking tools required for day-to-day operations.

---

## Linux Network Stack

```
Application

↓

Socket

↓

TCP / UDP

↓

IP

↓

Network Interface

↓

Ethernet / InfiniBand

↓

Physical Network
```

---

## View Network Interfaces

Modern Linux uses the **ip** command.

Display interfaces

```bash
ip addr
```

or

```bash
ip a
```

Example

```text
ens1f0
ib0
lo
docker0
```

---

## View Routing Table

```bash
ip route
```

Example

```text
default via 192.168.1.1

192.168.1.0/24 dev ens1f0
```

---

## Display Interface Statistics

```bash
ip -s link
```

---

## View Listening Ports

```bash
ss -tulpn
```

Examples

```bash
ss -tunap

ss -lnt

ss -lun
```

---

## Test Connectivity

Ping

```bash
ping node01
```

Trace path

```bash
traceroute node01
```

DNS lookup

```bash
dig node01
```

or

```bash
host node01
```

---

## Network Diagnostics

Show neighbor table

```bash
ip neigh
```

ARP cache

```bash
arp -n
```

Check interface

```bash
ethtool ens1f0
```

---

## Why Networking Matters in HPC

Linux networking is used by:

- Slurm
- LDAP
- Lustre
- xCAT
- SSH
- Monitoring
- MPI

A network failure often affects multiple cluster services simultaneously.

---

# 2.39 Linux Troubleshooting Methodology

One of the most important skills of an HPC Infrastructure Engineer is systematic troubleshooting.

Never troubleshoot by guessing.

Instead, collect evidence and eliminate possibilities.

---

## The Troubleshooting Pyramid

```
            Application

                 ▲

              Service

                 ▲

            Operating System

                 ▲

            Storage / Network

                 ▲

              Hardware
```

Always start from the lowest layer that could explain the problem.

---

## Standard Workflow

```
Problem Reported

↓

Observe

↓

Collect Information

↓

Identify Symptoms

↓

Check Logs

↓

Verify Configuration

↓

Test Hypothesis

↓

Implement Fix

↓

Validate

↓

Document
```

---

## Golden Rules

- Change one thing at a time.
- Record every change.
- Verify assumptions.
- Preserve logs before rebooting.
- Reproduce the issue if possible.
- Identify the root cause, not just the symptom.

---

## Useful Diagnostic Commands

System information

```bash
hostnamectl

uname -a

cat /etc/os-release
```

Hardware

```bash
lscpu

lsblk

lspci

dmidecode
```

Kernel

```bash
dmesg

journalctl -k
```

Processes

```bash
ps -ef

top

pstree
```

Memory

```bash
free -h

vmstat

cat /proc/meminfo
```

Storage

```bash
df -h

iostat -xz

mount

findmnt
```

Network

```bash
ip a

ip route

ss -tulpn

ethtool
```

---

# 2.40 Production Troubleshooting Scenarios

The following scenarios are common in HPC production environments.

---

## Scenario 1 – High CPU Utilization

### Symptoms

- Server becomes slow.
- Load average increases.
- Users report poor performance.

### Investigation

```bash
top

htop

mpstat -P ALL 1

pidstat
```

### Possible Causes

- Infinite loops
- Runaway process
- Excessive MPI ranks
- CPU oversubscription

---

## Scenario 2 – High Memory Usage

### Symptoms

- System swapping
- Slow applications
- OOM events

### Investigation

```bash
free -h

vmstat 1

cat /proc/meminfo

dmesg | grep -i oom
```

### Possible Causes

- Memory leak
- Large datasets
- Too many concurrent jobs

---

## Scenario 3 – Disk Full

### Symptoms

Applications fail with:

```
No space left on device
```

### Investigation

```bash
df -h

du -sh /*

find / -type f -size +1G
```

### Possible Causes

- Logs
- Core dumps
- Temporary files
- User data
- Backups

---

## Scenario 4 – Filesystem Read-Only

### Symptoms

Unable to create files.

### Investigation

```bash
mount

dmesg

journalctl
```

Possible causes

- Disk failure
- Filesystem corruption
- Storage controller issues

---

## Scenario 5 – Service Not Starting

### Investigation

```bash
systemctl status SERVICE

journalctl -u SERVICE

systemctl cat SERVICE
```

Possible causes

- Invalid configuration
- Missing dependencies
- Permission issues

---

## Scenario 6 – SSH Login Failure

### Investigation

```bash
systemctl status sshd

journalctl -u sshd

ss -lnt

getenforce
```

Possible causes

- SSH daemon stopped
- Firewall
- SELinux
- Incorrect configuration
- Authentication failure

---

## Scenario 7 – High Disk I/O Wait

### Investigation

```bash
iostat -xz 1

iotop

vmstat 1
```

Possible causes

- Storage bottleneck
- Slow RAID
- Heavy filesystem activity
- Large data transfers

---

## Scenario 8 – Kernel Panic

### Investigation

```bash
journalctl -k

dmesg
```

Check

- Hardware
- Drivers
- Memory
- Filesystem
- Recent kernel updates

---

# 2.41 HPC Linux Best Practices

Linux systems in HPC environments should be managed consistently.

---

## Security

- Disable direct root SSH login.
- Use SSH keys.
- Apply least-privilege access.
- Keep SELinux enabled.
- Update systems regularly.

---

## Performance

- Monitor CPU, memory, storage, and network usage.
- Minimize unnecessary background services.
- Use XFS for large data volumes where appropriate.
- Keep swap usage low under normal operation.
- Monitor NUMA placement for large-memory applications.

---

## Administration

- Use configuration management tools (e.g., Ansible).
- Standardize server configurations.
- Maintain accurate documentation.
- Test changes in a non-production environment.
- Automate repetitive tasks.

---

## Logging

- Monitor logs daily.
- Configure log rotation.
- Investigate recurring warnings.
- Centralize logs where possible.

---

## Backup

Always back up:

- Configuration files
- User data
- Scripts
- Critical databases
- Cluster metadata

---

## Monitoring

Continuously monitor:

- CPU
- Memory
- Storage
- Network
- Services
- Hardware health

Monitoring enables proactive issue detection.

---

# 2.42 Essential Linux Commands Cheat Sheet

## System Information

```bash
hostnamectl
uname -a
uptime
date
timedatectl
```

---

## Users

```bash
who
w
id
groups
last
lastlog
```

---

## Processes

```bash
ps -ef
top
htop
pgrep
pstree
kill
pkill
```

---

## Memory

```bash
free -h
vmstat
cat /proc/meminfo
```

---

## CPU

```bash
lscpu
mpstat
pidstat
sar
```

---

## Storage

```bash
df -h
du -sh
lsblk
blkid
findmnt
mount
```

---

## Files

```bash
find
locate
stat
file
chmod
chown
ln
```

---

## Networking

```bash
ip a
ip route
ss -tulpn
ping
dig
host
ethtool
```

---

## Services

```bash
systemctl
journalctl
systemctl --failed
```

---

## Logs

```bash
journalctl
dmesg
tail -f
grep
```

---

## Packages

```bash
dnf
rpm
```

---

# 2.43 Linux Interview Questions

## Basic

1. What is Linux?
2. What is the Linux kernel?
3. Explain User Space and Kernel Space.
4. What happens during the Linux boot process?
5. What is an inode?

---

## Intermediate

1. Explain virtual memory.
2. What happens when Linux runs out of memory?
3. What is the OOM Killer?
4. Explain process states.
5. What is systemd?
6. Difference between `systemctl` and `service`?
7. Difference between XFS and EXT4?
8. Explain LVM architecture.
9. Explain RAID levels.
10. What is swap?

---

## Advanced

1. How would you troubleshoot a slow Linux server?
2. A service fails to start. What is your approach?
3. How do you identify a memory leak?
4. How do you diagnose high I/O wait?
5. What causes a process to enter D state?
6. How would you investigate a kernel panic?
7. Explain CPU affinity and its use cases.
8. How would you troubleshoot high load average?
9. How would you investigate excessive swap usage?
10. What Linux knowledge is most important for HPC administrators?

---

## Scenario-Based Questions

- Users report that Slurm jobs are slow. Which Linux subsystems would you check first?
- The root filesystem reaches 100% utilization. How would you identify the largest consumers of disk space?
- A compute node becomes unreachable after reboot. Which logs and services would you inspect?
- A process cannot be terminated with `kill -15`. What could be happening, and what would you investigate?
- SSH access fails after a configuration change. Describe your recovery process.

---

# 2.44 Chapter Summary

This chapter established the Linux administration skills required for managing HPC and AI infrastructure.

We covered:

- Linux architecture and kernel fundamentals
- Boot process and filesystem hierarchy
- Process and service management
- Memory, CPU, storage, and performance analysis
- User administration and security
- SSH, package management, and logging
- Automation with cron and systemd timers
- Troubleshooting methodology
- Production scenarios
- Essential Linux commands
- Interview preparation

These concepts form the operational foundation for every higher-level HPC technology discussed in the following chapters.

---

# Chapter Completion Checklist

You should now be able to:

- Explain Linux architecture.
- Understand the Linux boot process.
- Manage users, groups, permissions, and services.
- Diagnose CPU, memory, storage, and process issues.
- Monitor system performance using standard Linux tools.
- Troubleshoot common production problems methodically.
- Administer Linux systems securely.
- Use essential Linux commands confidently.
- Prepare for Linux-focused HPC infrastructure interviews.

---

# Linux Administration Mind Map

```text
                    Linux Administration
                           │
     ┌─────────────────────┼─────────────────────┐
     │                     │                     │
 Architecture         Processes           Filesystems
     │                     │                     │
 Boot Process         systemd             XFS / EXT4
     │                     │                     │
 Memory              Services              LVM / RAID
     │                     │                     │
 Networking         Troubleshooting       Performance
     │                     │                     │
 Security           Monitoring            Automation
```

---

# Next Chapter

# Chapter 3 – Networking

Topics include:

- OSI & TCP/IP Models
- IPv4 & IPv6
- Routing
- VLANs
- Bonding
- DNS
- MTU & Jumbo Frames
- Network Performance
- Linux Networking Tools
- HPC Network Design
- Production Troubleshooting
- Interview Questions
