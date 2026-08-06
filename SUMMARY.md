# SUMMARY

# HPC-AI Infrastructure Engineer Notebook

**Version:** 1.0.0

---

# Welcome

Welcome to the **HPC-AI Infrastructure Engineer Notebook**.

This handbook is designed as a complete learning and reference guide for engineers responsible for designing, deploying, operating, automating, and troubleshooting High Performance Computing (HPC) and Artificial Intelligence (AI) infrastructure.

The notebook follows the complete lifecycle of an HPC cluster—from the operating system and networking layers to provisioning, scheduling, storage, GPU acceleration, and production operations.

Unlike traditional documentation, each chapter combines:

- Core concepts
- Internal architecture
- Production workflows
- Hands-on commands
- Troubleshooting guides
- Best practices
- Real-world examples
- Interview preparation

---

# Learning Roadmap

The recommended order of study is shown below.

```
                 Linux
                    │
                    ▼
              Networking
                    │
                    ▼
            InfiniBand Fabric
                    │
                    ▼
              NVIDIA GPU
                    │
                    ▼
             LDAP Services
                    │
                    ▼
             xCAT Provisioning
                    │
                    ▼
          Slurm Workload Manager
                    │
                    ▼
            Lustre File System
                    │
                    ▼
          HPC-AI Architecture
                    │
                    ▼
       Production Operations
                    │
                    ▼
             OpenCHAI Platform
```

Each chapter builds upon the previous one. Skipping foundational chapters may make advanced topics more difficult to understand.

---

# Notebook Structure

| Chapter | Topic | Difficulty | Estimated Time |
|----------|-------|------------|----------------|
| 01 | Introduction | Beginner | 2 Hours |
| 02 | Linux | Beginner → Advanced | 10–15 Hours |
| 03 | Networking | Intermediate | 6–8 Hours |
| 04 | InfiniBand | Advanced | 6 Hours |
| 05 | NVIDIA GPU | Intermediate | 6 Hours |
| 06 | xCAT Provisioning | Advanced | 10 Hours |
| 07 | Slurm Workload Manager | Advanced | 12 Hours |
| 08 | LDAP Authentication | Intermediate | 5 Hours |
| 09 | Lustre Storage | Advanced | 8 Hours |
| 10 | HPC-AI Architecture | Advanced | 5 Hours |
| 11 | Production Troubleshooting | Advanced | Continuous |
| 12 | Command Cheat Sheet | Reference | As Needed |

---

# Chapter Overview

## 01 - Introduction

This chapter introduces the HPC ecosystem and explains how modern AI infrastructure is built.

Topics include:

- What is HPC?
- What is AI Infrastructure?
- Cluster architecture
- HPC components
- Roles of an HPC Infrastructure Engineer
- Data flow through an HPC cluster
- Compute, storage, and networking overview

By the end of this chapter, you should understand how all major components fit together.

---

## 02 - Linux

Linux is the foundation of every HPC cluster.

Topics include:

- Linux architecture
- Boot process
- Filesystems
- Process management
- Memory management
- CPU scheduling
- NUMA
- Systemd
- User management
- Permissions
- Networking basics
- Storage devices
- Performance monitoring
- Package management
- Troubleshooting

Hands-on examples include:

- Managing services
- Monitoring system performance
- Debugging memory issues
- Identifying disk bottlenecks
- Investigating high CPU usage

---

## 03 - Networking

Networking connects every component of an HPC cluster.

Topics include:

- OSI model
- TCP/IP
- Ethernet
- IP addressing
- Routing
- VLANs
- Bonding
- DNS
- DHCP
- MTU
- Jumbo Frames
- Firewalls

Practical skills include:

- Capturing packets
- Testing throughput
- Diagnosing connectivity issues
- Configuring interfaces
- Measuring latency

---

## 04 - InfiniBand

InfiniBand provides high-speed, low-latency communication between compute nodes.

Topics include:

- RDMA
- OpenSM
- OFED
- HCAs
- Switches
- LIDs
- GPUDirect RDMA
- Performance tuning

Real-world scenarios include:

- Link failures
- Port state troubleshooting
- Subnet Manager issues
- Bandwidth testing

---

## 05 - NVIDIA GPU

GPU acceleration is central to modern AI workloads.

Topics include:

- CUDA
- Drivers
- MIG
- NVML
- NCCL
- Multi-GPU communication
- GPU topology
- GPU scheduling

Hands-on examples include:

- Monitoring GPU utilization
- Diagnosing driver mismatches
- Running CUDA applications
- Measuring GPU performance

---

## 06 - xCAT Provisioning

xCAT automates cluster provisioning and lifecycle management.

Topics include:

- xCAT architecture
- PXE boot
- DHCP
- TFTP
- OS images
- Node definitions
- Stateful deployment
- Stateless deployment
- Discovery
- Postscripts

Practical exercises include:

- Adding compute nodes
- Building OS images
- Provisioning new systems
- Updating nodes

---

## 07 - Slurm Workload Manager

Slurm schedules and manages jobs across the cluster.

Topics include:

- Scheduler architecture
- slurmctld
- slurmd
- slurmdbd
- Partitions
- Accounts
- QoS
- Fairshare
- GRES
- GPU scheduling

Hands-on examples include:

- Submitting jobs
- Monitoring queues
- Managing partitions
- Debugging pending jobs
- Draining nodes

---

## 08 - LDAP Authentication

LDAP provides centralized authentication.

Topics include:

- Directory structure
- LDAP schema
- Users
- Groups
- PAM
- NSS
- SSSD
- Replication

Practical tasks include:

- Creating users
- Managing groups
- Testing authentication
- Troubleshooting login failures

---

## 09 - Lustre Storage

Lustre is a parallel filesystem designed for HPC workloads.

Topics include:

- MGS
- MDT
- OST
- OSS
- Clients
- LNet
- Striping
- Quotas

Hands-on exercises include:

- Mounting clients
- Configuring striping
- Monitoring filesystem health
- Diagnosing storage failures

---

## 10 - HPC-AI Architecture

This chapter explains how all infrastructure components integrate to support AI and scientific computing.

Topics include:

- Login nodes
- Compute nodes
- GPU clusters
- Storage
- Networking
- Authentication
- Scheduling
- Monitoring
- Container platforms
- OpenCHAI concepts

---

## 11 - Production Troubleshooting

A collection of real-world operational scenarios.

Examples include:

- Compute node not booting
- GPU unavailable
- Slurm jobs stuck
- LDAP authentication failure
- Lustre client timeout
- InfiniBand port down
- PXE boot failure
- DNS issues
- Disk bottlenecks

Each scenario follows a structured troubleshooting methodology:

1. Symptoms
2. Investigation
3. Root cause
4. Resolution
5. Prevention

---

## 12 - Command Cheat Sheet

A quick-reference section containing commonly used commands.

Categories include:

- Linux
- Networking
- InfiniBand
- NVIDIA GPU
- xCAT
- Slurm
- LDAP
- Lustre
- Storage
- Performance monitoring

This chapter is intended for day-to-day operational use.

---

# How to Use This Notebook

For each chapter:

1. Read the concepts.
2. Study the architecture diagrams.
3. Practice the commands.
4. Perform the suggested lab exercises.
5. Review troubleshooting scenarios.
6. Answer the interview questions.
7. Document your observations.

Active practice is essential for mastering infrastructure engineering.

---

# Suggested Lab Environment

Minimum setup:

- 1 Management Node
- 2 Compute Nodes
- Rocky Linux 9.x
- 8 GB RAM per VM
- VirtualBox, VMware, or KVM

Recommended additions:

- OpenLDAP
- Slurm
- xCAT
- Lustre
- Docker or Podman
- NVIDIA GPU (optional)
- InfiniBand (physical hardware if available)

---

# Symbols Used Throughout the Notebook

| Symbol | Meaning |
|--------|---------|
| 💡 | Key Concept |
| 🔧 | Configuration |
| 🛠 | Troubleshooting |
| ⚠️ | Warning |
| ✅ | Best Practice |
| 📌 | Interview Tip |
| 🚀 | Performance Optimization |

---

# Final Advice

Becoming an HPC-AI Infrastructure Engineer requires more than learning commands. Develop a deep understanding of how operating systems, networks, storage, schedulers, and accelerators interact to build reliable, high-performance platforms.

Study systematically, practice regularly, and treat every troubleshooting exercise as an opportunity to improve your engineering intuition.

---

**Next Chapter:** `01-Introduction.md`
