## Part 1 – Foundations of High Performance Computing

* [1.1 What is High Performance Computing?](#11-what-is-high-performance-computing)
* [1.2 Why Does HPC Exist?](#12-why-does-hpc-exist)
* [1.3 Evolution of Computing](#13-evolution-of-computing)
* [1.4 Parallel Computing](#14-parallel-computing)
* [1.5 Distributed Computing](#15-distributed-computing)
* [1.6 Vertical vs Horizontal Scaling](#16-vertical-vs-horizontal-scaling)
* [1.7 Characteristics of an HPC System](#17-characteristics-of-an-hpc-system)
* [1.8 Real-World Applications of HPC](#18-real-world-applications-of-hpc)

# 1.1 What is High Performance Computing?

## Definition

High Performance Computing (HPC) is the practice of combining many computing resources so they operate as a single logical system capable of solving computational problems that exceed the capabilities of a single computer.

Rather than making one computer extremely powerful, HPC connects many computers through a high-speed network and coordinates their work to solve large problems efficiently.

In simple terms:

> **HPC is about solving bigger problems faster by allowing many computers to work together.**

---

## Engineering Perspective

Think of a construction project.

### Traditional Computing

One engineer attempts to build an entire bridge alone.

```mermaid
flowchart LR

    E["Engineer"]
    W["Design + Planning"]
    B["Complete Bridge"]

    E --> W
    W --> B

    classDef person fill:#E0F2FE,stroke:#0284C7,color:#082F49,stroke-width:2px
    classDef work fill:#FFF7ED,stroke:#EA580C,color:#431407,stroke-width:2px
    classDef result fill:#DCFCE7,stroke:#16A34A,color:#14532D,stroke-width:2px

    class E person
    class W work
    class B result
```

The work is sequential and slow.

---

### HPC

A complete engineering team works simultaneously.

```mermaid
flowchart TB

    subgraph TEAM["Engineering Team"]
        A["Architect"]
        S["Structural"]
        C["Civil"]
        EL["Electrical"]
        M["Mechanical"]
    end

    BRIDGE["Bridge<br/>Completed"]

    A --> BRIDGE
    S --> BRIDGE
    C --> BRIDGE
    EL --> BRIDGE
    M --> BRIDGE

    classDef team fill:#E8F1FF,stroke:#2563EB,color:#172554,stroke-width:2px
    classDef result fill:#DCFCE7,stroke:#16A34A,color:#14532D,stroke-width:2px

    class A,S,C,EL,M team
    class BRIDGE result
```

Every team works in parallel, dramatically reducing the overall completion time.

An HPC cluster follows the same philosophy.

---

## The Mental Model

An HPC cluster should not be viewed as hundreds of independent servers.

Instead, think of it as **one giant computer** built from many smaller computers.

```mermaid
flowchart TB

    subgraph HPC["HPC Cluster — One Computing Platform"]
        direction LR

        N1["Node 01"]
        N2["Node 02"]
        N3["Node 03"]
        N4["Node N"]

        FABRIC["High-Speed<br/>Interconnect"]

        N1 <--> FABRIC
        N2 <--> FABRIC
        N3 <--> FABRIC
        N4 <--> FABRIC
    end

    PLATFORM["Logical HPC<br/>Computing Platform"]

    FABRIC --> PLATFORM

    classDef node fill:#EDE9FE,stroke:#7C3AED,color:#2E1065,stroke-width:2px
    classDef fabric fill:#FCE7F3,stroke:#DB2777,color:#500724,stroke-width:2px
    classDef platform fill:#DCFCE7,stroke:#16A34A,color:#14532D,stroke-width:2px

    class N1,N2,N3,N4 node
    class FABRIC fabric
    class PLATFORM platform
```

Users typically interact with the cluster as though it were a single system, while the scheduler and infrastructure distribute work across many compute nodes.

---

# 1.2 Why Does HPC Exist?

## The Problem

As scientific and engineering challenges grew more complex, a single server was no longer sufficient.

Consider the following examples:

* Simulating global weather patterns
* Training a large language model (LLM)
* Genome sequencing
* Drug discovery
* Computational Fluid Dynamics (CFD)
* Earthquake simulation
* Financial risk analysis

These workloads require:

* Trillions of calculations
* Terabytes of memory
* Massive datasets
* Fast communication between processors

A single machine eventually reaches physical and economic limits.

---

## The Solution

Instead of building one infinitely large server, engineers connect many servers into a cluster.

```mermaid
flowchart LR

    subgraph RESOURCES["Distributed Resources"]
        S1["Server 01"]
        S2["Server 02"]
        S3["Server 03"]
        S4["Server 04"]
    end

    FABRIC["High-Speed<br/>Interconnect"]

    CLUSTER["Unified<br/>HPC Cluster"]

    S1 <--> FABRIC
    S2 <--> FABRIC
    S3 <--> FABRIC
    S4 <--> FABRIC

    FABRIC --> CLUSTER

    classDef server fill:#E8F1FF,stroke:#2563EB,color:#172554,stroke-width:2px
    classDef fabric fill:#FCE7F3,stroke:#DB2777,color:#500724,stroke-width:2px
    classDef cluster fill:#DCFCE7,stroke:#16A34A,color:#14532D,stroke-width:2px

    class S1,S2,S3,S4 server
    class FABRIC fabric
    class CLUSTER cluster
```

Each server contributes CPU cores, memory, storage, and network bandwidth to the overall platform.

This approach provides:

* Higher performance
* Better scalability
* Fault isolation
* More efficient resource utilization

---

# 1.3 Evolution of Computing

Computing has evolved in response to increasing computational demands.

```mermaid
flowchart LR

    PC["Personal<br/>Computer"]
    DS["Department<br/>Server"]
    MC["Multi-Core<br/>Server"]
    CC["Cluster<br/>Computing"]
    HPC["High Performance<br/>Computing"]
    GPU["GPU-Accelerated<br/>HPC"]
    AI["AI<br/>Infrastructure"]

    PC --> DS
    DS --> MC
    MC --> CC
    CC --> HPC
    HPC --> GPU
    GPU --> AI

    classDef early fill:#E0F2FE,stroke:#0284C7,color:#082F49,stroke-width:2px
    classDef scale fill:#E8F1FF,stroke:#2563EB,color:#172554,stroke-width:2px
    classDef hpc fill:#EDE9FE,stroke:#7C3AED,color:#2E1065,stroke-width:2px
    classDef gpu fill:#FEF3C7,stroke:#D97706,color:#451A03,stroke-width:2px
    classDef ai fill:#DCFCE7,stroke:#16A34A,color:#14532D,stroke-width:2px

    class PC,DS early
    class MC,CC scale
    class HPC hpc
    class GPU gpu
    class AI ai
```

Each stage represents a shift toward greater parallelism and larger-scale resource sharing.

---

## Why the Shift Happened

As processors became faster, simply increasing clock speeds was no longer practical due to power consumption and heat dissipation.

The industry responded by:

* Adding more CPU cores.
* Deploying multiple servers.
* Introducing high-speed interconnects.
* Leveraging GPUs for massively parallel workloads.

Modern AI infrastructure is the latest stage of this evolution.

---

# 1.4 Parallel Computing

## Definition

Parallel computing divides a large problem into multiple smaller tasks that execute simultaneously.

Instead of processing one task after another:

```mermaid
flowchart LR

    A["Task A"]
    B["Task B"]
    C["Task C"]
    D["Task D"]

    A --> B --> C --> D

    classDef task fill:#F1F5F9,stroke:#64748B,color:#0F172A,stroke-width:2px
    class A,B,C,D task
```

the tasks execute concurrently:

```mermaid
flowchart TB

    PROBLEM["Large Problem"]

    subgraph TASKS["Independent / Coordinated Tasks"]
        A["Task A"]
        B["Task B"]
        C["Task C"]
        D["Task D"]
    end

    RESULT["Combined Result"]

    PROBLEM --> A
    PROBLEM --> B
    PROBLEM --> C
    PROBLEM --> D

    A --> RESULT
    B --> RESULT
    C --> RESULT
    D --> RESULT

    classDef problem fill:#E0F2FE,stroke:#0284C7,color:#082F49,stroke-width:2px
    classDef task fill:#EDE9FE,stroke:#7C3AED,color:#2E1065,stroke-width:2px
    classDef result fill:#DCFCE7,stroke:#16A34A,color:#14532D,stroke-width:2px

    class PROBLEM problem
    class A,B,C,D task
    class RESULT result
```

This significantly reduces execution time when the problem can be decomposed into independent or coordinated subtasks.

---

## Everyday Example

Imagine grading 1,000 exam papers.

### Sequential

```mermaid
flowchart LR

    T["Teacher"]
    P["1,000 Papers"]
    TIME["~10 Hours"]

    T --> P --> TIME

    classDef person fill:#E0F2FE,stroke:#0284C7,color:#082F49,stroke-width:2px
    classDef work fill:#FFF7ED,stroke:#EA580C,color:#431407,stroke-width:2px
    classDef time fill:#FDECEC,stroke:#DC2626,color:#450A0A,stroke-width:2px

    class T person
    class P work
    class TIME time
```

### Parallel

```mermaid
flowchart TB

    PAPERS["1,000 Papers"]

    subgraph TEACHERS["Parallel Work"]
        T1["Teacher 01<br/>100 Papers"]
        T2["Teacher 02<br/>100 Papers"]
        T3["Teacher 03<br/>100 Papers"]
        T4["Teacher 04<br/>100 Papers"]
        T5["Teacher 05<br/>100 Papers"]
        T6["Teacher 06<br/>100 Papers"]
        T7["Teacher 07<br/>100 Papers"]
        T8["Teacher 08<br/>100 Papers"]
        T9["Teacher 09<br/>100 Papers"]
        T10["Teacher 10<br/>100 Papers"]
    end

    RESULT["~1 Hour"]

    PAPERS --> T1
    PAPERS --> T2
    PAPERS --> T3
    PAPERS --> T4
    PAPERS --> T5
    PAPERS --> T6
    PAPERS --> T7
    PAPERS --> T8
    PAPERS --> T9
    PAPERS --> T10

    T1 --> RESULT
    T2 --> RESULT
    T3 --> RESULT
    T4 --> RESULT
    T5 --> RESULT
    T6 --> RESULT
    T7 --> RESULT
    T8 --> RESULT
    T9 --> RESULT
    T10 --> RESULT

    classDef input fill:#E0F2FE,stroke:#0284C7,color:#082F49,stroke-width:2px
    classDef worker fill:#EDE9FE,stroke:#7C3AED,color:#2E1065,stroke-width:2px
    classDef result fill:#DCFCE7,stroke:#16A34A,color:#14532D,stroke-width:2px

    class PAPERS input
    class T1,T2,T3,T4,T5,T6,T7,T8,T9,T10 worker
    class RESULT result
```

The total work is unchanged, but the completion time is dramatically reduced.

---

# 1.5 Distributed Computing

Parallel computing answers the question:

> **How can multiple tasks run at the same time?**

Distributed computing answers:

> **Where do those tasks run?**

In a distributed system, tasks execute across multiple physical computers connected by a network.

```mermaid
flowchart TB

    subgraph CLUSTER["Distributed Computing Environment"]
        N1["Node 1"]
        N2["Node 2"]
        N3["Node 3"]

        N1 --- N2
        N2 --- N3
        N1 --- N3
    end

    NET["High-Speed<br/>Network"]

    N1 --- NET
    N2 --- NET
    N3 --- NET

    classDef node fill:#EDE9FE,stroke:#7C3AED,color:#2E1065,stroke-width:2px
    classDef network fill:#FCE7F3,stroke:#DB2777,color:#500724,stroke-width:2px

    class N1,N2,N3 node
    class NET network
```

Each node performs part of the overall computation while coordinating with the others.

---

## Parallel vs Distributed

| Parallel Computing                        | Distributed Computing                           |
| ----------------------------------------- | ----------------------------------------------- |
| Focuses on executing tasks simultaneously | Focuses on using multiple physical machines     |
| May occur on one machine or many          | Always spans multiple machines                  |
| Reduces execution time                    | Increases scalability and resource availability |

Modern HPC combines **both** approaches.

---

# 1.6 Vertical vs Horizontal Scaling

## Vertical Scaling

Increase the capacity of a single server.

```mermaid
flowchart TB

    subgraph SERVER["Single Server"]
        CPU["More CPU"]
        RAM["More RAM"]
        STORAGE["More Storage"]
    end

    CAPACITY["Greater<br/>Single-Node Capacity"]

    CPU --> CAPACITY
    RAM --> CAPACITY
    STORAGE --> CAPACITY

    classDef resource fill:#EDE9FE,stroke:#7C3AED,color:#2E1065,stroke-width:2px
    classDef result fill:#DCFCE7,stroke:#16A34A,color:#14532D,stroke-width:2px

    class CPU,RAM,STORAGE resource
    class CAPACITY result
```

Advantages:

* Simpler administration.
* No application redesign required.

Limitations:

* Hardware limits.
* High cost.
* Single point of failure.

---

## Horizontal Scaling

Add additional servers.

```mermaid
flowchart TB

    subgraph CLUSTER["Horizontal Scaling"]
        N1["Node 01"]
        N2["Node 02"]
        N3["Node 03"]
        N4["Node 04"]

        FABRIC["Cluster<br/>Interconnect"]

        N1 <--> FABRIC
        N2 <--> FABRIC
        N3 <--> FABRIC
        N4 <--> FABRIC
    end

    CAPACITY["Aggregate<br/>Cluster Capacity"]

    FABRIC --> CAPACITY

    classDef node fill:#E8F1FF,stroke:#2563EB,color:#172554,stroke-width:2px
    classDef fabric fill:#FCE7F3,stroke:#DB2777,color:#500724,stroke-width:2px
    classDef result fill:#DCFCE7,stroke:#16A34A,color:#14532D,stroke-width:2px

    class N1,N2,N3,N4 node
    class FABRIC fabric
    class CAPACITY result
```

Advantages:

* Better scalability.
* Incremental growth.
* Improved resilience.
* Higher aggregate performance.

This is the architectural principle behind HPC clusters.

---

# 1.7 Characteristics of an HPC System

A modern HPC system typically exhibits the following characteristics:

| Characteristic         | Description                                     |
| ---------------------- | ----------------------------------------------- |
| Parallel Processing    | Many tasks execute simultaneously.              |
| Distributed Resources  | Compute is spread across multiple nodes.        |
| Low-Latency Network    | Fast communication between nodes.               |
| Shared Storage         | Data is accessible to all compute nodes.        |
| Centralized Scheduling | Jobs are managed by a workload manager.         |
| Scalability            | New nodes can be added with minimal disruption. |
| High Availability      | Critical services minimize downtime.            |
| Automation             | Provisioning and management are automated.      |

---

# 1.8 Real-World Applications of HPC

HPC supports a wide range of industries.

| Industry                | Example Workloads                                                |
| ----------------------- | ---------------------------------------------------------------- |
| Scientific Research     | Climate modeling, astrophysics, molecular dynamics               |
| Engineering             | CFD, finite element analysis, crash simulation                   |
| Healthcare              | Genomics, protein folding, drug discovery                        |
| Finance                 | Monte Carlo simulations, fraud detection, portfolio optimization |
| Energy                  | Reservoir simulation, seismic imaging                            |
| Manufacturing           | Digital twins, product design optimization                       |
| Artificial Intelligence | Deep learning, LLM training, recommendation systems              |

The rapid growth of AI has significantly increased the demand for GPU-accelerated HPC infrastructure.

---

# Production Insight

> HPC is not simply "many servers connected together." It is an integrated ecosystem in which compute, networking, storage, scheduling, authentication, and monitoring work together. A failure in any one layer can impact the entire workload. Successful infrastructure engineers understand these dependencies rather than viewing each technology in isolation.

---

# Best Practices

* Focus on understanding concepts before memorizing commands.
* Think in terms of systems rather than individual machines.
* Always consider scalability when designing infrastructure.
* Remember that network latency is often as important as raw CPU performance.
* Develop a habit of documenting architectural decisions and operational procedures.

---

# Interview Questions

1. What is High Performance Computing, and how would you explain it to a non-technical audience?
2. Why can't organizations simply purchase larger servers instead of building HPC clusters?
3. Explain the difference between parallel computing and distributed computing.
4. What are the primary characteristics of an HPC system?
5. Why is horizontal scaling preferred in HPC environments?
6. Name five real-world industries that rely heavily on HPC.

---

# Summary

In this part, we established the foundation for understanding High Performance Computing. We explored why HPC exists, how it evolved from traditional computing, and the core principles of parallelism, distributed systems, and horizontal scaling. These concepts form the basis for modern AI infrastructure and will be referenced throughout the remainder of this handbook.

---

## Next Part

**Part 2 – Evolution to AI Infrastructure**

Topics:

* HPC vs Cloud Computing
* HPC vs AI Infrastructure
* AI Factory Concept
* Evolution from CPU Clusters to GPU Clusters
* Modern AI Workloads
* Why NVIDIA Changed HPC Forever
* The Rise of Foundation Models
