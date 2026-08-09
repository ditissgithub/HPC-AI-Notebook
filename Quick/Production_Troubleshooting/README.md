# Chapter 17 – Production Troubleshooting

## Part 1 – Troubleshooting Methodology & Linux

> **Notebook focus:** A concise production troubleshooting workflow for HPC-AI infrastructure. Start with evidence, isolate the failing layer, fix safely, and validate.

* [17.1 Production Troubleshooting Mindset](#171-production-troubleshooting-mindset)
* [17.2 The 7-Step Troubleshooting Workflow](#172-the-7-step-troubleshooting-workflow)
* [17.3 Identify the Scope](#173-identify-the-scope)
* [17.4 Linux Troubleshooting Layers](#174-linux-troubleshooting-layers)
* [17.5 CPU Problems](#175-cpu-problems)
* [17.6 Memory Problems](#176-memory-problems)
* [17.7 Disk and I/O Problems](#177-disk-and-io-problems)
* [17.8 Process Problems](#178-process-problems)
* [17.9 Service Problems](#179-service-problems)
* [17.10 Kernel Problems](#1710-kernel-problems)
* [17.11 Production Checklist](#1711-production-checklist)
* [17.12 Quick Revision](#1712-quick-revision)

## Part 2 – Slurm, Node & Job Troubleshooting

> **Notebook focus:** Quickly diagnose common production problems involving Slurm controllers, compute nodes, jobs, partitions, QoS and node states.

* [17.13 Slurm Troubleshooting Model](#1713-slurm-troubleshooting-model)
* [17.14 Node Down or Drain](#1714-node-down-or-drain)
* [17.15 slurmd Not Running](#1715-slurmd-not-running)
* [17.16 Node Not Registering](#1716-node-not-registering)
* [17.17 Job Pending](#1717-job-pending)
* [17.18 Job Failed](#1718-job-failed)
* [17.19 Job Out of Memory](#1719-job-out-of-memory)
* [17.20 Job Timeout](#1720-job-timeout)
* [17.21 Partition Problems](#1721-partition-problems)
* [17.22 QoS and Account Problems](#1722-qos-and-account-problems)
* [17.23 Slurm Controller Troubleshooting](#1723-slurm-controller-troubleshooting)
* [17.24 Production Recovery Workflow](#1724-production-recovery-workflow)
* [17.25 Quick Revision](#1725-quick-revision)
