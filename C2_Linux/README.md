# Chapter 2
# Linux Administration for HPC-AI Infrastructure Engineers

---

- [Learning Objectives](#learning-objectives)
- [Prerequisites](#prerequisites)
- [Estimated Reading Time](#estimated-reading-time)

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

## Part 2 – Process & Service Management 

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


## Part 3 – Performance & Storage 

- [2.20 Linux Memory Management](#220-linux-memory-management)
- [2.21 CPU Scheduling & Performance](#221-cpu-scheduling--performance)
- [2.22 Disk I/O](#222-disk-io)
- [2.23 Linux Filesystems](#223-linux-filesystems)
- [2.24 Logical Volume Manager (LVM)](#224-logical-volume-manager-lvm)
- [2.25 RAID Fundamentals](#225-raid-fundamentals)
- [2.26 Swap Memory](#226-swap-memory)
- [2.27 Performance Monitoring Tools](#227-performance-monitoring-tools)
- [2.28 Performance Troubleshooting Workflow](#228-performance-troubleshooting-workflow)

## Part 4 – Security & Administration

- [2.29 Linux User & Group Management](#229-linux-user--group-management)
- [2.30 File Ownership & Permissions](#230-file-ownership--permissions)
- [2.31 Access Control Lists (ACL)](#231-access-control-lists-acl)
- [2.32 SELinux](#232-selinux)
- [2.33 SSH Administration](#233-ssh-administration)
- [2.34 Package Management](#234-package-management)
- [2.35 Log Management](#235-log-management)
- [2.36 Cron & systemd Timers](#236-cron--systemd-timers)
- [2.37 Linux Administration Checklist](#237-linux-administration-checklist)
  
## Part 5 – Production Operations 

- [2.38 Linux Networking Basics](#238-linux-networking-basics)
- [2.39 Linux Troubleshooting Methodology](#239-linux-troubleshooting-methodology)
- [2.40 Production Troubleshooting Scenarios](#240-production-troubleshooting-scenarios)
- [2.41 HPC Linux Best Practices](#241-hpc-linux-best-practices)
- [2.42 Essential Linux Commands Cheat Sheet](#242-essential-linux-commands-cheat-sheet)
- [2.43 Linux Interview Questions](#243-linux-interview-questions)
- [2.44 Chapter Summary](#244-chapter-summary)
  
---

# Learning Objectives

After completing this part, you should be able to:

- Understand Linux architecture.
- Explain the role of the Linux kernel.
- Differentiate User Space and Kernel Space.
- Understand the Linux boot sequence.
- Navigate the Linux filesystem hierarchy.
- Identify Linux file types.
- Explain how inodes work.
- Understand mount points and filesystem mounting.

---

# Prerequisites

- Basic computer architecture
- Basic command-line familiarity
- No prior Linux administration experience required

---

# Estimated Reading Time

**60–75 Minutes**

---
