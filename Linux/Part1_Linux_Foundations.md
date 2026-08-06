## Part 1 – Linux Foundations

- [2.1 Introduction to Linux](#21-introduction-to-linux)
- [2.2 Linux Architecture](#22-linux-architecture)
- [2.3 Linux Kernel](#23-linux-kernel)
- [2.4 User Space vs Kernel Space](#24-user-space-vs-kernel-space)
- [2.5 Linux Boot Process Overview](#25-linux-boot-process-overview)
- [2.6 Linux Filesystem Hierarchy (FHS)](#26-linux-filesystem-hierarchy-fhs)
- [2.7 Linux File Types](#27-linux-file-types)
- [2.8 Inodes](#28-inodes)
- [2.9 Mount Points](#29-mount-points)


# 2.1 Introduction to Linux

## What is Linux?

Linux is an open-source, Unix-like operating system kernel created by **Linus Torvalds** in 1991. Combined with user-space tools from the GNU Project and other open-source communities, it forms a complete operating system used in servers, cloud platforms, embedded systems, supercomputers, and AI infrastructure.

Today, Linux powers:

- Most of the world's supercomputers
- Large cloud platforms
- Enterprise data centers
- Kubernetes clusters
- AI training platforms
- HPC clusters

---

## Why Linux Dominates HPC

Linux provides:

- High performance
- Excellent hardware support
- Stability
- Scalability
- Automation capabilities
- Strong networking stack
- Advanced filesystem support
- GPU support
- Open-source flexibility

These characteristics make Linux the preferred operating system for scientific computing.

---

## Linux in an HPC Cluster

```
                  User Applications
                         │
                         ▼
                    Slurm Scheduler
                         │
                         ▼
                 Linux Operating System
                         │
      ┌──────────────────┼──────────────────┐
      ▼                  ▼                  ▼
     CPU              Memory            Storage
      │                  │                  │
      └──────────────────┼──────────────────┘
                         │
                         ▼
                  InfiniBand Network
```

Linux is the foundation upon which every other HPC technology operates.

---

# 2.2 Linux Architecture

Linux follows a layered architecture.

```
+--------------------------------------+
|         User Applications            |
+--------------------------------------+
|        System Libraries              |
+--------------------------------------+
|        System Call Interface         |
+--------------------------------------+
|          Linux Kernel                |
+--------------------------------------+
| Device Drivers | CPU | RAM | Disk    |
| Network | GPU | Filesystem           |
+--------------------------------------+
|             Hardware                 |
+--------------------------------------+
```

---

## Architecture Components

### Applications

Programs executed by users.

Examples:

- Bash
- Python
- Slurm commands
- CUDA applications
- MPI programs

---

### System Libraries

Libraries provide APIs for applications.

Examples:

- glibc
- libpthread
- libm

Applications interact with the kernel through these libraries.

---

### System Calls

System calls are the interface between user programs and the kernel.

Common system calls include:

- `open()`
- `read()`
- `write()`
- `fork()`
- `execve()`
- `mount()`
- `socket()`

---

### Linux Kernel

The kernel is the core of the operating system.

Responsibilities include:

- Process management
- Memory management
- Device management
- Filesystem management
- Networking
- Security

---

### Hardware

The kernel communicates directly with physical hardware through device drivers.

Examples:

- CPUs
- RAM
- Storage devices
- Network adapters
- GPUs

---

# 2.3 Linux Kernel

## What is the Kernel?

The Linux kernel is the central component of the operating system.

It manages hardware resources and provides services to user applications.

```
Application

↓

System Call

↓

Linux Kernel

↓

Hardware
```

---

## Kernel Responsibilities

The kernel manages:

- CPU scheduling
- Memory allocation
- Process creation
- Filesystems
- Network communication
- Interrupt handling
- Device drivers
- Security

Without the kernel, applications cannot access hardware directly.

---

## Kernel Modules

Linux supports dynamically loadable kernel modules.

Examples:

- NVIDIA GPU driver
- Mellanox OFED driver
- Filesystem modules
- RAID drivers

Useful commands:

```bash
lsmod
modprobe
modinfo
rmmod
```

Example:

```bash
lsmod | grep nvidia
```

---

## Kernel Version

Display the running kernel version:

```bash
uname -r
```

Example output:

```text
5.14.0-570.el9.x86_64
```

Display complete kernel information:

```bash
uname -a
```

---

# 2.4 User Space vs Kernel Space

Linux separates execution into two protection domains.

```
+-----------------------------+
|        User Space           |
| Bash                        |
| Python                      |
| Slurm                       |
| MPI                         |
| CUDA                        |
+-------------▲---------------+
              │ System Calls
+-------------▼---------------+
|       Kernel Space          |
| Scheduler                   |
| Memory Manager              |
| Device Drivers              |
| Network Stack               |
| Filesystems                 |
+-----------------------------+
```

---

## User Space

User applications execute in User Space.

Characteristics:

- Limited hardware access
- Protected memory
- Lower privileges
- Safer execution

Examples:

- Bash
- Vim
- Python
- Slurm client
- SSH

---

## Kernel Space

Kernel Space has unrestricted access to hardware.

Characteristics:

- Full hardware control
- Privileged execution
- Device drivers
- Process scheduler

Errors in kernel space can crash the entire operating system.

---

## Why Separation Matters

Benefits include:

- Stability
- Security
- Memory protection
- Fault isolation

---

# 2.5 Linux Boot Process Overview

When a Linux server starts, several components work together before the login prompt appears.

```
Power On

↓

BIOS / UEFI

↓

Bootloader (GRUB2)

↓

Linux Kernel

↓

initramfs

↓

systemd

↓

Services

↓

Login Prompt
```

---

## Step 1 — BIOS / UEFI

Responsibilities:

- Hardware initialization
- POST (Power-On Self-Test)
- Detect boot devices
- Load bootloader

---

## Step 2 — GRUB2

GRUB2 loads:

- Linux kernel
- initramfs

Useful commands:

```bash
grubby --default-kernel
grubby --info ALL
```

---

## Step 3 — Linux Kernel

The kernel initializes:

- CPU
- Memory
- Device drivers
- Storage controllers
- Filesystems

---

## Step 4 — initramfs

Initial RAM filesystem provides temporary drivers required to mount the real root filesystem.

---

## Step 5 — systemd

`systemd` becomes PID 1.

Responsibilities:

- Start services
- Mount filesystems
- Initialize networking
- Reach the target state

---

## Step 6 — Login

System becomes ready for user access.

---

## Viewing Boot Logs

```bash
journalctl -b
```

Previous boot:

```bash
journalctl -b -1
```

---

# 2.6 Linux Filesystem Hierarchy (FHS)

Linux organizes files into a standardized directory structure.

```
/

├── bin
├── boot
├── dev
├── etc
├── home
├── lib
├── media
├── mnt
├── opt
├── proc
├── root
├── run
├── sbin
├── srv
├── sys
├── tmp
├── usr
└── var
```

---

## Important Directories

| Directory | Purpose |
|------------|---------|
| `/` | Root filesystem |
| `/boot` | Kernel and bootloader files |
| `/etc` | System configuration |
| `/home` | User home directories |
| `/root` | Root user's home |
| `/var` | Logs, mail, spool, databases |
| `/tmp` | Temporary files |
| `/usr` | User programs and libraries |
| `/dev` | Device files |
| `/proc` | Process information |
| `/sys` | Kernel and hardware information |
| `/mnt` | Temporary mount points |
| `/opt` | Optional software |

---

## Important Commands

Display filesystem usage:

```bash
df -h
```

Display directory usage:

```bash
du -sh /home
```

Display mounted filesystems:

```bash
mount
```

---

# 2.7 Linux File Types

Linux treats almost everything as a file.

View file types:

```bash
ls -l
```

Example:

```text
-rw-r--r-- file.txt
drwxr-xr-x directory
lrwxrwxrwx link
```

---

## Common File Types

| Symbol | Type |
|----------|------|
| `-` | Regular file |
| `d` | Directory |
| `l` | Symbolic link |
| `b` | Block device |
| `c` | Character device |
| `p` | Named pipe |
| `s` | Socket |

---

Useful command:

```bash
file filename
```

Example:

```bash
file /bin/bash
```

---

# 2.8 Inodes

Every file in Linux is represented by an inode.

An inode stores metadata, not the filename.

```
Filename

↓

Inode

↓

Data Blocks
```

---

## Inode Contains

- Owner
- Group
- Permissions
- Size
- Timestamps
- Data block pointers

---

## View Inode

```bash
ls -i
```

Display inode usage:

```bash
df -i
```

---

## Why Inodes Matter

A filesystem can run out of inodes even if disk space is still available.

This is common on systems containing millions of small files.

---

# 2.9 Mount Points

Linux mounts every filesystem into a single directory tree.

```
/

├── home
├── boot
├── data
├── lustre
└── backup
```

Unlike Windows, Linux does not use drive letters.

---

## View Mounted Filesystems

```bash
mount
```

or

```bash
findmnt
```

---

## Mount a Filesystem

```bash
mount /dev/sdb1 /data
```

Unmount:

```bash
umount /data
```

---

## Persistent Mounts

Permanent mounts are defined in:

```text
/etc/fstab
```

Example:

```text
UUID=xxxx  /data   xfs   defaults   0 0
```

Reload after changes:

```bash
mount -a
```

---

# Production Insight

Linux is the operational foundation of every HPC cluster. Whether troubleshooting Slurm scheduling issues, diagnosing Lustre mount failures, configuring InfiniBand interfaces, or validating NVIDIA drivers, administrators ultimately rely on Linux kernel services, filesystems, networking, processes, and logs. Strong Linux fundamentals significantly reduce troubleshooting time across the entire HPC software stack.

---

# Key Takeaways

- Linux is the foundation of modern HPC and AI infrastructure.
- The kernel manages hardware resources and system services.
- User Space and Kernel Space provide security and stability through isolation.
- The Linux boot process progresses from firmware to GRUB2, kernel, initramfs, systemd, and user login.
- The Filesystem Hierarchy Standard (FHS) organizes system files into predictable locations.
- Linux supports multiple file types beyond regular files and directories.
- Inodes store file metadata independently of filenames.
- Mount points integrate multiple filesystems into a unified directory tree.

---

## Next Part

**Part 2 – Process & Service Management**

Topics covered:

- Process Management
- Process States
- CPU Scheduling
- Signals
- Services
- systemd
- Journald
- Boot Targets
- Essential Process Management Commands
