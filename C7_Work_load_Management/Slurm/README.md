# Chapter 7 – Slurm Workload Manager

> **Notebook Goal:** Understand Slurm from an HPC-AI Infrastructure Engineer's perspective — architecture, scheduling, resources, commands, production operations, and troubleshooting.

---

## Table of Contents

### Part 1 – Slurm Architecture & Job Lifecycle

* [7.1 What is Slurm?](#71-what-is-slurm)
* [7.2 Why HPC Uses Slurm](#72-why-hpc-uses-slurm)
* [7.3 Slurm Architecture](#73-slurm-architecture)
* [7.4 Core Slurm Components](#74-core-slurm-components)
* [7.5 Job Lifecycle](#75-job-lifecycle)
* [7.6 Slurm Configuration Files](#76-slurm-configuration-files)

### Part 2 – Nodes, Partitions, Accounts & QoS

* [7.7 Nodes](#77-nodes)
* [7.8 Partitions](#78-partitions)
* [7.9 Accounts](#79-accounts)
* [7.10 Quality of Service](#710-quality-of-service)
* [7.11 Resources](#711-resources)
* [7.12 CPU, Memory and GPU Allocation](#712-cpu-memory-and-gpu-allocation)

### Part 3 – Jobs & Essential Commands

* [7.13 Interactive Jobs](#713-interactive-jobs)
* [7.14 Batch Jobs](#714-batch-jobs)
* [7.15 Job Arrays](#715-job-arrays)
* [7.16 Important User Commands](#716-important-user-commands)
* [7.17 Important Administrator Commands](#717-important-administrator-commands)

### Part 4 – Scheduling & Production Operations

* [7.18 How Slurm Schedules Jobs](#718-how-slurm-schedules-jobs)
* [7.19 Priority and Fairness](#719-priority-and-fairness)
* [7.20 Slurm Database](#720-slurm-database)
* [7.21 Monitoring Jobs and Nodes](#721-monitoring-jobs-and-nodes)
* [7.22 Slurm + xCAT](#722-slurm--xcat)
* [7.23 Slurm + GPU](#723-slurm--gpu)

### Part 5 – Troubleshooting & Best Practices

* [7.24 Common Node Problems](#724-common-node-problems)
* [7.25 Common Job Problems](#725-common-job-problems)
* [7.26 GPU Scheduling Problems](#726-gpu-scheduling-problems)
* [7.27 Troubleshooting Workflow](#727-troubleshooting-workflow)
* [7.28 Production Best Practices](#728-production-best-practices)

### Part 6 – Interview & Quick Revision

* [7.29 Real-World HPC Scenarios](#729-real-world-hpc-scenarios)
* [7.30 Interview Questions](#730-interview-questions)
* [7.31 Essential Slurm Commands](#731-essential-slurm-commands)
* [7.32 Final Revision](#732-final-revision)

---
