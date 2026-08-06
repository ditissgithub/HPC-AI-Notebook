# Part 2 – Process & Service Management

- [2.10 Process Management](#210-process-management)
- [2.11 Linux Process States](#211-linux-process-states)
- [2.12 CPU Scheduling](#212-cpu-scheduling)
- [2.13 Signals](#213-signals)
- [2.14 Services](#214-services)
- [2.15 systemd](#215-systemd)
- [2.16 systemctl Commands](#216-systemctl-commands)
- [2.17 Journald](#217-journald)
- [2.18 Boot Targets](#218-boot-targets)
- [2.19 Service Troubleshooting Workflow](#219-service-troubleshooting-workflow)
- [Production Insight](#production-insight)
- [Key Takeaways](#key-takeaways)

---

# 2.10 Process Management

## What is a Process?

A **process** is an executing instance of a program.

For example:

```
Executable

↓

Kernel

↓

Running Process
```

Every command you execute creates one or more processes.

Example:

```bash
ls
```

```
bash

↓

fork()

↓

ls Process

↓

Exit
```

---

## Process Components

Each process contains:

- Process ID (PID)
- Parent Process ID (PPID)
- User ID (UID)
- Memory Space
- Open Files
- Environment Variables
- Scheduling Information

---

## Process Hierarchy

Linux follows a parent-child relationship.

```
systemd (PID 1)

│

├── sshd

│      └── bash

│              └── vim

│

├── crond

├── rsyslog

└── slurmd
```

Every process (except PID 1) has a parent.

---

## Important Commands

Display running processes

```bash
ps -ef
```

BSD format

```bash
ps aux
```

Interactive process viewer

```bash
top
```

Improved viewer

```bash
htop
```

Process tree

```bash
pstree
```

Search process

```bash
pgrep sshd
```

Locate process

```bash
pidof sshd
```

---

## Process Information

View a specific process

```bash
ps -fp <PID>
```

Example

```bash
ps -fp 1432
```

Display process hierarchy

```bash
pstree -p
```

---

# 2.11 Linux Process States

Every process transitions through different states during execution.

```
New

↓

Ready

↓

Running

↓

Sleeping

↓

Stopped

↓

Zombie

↓

Terminated
```

---

## Process States

| State | Meaning |
|--------|----------|
| R | Running or Runnable |
| S | Interruptible Sleep |
| D | Uninterruptible Sleep (usually I/O) |
| T | Stopped |
| Z | Zombie |
| I | Idle Kernel Thread |

---

## View Process State

```bash
ps -eo pid,ppid,stat,cmd
```

Example

```text
PID   STAT CMD

2314  R    bash

5432  S    sshd

7890  D    kworker

9001  Z    python
```

---

## Zombie Process

Zombie processes have completed execution but remain in the process table because the parent has not collected their exit status.

```
Parent

↓

Child exits

↓

Zombie

↓

Parent waits()

↓

Removed
```

View zombies

```bash
ps aux | grep Z
```

---

## D-State Processes

"D" state indicates **Uninterruptible Sleep**.

Usually caused by:

- Storage problems
- NFS issues
- Lustre issues
- Disk I/O wait

These processes cannot be killed until the kernel operation completes.

---

# 2.12 CPU Scheduling

The Linux scheduler decides which process executes next.

```
Ready Queue

↓

CPU Scheduler

↓

CPU Core
```

The scheduler attempts to maximize:

- Fairness
- Throughput
- Responsiveness

---

## Scheduling Priorities

Linux uses **nice values**.

Range

```
-20

↓

0

↓

19
```

Lower value

↓

Higher Priority

---

## View Priority

```bash
ps -eo pid,ni,pri,cmd
```

---

## Change Priority

Start process

```bash
nice -n 10 command
```

Modify running process

```bash
renice -n 5 -p PID
```

Example

```bash
renice -n -5 -p 1023
```

---

## CPU Affinity

Bind process to specific CPUs.

View affinity

```bash
taskset -p PID
```

Assign affinity

```bash
taskset -cp 0-7 PID
```

Useful in HPC environments.

---

# 2.13 Signals

Signals provide communication between processes.

```
Kernel

↓

Signal

↓

Process
```

---

## Common Signals

| Signal | Number | Purpose |
|----------|----------|----------|
| SIGHUP | 1 | Reload configuration |
| SIGINT | 2 | Ctrl+C |
| SIGQUIT | 3 | Quit |
| SIGKILL | 9 | Force kill |
| SIGTERM | 15 | Graceful termination |
| SIGSTOP | 19 | Pause process |
| SIGCONT | 18 | Resume process |

---

## Send Signal

Terminate

```bash
kill PID
```

Force kill

```bash
kill -9 PID
```

Reload daemon

```bash
kill -HUP PID
```

Kill by process name

```bash
pkill sshd
```

---

## Best Practice

Prefer

```bash
SIGTERM
```

before

```bash
SIGKILL
```

because SIGTERM allows applications to clean up resources.

---

# 2.14 Services

Linux services are background processes providing system functionality.

Examples:

- sshd
- chronyd
- rsyslog
- firewalld
- slurmd
- munge
- xcatd

---

## Service Architecture

```
systemd

↓

Service

↓

Background Process
```

---

## Service Unit Files

Usually stored in

```
/usr/lib/systemd/system/

or

/etc/systemd/system/
```

---

## Example

```
sshd.service
```

```
ExecStart=/usr/sbin/sshd
```

---

# 2.15 systemd

systemd is the init system used by modern Linux distributions.

PID 1

```
Power On

↓

Kernel

↓

systemd

↓

Services

↓

Login
```

---

## Responsibilities

- Start services
- Stop services
- Restart services
- Dependency management
- Boot targets
- Logging
- Timers

---

## Verify PID 1

```bash
ps -p 1
```

Output

```
systemd
```

---

# 2.16 systemctl Commands

List services

```bash
systemctl list-units
```

Check service

```bash
systemctl status sshd
```

Start

```bash
systemctl start sshd
```

Stop

```bash
systemctl stop sshd
```

Restart

```bash
systemctl restart sshd
```

Reload

```bash
systemctl reload sshd
```

Enable

```bash
systemctl enable sshd
```

Disable

```bash
systemctl disable sshd
```

Check boot status

```bash
systemctl is-enabled sshd
```

Failed services

```bash
systemctl --failed
```

List dependencies

```bash
systemctl list-dependencies multi-user.target
```

---

## Essential Services in HPC

Examples

```
sshd

chronyd

NetworkManager

munge

slurmd

slurmctld

xcatd

nfs-server

rpcbind
```

---

# 2.17 Journald

systemd-journald stores system logs.

View logs

```bash
journalctl
```

Current boot

```bash
journalctl -b
```

Previous boot

```bash
journalctl -b -1
```

Specific service

```bash
journalctl -u sshd
```

Live logs

```bash
journalctl -f
```

Since today

```bash
journalctl --since today
```

Last hour

```bash
journalctl --since "1 hour ago"
```

Errors only

```bash
journalctl -p err
```

---

## Why Journald Matters

During troubleshooting:

```
Service Failure

↓

journalctl

↓

Error

↓

Fix

↓

Restart
```

Most production issues begin by checking logs.

---

# 2.18 Boot Targets

Targets replace traditional runlevels.

```
Power On

↓

systemd

↓

Target

↓

Services
```

---

## Common Targets

| Target | Purpose |
|----------|----------|
| poweroff.target | Shutdown |
| rescue.target | Single-user mode |
| multi-user.target | CLI mode |
| graphical.target | GUI |
| reboot.target | Restart |

---

## View Current Target

```bash
systemctl get-default
```

Change target

```bash
systemctl set-default multi-user.target
```

Switch target

```bash
systemctl isolate rescue.target
```

---

# 2.19 Service Troubleshooting Workflow

A production workflow for troubleshooting services.

```
Service Failed

↓

systemctl status

↓

journalctl

↓

Configuration

↓

Dependencies

↓

Restart

↓

Verify
```

---

## Example

```
SSH Not Working

↓

systemctl status sshd

↓

journalctl -u sshd

↓

Verify sshd_config

↓

Restart sshd

↓

Reconnect
```

---

## Common Checks

Service status

```bash
systemctl status SERVICE
```

Port

```bash
ss -tulpn
```

Logs

```bash
journalctl -u SERVICE
```

Process

```bash
ps -ef
```

Configuration

```bash
systemctl cat SERVICE
```

---

# Production Insight

Nearly every HPC infrastructure component runs as a Linux service. Examples include `slurmctld`, `slurmd`, `munged`, `xcatd`, `sshd`, `chronyd`, and monitoring agents. When one of these services fails, the first troubleshooting steps are almost always:

1. Check the service status.
2. Review the journal logs.
3. Verify configuration files.
4. Confirm dependencies are available.
5. Restart the service if appropriate.
6. Validate that the service is functioning correctly.

Developing a consistent troubleshooting workflow is far more valuable than memorizing individual commands.

---

# Key Takeaways

- A process is a running instance of a program.
- Linux organizes processes in a parent-child hierarchy.
- Process states provide insight into application behavior.
- CPU scheduling ensures efficient resource utilization.
- Signals allow processes to communicate and terminate gracefully.
- `systemd` is the initialization and service management framework used by modern Linux distributions.
- `systemctl` is the primary tool for managing services.
- `journalctl` is the first place to investigate most service failures.
- Understanding process and service management is essential for operating production HPC clusters.

---

## Next Part

**Part 3 – Performance & Storage**

Topics covered:

- Memory Management
- CPU Performance
- Disk I/O
- Filesystems (XFS & EXT4)
- LVM
- RAID
- Swap
- Performance Monitoring
- Essential Performance Commands
