# HPC-AI Infrastructure Engineer Notebook

**Version:** 1.0.0

**Author:** Satish Gupta

**Organization:** HPC-AI World

---

# About this Notebook

This notebook is a practical engineering guide for designing, deploying, operating, automating, and troubleshooting High Performance Computing (HPC) and Artificial Intelligence (AI) infrastructure.

Unlike traditional Linux or HPC books, this notebook focuses on **real-world production infrastructure** used in enterprise and national supercomputing environments.

The goal is to bridge the gap between:

- Linux System Administration
- HPC Cluster Administration
- AI Infrastructure
- GPU Computing
- Cluster Provisioning
- Cluster Scheduling
- Storage
- Networking
- Infrastructure Automation

This notebook is designed to become a long-term engineering reference that can be used during:

- Daily operations
- Production troubleshooting
- Cluster deployment
- Interview preparation
- Infrastructure design
- OpenCHAI development
- Learning new technologies

---

# Why this Notebook?

Most books teach technologies independently.

For example:

- Linux
- Networking
- Slurm
- Lustre
- NVIDIA GPU

However, production HPC clusters combine all of these technologies together.

Example:

```
               Users
                  │
                  ▼
           Login Node
                  │
                  ▼
         Slurm Scheduler
                  │
        ┌─────────┴─────────┐
        │                   │
        ▼                   ▼
 Compute Node 1      Compute Node 2
        │                   │
        ├─────────┬─────────┤
                  │
            InfiniBand Fabric
                  │
                  ▼
             Lustre Storage
                  │
                  ▼
             GPU Resources
```

An HPC Infrastructure Engineer must understand how every component works together.

---

# Objectives

After completing this notebook, you should be able to:

- Understand Linux from an infrastructure perspective.
- Design HPC cluster architecture.
- Deploy compute nodes using xCAT.
- Configure Slurm Workload Manager.
- Manage LDAP authentication.
- Deploy Lustre storage.
- Troubleshoot InfiniBand networking.
- Manage NVIDIA GPU clusters.
- Build AI infrastructure.
- Understand OpenCHAI architecture.
- Diagnose production issues.
- Automate infrastructure tasks.
- Prepare for senior HPC Engineering Role.

---

# Target Audience

This notebook is intended for:

- Linux System Administrators
- HPC Engineers
- AI Infrastructure Engineers
- DevOps Engineers
- Site Reliability Engineers (SRE)
- Cluster Administrators
- GPU Infrastructure Engineers
- Research Computing Engineers
- Infrastructure Architects

---

# Prerequisites

Readers should have basic knowledge of:

- Linux commands
- Bash scripting
- Networking fundamentals
- SSH
- Filesystems
- Virtual Machines

No prior HPC experience is required.

---

# Learning Roadmap

The recommended learning sequence is:

```
Linux
   │
   ▼
Networking
   │
   ▼
Storage
   │
   ▼
GPU
   │
   ▼
InfiniBand
   │
   ▼
LDAP
   │
   ▼
xCAT
   │
   ▼
Slurm
   │
   ▼
Automation
   │
   ▼
Production Operations
   │
   ▼
OpenCHAI
```

---

# Repository Structure

```
HPC-AI-Infrastructure-Engineer-Notebook/

├── README.md
├── SUMMARY.md
│
├── 01-Introduction.md
├── 02-Linux.md
├── 03-Networking.md
├── 04-InfiniBand.md
├── 05-NVIDIA-GPU.md
├── 06-xCAT.md
├── 07-Slurm.md
├── 08-LDAP.md
├── 09-Lustre.md
├── 10-HPC-AI-Architecture.md
├── 11-Production-Troubleshooting.md
├── 12-Command-CheatSheet.md
│
├── diagrams/
├── images/
└── references/
```

---

# Notebook Design Philosophy

Every chapter follows the same structure.

```
Concept

↓

Architecture

↓

Components

↓

Workflow

↓

Commands

↓

Examples

↓

Production Notes

↓

Troubleshooting

↓

Interview Questions

↓

Best Practices
```

This ensures consistency throughout the notebook.

---

# Engineering Philosophy

An HPC Infrastructure Engineer should never rely solely on memorizing commands.

Instead, focus on understanding:

- Why does the technology exist?
- What problem does it solve?
- How does it work internally?
- How does it interact with other components?
- How do you troubleshoot it?
- What happens when it fails?

This notebook emphasizes systems thinking rather than command memorization.

---

# Practical Lab Environment

The examples in this notebook can be practiced using:

## Minimum Lab

- 1 Management VM
- 2 Compute VMs
- Rocky Linux 9.x
- 8 GB RAM per VM
- VirtualBox / VMware / KVM

---

## Recommended Lab

- Management Node
- Login Node
- Two Compute Nodes
- Shared Storage
- Docker
- Podman
- Slurm
- OpenLDAP
- NVIDIA GPU (optional)

---

# Core Technologies Covered

## Linux

- Filesystem
- Processes
- Services
- Users
- Permissions
- Performance
- Troubleshooting

---

## Networking

- TCP/IP
- Routing
- VLAN
- Bonding
- DNS
- DHCP
- MTU
- Network Performance

---

## InfiniBand

- OFED
- OpenSM
- RDMA
- GPUDirect RDMA
- Performance Tuning

---

## NVIDIA GPU

- CUDA
- Drivers
- MIG
- NVML
- NCCL
- DCGM

---

## xCAT

- Architecture
- PXE Boot
- Discovery
- Stateful Provisioning
- Stateless Provisioning
- Images
- DHCP
- DNS

---

## Slurm

- Scheduling
- Partitions
- QoS
- Fairshare
- Accounting
- GRES
- GPU Scheduling

---

## LDAP

- Authentication
- NSS
- PAM
- SSSD
- Replication

---

## Lustre

- MGS
- MDT
- OST
- OSS
- LNet
- Client Configuration

---

## AI Infrastructure

- GPU Clusters
- AI Workloads
- Containers
- NCCL
- CUDA
- PyTorch
- TensorFlow
- Kubernetes Overview
- Slurm Integration

---

# Production Mindset

Production infrastructure differs significantly from lab environments.

In production:

- High Availability (HA) is essential.
- Monitoring is mandatory.
- Security is continuous.
- Automation is expected.
- Downtime must be minimized.
- Documentation is part of engineering.

Throughout this notebook, emphasis is placed on production-ready practices rather than one-off demonstrations.

---

# Troubleshooting Philosophy

When a problem occurs:

1. Observe symptoms.
2. Verify assumptions.
3. Collect logs.
4. Check dependencies.
5. Identify the root cause.
6. Implement a fix.
7. Validate the solution.
8. Document the incident.

Avoid changing multiple variables simultaneously during troubleshooting.

---

# Versioning

This notebook follows semantic versioning.

```
v1.0.0

Major:
Architecture changes

Minor:
New chapters

Patch:
Corrections
```

---

# How to Study

Recommended order:

1. Read the concept.
2. Understand the architecture.
3. Practice commands.
4. Perform the lab.
5. Break the system intentionally.
6. Troubleshoot.
7. Review interview questions.
8. Document your findings.

Hands-on practice is essential.

---

# Future Expansion

Future versions may include:

- Kubernetes for AI
- NVIDIA AI Enterprise
- OpenCHAI Control Plane
- Ceph Storage
- Grafana
- Prometheus
- ELK Stack
- HAProxy
- Keepalived
- Docker Swarm
- Ansible Automation
- Infrastructure as Code
- AI Factory Architecture

---

# Conventions Used

| Symbol | Meaning |
|---------|---------|
| 💡 | Important Concept |
| ⚠️ | Warning |
| ✅ | Best Practice |
| 🔧 | Configuration |
| 🛠 | Troubleshooting |
| 📌 | Interview Tip |

---

# References

Primary references include:

- Linux Documentation Project
- Rocky Linux Documentation
- Slurm Documentation
- xCAT Documentation
- OpenLDAP Documentation
- Lustre Documentation
- NVIDIA Documentation
- Mellanox/NVIDIA Networking Documentation
- OpenHPC Documentation

Readers are encouraged to consult official documentation alongside this notebook for version-specific details.

---

# Final Note

Infrastructure engineering is not about knowing every command—it is about understanding how systems interact, how failures propagate, and how to build reliable, scalable platforms.

Approach each topic with curiosity, practice consistently, and focus on building a strong mental model of the entire HPC-AI ecosystem. Over time, this notebook should become both a learning resource and a trusted operational reference.
