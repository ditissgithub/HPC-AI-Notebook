# Chapter 6 – xCAT

## Part 1 – xCAT Architecture & Core Concepts

* [6.1 What is xCAT?](#61-what-is-xcat)
* [6.2 Why xCAT is Used in HPC](#62-why-xcat-is-used-in-hpc)
* [6.3 xCAT Architecture](#63-xcat-architecture)
* [6.4 xCAT Components](#64-xcat-components)
* [6.5 xCAT Database](#65-xcat-database)
* [6.6 Management Node](#66-management-node)
* [6.7 Compute Nodes](#67-compute-nodes)
* [6.8 Provisioning Flow](#68-provisioning-flow)
* [6.9 xCAT and Network Services](#69-xcat-and-network-services)
* [6.10 Stateful vs Stateless Provisioning](#610-stateful-vs-stateless-provisioning)
* [6.11 xCAT in a Production HPC Environment](#611-xcat-in-a-production-hpc-environment)
* [6.12 Important xCAT Commands](#612-important-xcat-commands)

## Part 2 – Node Definitions, Networks, Images & Discovery

* [6.16 xCAT Node Definitions](#616-xcat-node-definitions)
* [6.17 Node Groups](#617-node-groups)
* [6.18 Node Attributes](#618-node-attributes)
* [6.19 xCAT Networks](#619-xcat-networks)
* [6.20 Network Interfaces](#620-network-interfaces)
* [6.21 DHCP Configuration](#621-dhcp-configuration)
* [6.22 DNS Configuration](#622-dns-configuration)
* [6.23 OS Images](#623-os-images)
* [6.24 Boot Configuration](#624-boot-configuration)
* [6.25 Node Discovery](#625-node-discovery)
* [6.26 Hardware Discovery](#626-hardware-discovery)
* [6.27 Provisioning Commands](#627-provisioning-commands)
* [6.28 Practical xCAT Example](#628-practical-xcat-example)
* [6.29 Production Deployment Workflow](#629-production-deployment-workflow)
* [6.30 Troubleshooting Checklist](#630-troubleshooting-checklist)

## Part 3 – OS Images, Stateful/Stateless Provisioning & Post-Install Configuration

* [6.33 OS Image Management](#633-os-image-management)
* [6.34 Golden Image Concept](#634-golden-image-concept)
* [6.35 Stateful Provisioning](#635-stateful-provisioning)
* [6.36 Stateless Provisioning](#636-stateless-provisioning)
* [6.37 Stateful vs Stateless in HPC](#637-stateful-vs-stateless-in-hpc)
* [6.38 Linux Installation Workflow](#638-linux-installation-workflow)
* [6.39 Kickstart-Based Installation](#639-kickstart-based-installation)
* [6.40 Post-Install Configuration](#640-post-install-configuration)
* [6.41 xCAT + Ansible](#641-xcat--ansible)
* [6.42 xCAT + Slurm](#642-xcat--slurm)
* [6.43 GPU Node Provisioning](#643-gpu-node-provisioning)
* [6.44 InfiniBand Node Provisioning](#644-infiniband-node-provisioning)
* [6.45 Real HPC Provisioning Example](#645-real-hpc-provisioning-example)
* [6.46 Production Best Practices](#646-production-best-practices)
* [6.47 Troubleshooting Scenarios](#647-troubleshooting-scenarios)

## Part 4 – Database, Services, Node Lifecycle & Production Operations

* [6.50 xCAT Database Architecture](#650-xcat-database-architecture)
* [6.51 Important xCAT Tables](#651-important-xcat-tables)
* [6.52 Node and Network Relationships](#652-node-and-network-relationships)
* [6.53 Site and Global Configuration](#653-site-and-global-configuration)
* [6.54 xCAT Service Architecture](#654-xcat-service-architecture)
* [6.55 DHCP and DNS Generation](#655-dhcp-and-dns-generation)
* [6.56 Production xCAT Commands](#656-production-xcat-commands)
* [6.57 Node Lifecycle Management](#657-node-lifecycle-management)
* [6.58 Power Management](#658-power-management)
* [6.59 Remote Console Management](#659-remote-console-management)
* [6.60 Cluster Scaling](#660-cluster-scaling)
* [6.61 xCAT High Availability](#661-xcat-high-availability)
* [6.62 Production Scenario – Adding a Node](#662-production-scenario--adding-a-node)
* [6.63 Production Scenario – Rebuilding a Node](#663-production-scenario--rebuilding-a-node)
* [6.64 Production Scenario – Removing a Node](#664-production-scenario--removing-a-node)
* [6.65 Troubleshooting Workflow](#665-troubleshooting-workflow)

  ## Part 5 – Production Operations

* [6.69 xCAT in a Production HPC Workflow](#669-xcat-in-a-production-hpc-workflow)
* [6.70 Adding a New Compute Node](#670-adding-a-new-compute-node)
* [6.71 GPU Node Provisioning](#671-gpu-node-provisioning)
* [6.72 xCAT + Ansible](#672-xcat--ansible)
* [6.73 xCAT + Slurm](#673-xcat--slurm)
* [6.74 Configuration Drift](#674-configuration-drift)
* [6.75 Backup & Recovery](#675-backup--recovery)

## Part 6 – Troubleshooting & Quick Revision

* [6.76 xCAT Troubleshooting Flow](#676-xcat-troubleshooting-flow)
* [6.77 Essential xCAT Commands](#677-essential-xcat-commands)
* [6.78 xCAT Quick Architecture](#678-xcat-quick-architecture)
* [6.79 xCAT Interview Questions](#679-xcat-interview-questions)
* [6.80 HPC-AI Engineer Perspective](#680-hpc-ai-engineer-perspective)
* [6.81 Chapter 6 – Final Revision](#681-chapter-6--final-revision)
* [Chapter 6 Checklist](#chapter-6-checklist)
