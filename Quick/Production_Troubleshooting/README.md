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

## Part 3 – Network, InfiniBand, GPU & Lustre Troubleshooting

> **Notebook focus:** Concise troubleshooting patterns for the hardware and infrastructure layers most commonly involved in HPC-AI production incidents.

* [17.26 Network Troubleshooting](#1726-network-troubleshooting)
* [17.27 Interface Down](#1727-interface-down)
* [17.28 Connectivity Problems](#1728-connectivity-problems)
* [17.29 MTU Problems](#1729-mtu-problems)
* [17.30 InfiniBand Troubleshooting](#1730-infiniband-troubleshooting)
* [17.31 RDMA Troubleshooting](#1731-rdma-troubleshooting)
* [17.32 NVIDIA GPU Troubleshooting](#1732-nvidia-gpu-troubleshooting)
* [17.33 GPU Driver Problems](#1733-gpu-driver-problems)
* [17.34 Slurm GPU Problems](#1734-slurm-gpu-problems)
* [17.35 Lustre Troubleshooting](#1735-lustre-troubleshooting)
* [17.36 Lustre Quota Problems](#1736-lustre-quota-problems)
* [17.37 Production Diagnostic Matrix](#1737-production-diagnostic-matrix)
* [17.38 Quick Revision](#1738-quick-revision)

## Part 4 – Authentication, xCAT, Cluster-Wide Failures & Incident Handling

> **Notebook focus:** Concise production troubleshooting for LDAP/SSSD, xCAT provisioning, multi-node failures and safe incident handling.

* [17.39 LDAP and SSSD Troubleshooting](#1739-ldap-and-sssd-troubleshooting)
* [17.40 Authentication Failure Flow](#1740-authentication-failure-flow)
* [17.41 SSH Login Problems](#1741-ssh-login-problems)
* [17.42 xCAT Provisioning Troubleshooting](#1742-xcat-provisioning-troubleshooting)
* [17.43 PXE and DHCP Problems](#1743-pxe-and-dhcp-problems)
* [17.44 Node Provisioning Failure](#1744-node-provisioning-failure)
* [17.45 Cluster-Wide Failure](#1745-cluster-wide-failure)
* [17.46 Recent Change Analysis](#1746-recent-change-analysis)
* [17.47 Incident Handling](#1747-incident-handling)
* [17.48 Root Cause Analysis](#1748-root-cause-analysis)
* [17.49 Production Recovery Checklist](#1749-production-recovery-checklist)
* [17.50 Quick Revision](#1750-quick-revision)

## Part 5 – Advanced HPC-AI Troubleshooting & Final Checklist

> **Notebook focus:** Concise troubleshooting patterns for distributed workloads, performance degradation, cross-layer failures, and production recovery.

* [17.51 Performance Degradation](#1751-performance-degradation)
* [17.52 Slow HPC Job](#1752-slow-hpc-job)
* [17.53 MPI Job Failure](#1753-mpi-job-failure)
* [17.54 GPU Performance Problem](#1754-gpu-performance-problem)
* [17.55 Network Performance Problem](#1755-network-performance-problem)
* [17.56 Lustre Performance Problem](#1756-lustre-performance-problem)
* [17.57 Multi-Node Failure Correlation](#1757-multi-node-failure-correlation)
* [17.58 Configuration Drift](#1758-configuration-drift)
* [17.59 Safe Production Changes](#1759-safe-production-changes)
* [17.60 Final Troubleshooting Checklist](#1760-final-troubleshooting-checklist)
* [17.61 Chapter 17 Quick Revision](#1761-chapter-17-quick-revision)

