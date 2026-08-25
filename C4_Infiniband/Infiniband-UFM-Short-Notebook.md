# InfiniBand UFM — Short Engineering Notebook

**Version:** v1.0.0  
**Scope:** NVIDIA Unified Fabric Manager (UFM) Enterprise  
**Audience:** HPC / AI Infrastructure Engineers  
**Status:** Verified against NVIDIA documentation available in August 2026

> **Accuracy note:** Exact features, commands, configuration paths, and behavior can vary by UFM release. Always use the documentation matching the installed UFM version.

## 1. What is UFM?

**NVIDIA Unified Fabric Manager (UFM)** is a host-based solution for managing InfiniBand fabrics. It provides fabric-wide visibility and management functions including subnet management, topology awareness, device management, performance monitoring, events, and APIs. NVIDIA describes the UFM Server as having complete visibility over the managed fabric and managing routing on fabric devices; a secondary UFM server can be deployed for HA. [NVIDIA UFM Overview](https://docs.nvidia.com/networking/display/nvidia-ufm-enterprise-user-manual-v6-20-1.1.pdf)

```text
                     +------------------+
                     |   UFM Server     |
                     +--------+---------+
                              |
        +---------------------+---------------------+
        |                     |                     |
        v                     v                     v
 Subnet Manager       Performance Manager     Device Manager
   (OpenSM)                   |                     |
        |                     |                     |
        +---------------------+---------------------+
                              |
                              v
                       InfiniBand Fabric
                  +------+------+------+------+
                  |      |             |      |
                 HCA   Switch         Switch  HCA
                  |                     |
               Compute               Compute
```

## 2. Why UFM Matters in HPC / AI

Large InfiniBand GPU/HPC fabrics need centralized visibility and operations. Typical questions are:

- Is the fabric healthy?
- Which ports are down?
- Is a link flapping?
- Where is congestion occurring?
- Which ports have errors?
- Is the Subnet Manager operational?
- What changed before an incident?

NVIDIA positions UFM for high-performance GPU-to-GPU InfiniBand environments, including health monitoring, congestion detection, tenant isolation, and issue resolution. [NVIDIA IaaS documentation](https://docs.nvidia.com/dsx/ncp/part-2-software-components/nvidia-software-for-infrastructure-as-a-service)

## 3. Core UFM Components

```text
UFM
 |
 +-- Subnet Manager (SM)
 |      +-- OpenSM
 |      +-- Fabric bring-up
 |      +-- Routing
 |
 +-- Performance Manager
 |      +-- Performance data
 |
 +-- Device Manager
 |      +-- Device management
 |
 +-- UFM Switch Agent
 |      +-- Switch discovery / management
 |
 +-- Events / Telemetry / APIs
```

NVIDIA documents that UFM Enterprise uses the community **OpenSM Subnet Manager**. The SM is the InfiniBand routing engine responsible for fabric bring-up and routing management. UFM also provides a plugin API for runtime management and fabric-data export. [NVIDIA UFM User Manual](https://docs.nvidia.com/networking/display/nvidia-ufm-enterprise-user-manual-v6-15-6-4.6-4.pdf)

The Performance Manager collects performance data from managed fabric devices for fabric-wide analysis. Device Manager provides common management operations across devices, including SSH/native CLI interaction. The UFM Switch Agent is integrated into NVIDIA switch software for discovery and management functions. [NVIDIA UFM User Manual](https://docs.nvidia.com/networking/display/nvidia-ufm-enterprise-user-manual-v6-15-6-4.6-4.pdf)

## 4. UFM vs OpenSM

This is an important distinction:

```text
UFM
 |
 +---- Management / visibility / operations
 |
 +---- Subnet Manager
          |
          +---- OpenSM
                  |
                  +---- Fabric bring-up
                  +---- Routing
```

**UFM is broader than the Subnet Manager.**

- **OpenSM:** Subnet Manager function.
- **UFM:** Broader fabric management, visibility, monitoring, device management, events, telemetry, APIs, and integrations.

NVIDIA explicitly documents OpenSM as the Subnet Manager used by UFM Enterprise. [NVIDIA UFM User Manual](https://docs.nvidia.com/networking/display/nvidia-ufm-enterprise-user-manual-v6-15-6-4.6-4.pdf)

## 5. Important InfiniBand Concepts

| Term | Meaning |
|---|---|
| HCA | Host Channel Adapter; host-side InfiniBand adapter |
| SM | Subnet Manager |
| LID | Local Identifier within an InfiniBand subnet |
| GUID | Globally Unique Identifier for IB components/ports |
| P_Key | Partition key used for InfiniBand partition membership |
| M_Key | Management key used for management protection |
| SA | Subnet Administrator |
| VL | Virtual Lane used for traffic/QoS handling |
| UFM | Unified Fabric Manager |
| OpenSM | InfiniBand Subnet Manager used by UFM Enterprise |

## 6. UFM Configuration Paths

Do **not** copy paths blindly between releases.

NVIDIA documentation lists OpenSM configuration files under:

```text
/opt/ufm/files/conf/opensm/
```

Examples include:

```text
opensm.conf
ib-node-name-map
partitions.conf
qos-policy.conf
prefix-routes.conf
```

NVIDIA also documents newer primary configuration files such as:

```text
/opt/ufm/files/conf/large_scale_subnet.cfg
/opt/ufm/files/conf/small_scale_subnet.cfg
```

The exact configuration mechanism depends on the UFM release. [NVIDIA UFM documentation](https://docs.nvidia.com/nvidia-ufm-enterprise-user-manual-v6-15-1.pdf) [NVIDIA UFM v6.19 documentation](https://docs.nvidia.com/networking/display/nvidia-ufm-enterprise-user-manual-v6-19-1.1.pdf)

## 7. UFM High Availability

Conceptually:

```text
                 +------------------+
                 | Primary UFM      |
                 | Server           |
                 +--------+---------+
                          |
                    Fabric Management
                          |
                 +--------+---------+
                 | Secondary UFM    |
                 | HA Server        |
                 +------------------+
```

NVIDIA describes the UFM HA Server as a UFM-installed secondary server for High Availability deployment. Exact requirements and behavior depend on the deployed release. [NVIDIA UFM Overview](https://docs.nvidia.com/networking/display/nvidia-ufm-enterprise-user-manual-v6-20-1.1.pdf)

## 8. InfiniBand Partitioning

InfiniBand partitioning uses **P_Keys**.

```text
                 UFM / Subnet Manager
                         |
              +----------+----------+
              |                     |
           P_Key A                P_Key B
              |                     |
          Node A/ B              Node C/ D
```

NVIDIA's current Infra Controller documentation states that P_Key membership is enforced by the InfiniBand subnet manager and provides fabric-side isolation. Hosts that are not members of a partition cannot exchange IB traffic with members of that partition. [NVIDIA InfiniBand Partitioning](https://docs.nvidia.com/infra-controller/documentation/configuration-day-1/infini-band/infini-band-partitioning)

## 9. UFM Telemetry and REST API

UFM provides REST APIs for management and telemetry. NVIDIA's REST API documentation includes telemetry endpoints for top-X sessions and historical device/port metrics. [NVIDIA UFM REST API Guide](https://docs.nvidia.com/nvidia-ufm-enterprise-rest-api-guide-v6-16-0.pdf)

Operational model:

```text
IB Devices
    |
    v
Performance / Telemetry
    |
    v
   UFM
    |
    +---- Web UI
    +---- REST API
    +---- Automation / Monitoring
```

Use the REST API guide corresponding to the installed UFM release.

## 10. Host-Side InfiniBand Commands

These are **host-side InfiniBand/RDMA tools**, not UFM commands.

Check devices:

```bash
ibv_devices
```

Device information:

```bash
ibv_devinfo
```

Port/link state:

```bash
ibstat
```

Fabric topology:

```bash
ibnetdiscover
```

Fabric health:

```bash
ibcheckstate
```

Port counters:

```bash
perfquery
```

RDMA links:

```bash
rdma link
```

Network interfaces:

```bash
ip -br link
```

Driver information:

```bash
ethtool -i <interface>
```

Availability depends on the installed RDMA / NVIDIA networking software stack.

## 11. Practical Troubleshooting Workflow

```text
Application slowdown
       |
       v
Check host / GPU health
       |
       v
Check HCA and port state
       |
       v
Check UFM topology
       |
       v
Check switch / port errors
       |
       v
Check congestion / counters
       |
       v
Check routing / SM
       |
       v
Check cable / optics / physical layer
       |
       v
Validate with MPI / NCCL / RDMA benchmark
```

Do not restart the Subnet Manager as the first response. Collect evidence and identify the failing layer.

## 12. Troubleshooting Decision Tree

```text
IB Problem
    |
    +-- HCA visible?
    |      |
    |      +-- NO --> Driver / PCIe / firmware / OS
    |      |
    |      +-- YES
    |
    +-- Port ACTIVE?
    |      |
    |      +-- NO --> Cable / switch port / SM / link
    |      |
    |      +-- YES
    |
    +-- Expected topology?
    |      |
    |      +-- NO --> Fabric / cabling / discovery
    |      |
    |      +-- YES
    |
    +-- Errors increasing?
    |      |
    |      +-- YES --> Physical/link/fabric investigation
    |      |
    |      +-- NO
    |
    +-- Congestion?
           |
           +-- YES --> Traffic / QoS / routing investigation
           |
           +-- NO --> Continue toward application layer
```

## 13. Production Incident Patterns

### Port Down

Investigate in layers:

```text
Host HCA
  ↓
PCIe / Driver / Firmware
  ↓
Cable / Transceiver
  ↓
Switch Port
  ↓
Subnet Manager / Fabric
  ↓
UFM Events / Topology
```

### Link Flapping

Check:

- Cable/transceiver
- Switch port
- HCA
- Firmware compatibility
- Error counters
- Event history
- Timestamps

### Poor GPU Communication

Follow the end-to-end path:

```text
GPU
 ↓
PCIe
 ↓
HCA
 ↓
IB Link
 ↓
Switch
 ↓
IB Fabric
 ↓
Remote HCA
 ↓
Remote GPU
```

After fabric health is established, validate NCCL/MPI/RDMA at the application layer.

## 14. Interview Questions

### Q1. What is UFM?

UFM is NVIDIA's fabric-management solution for InfiniBand environments. It provides centralized fabric visibility and management, including subnet management through OpenSM, performance monitoring, device management, and operational interfaces.

### Q2. Is UFM the same as OpenSM?

No. OpenSM is the Subnet Manager. UFM Enterprise uses OpenSM as its Subnet Manager and adds broader fabric-management capabilities.

### Q3. What does the Subnet Manager do?

It performs InfiniBand subnet bring-up and routing management.

### Q4. Why deploy UFM HA?

To provide a secondary UFM server for high availability of fabric-management services. Exact HA requirements depend on the UFM release and deployment architecture.

### Q5. How would you troubleshoot an IB port that is down?

1. Verify HCA visibility.
2. Check port state.
3. Check PCIe, driver and firmware.
4. Check cable/transceiver.
5. Check switch port.
6. Check UFM topology/events.
7. Check Subnet Manager state.
8. Validate the end-to-end path.

### Q6. What is P_Key?

P_Key is the InfiniBand partition key used to define partition membership and provide fabric-level traffic isolation.

### Q7. What is M_Key?

M_Key is a management key used for InfiniBand management protection. Exact security behavior should be evaluated against the UFM/OpenSM configuration for the deployed version.

## 15. Senior Engineer Mental Model

```text
                Application
                    |
             MPI / NCCL / RDMA
                    |
                  HCA
                    |
              IB Port/Link
                    |
                 Cable
                    |
                Switch
                    |
                 Fabric
                    |
            Subnet Manager
                    |
                   UFM
                    |
          Monitoring / API
```

Each layer can fail independently. The goal is to identify **which layer is actually broken** before changing the system.

## 16. Quick Revision

```text
UFM
 |
 +-- Fabric Management
 |
 +-- OpenSM / Subnet Manager
 |       +-- Bring-up
 |       +-- Routing
 |
 +-- Performance Manager
 +-- Device Manager
 +-- Switch Agent
 +-- Events / Telemetry
 +-- REST APIs
 +-- HA
```

Remember:

```text
UFM != OpenSM

UFM uses OpenSM as the Subnet Manager.
```

## 17. Production Checklist

- [ ] UFM service health
- [ ] Subnet Manager health
- [ ] Fabric topology
- [ ] Expected switch/device inventory
- [ ] HCA visibility
- [ ] Port states
- [ ] Error counters
- [ ] Congestion indicators
- [ ] Relevant events
- [ ] Routing
- [ ] Partition/P_Key configuration where applicable
- [ ] Application-level validation (MPI/NCCL/RDMA)

## 18. Key Takeaways

1. UFM is a centralized InfiniBand fabric-management solution.
2. UFM Enterprise uses OpenSM as its Subnet Manager.
3. The Subnet Manager handles fabric bring-up and routing.
4. UFM adds broader management, visibility, performance monitoring, device management, events and APIs.
5. UFM supports an HA deployment model.
6. InfiniBand troubleshooting should be performed layer-by-layer.
7. P_Keys provide InfiniBand partition-based isolation.
8. Exact commands, paths and supported features must be checked against the installed UFM release.

## 19. Official References

- NVIDIA UFM Enterprise User Manual — UFM overview and architecture.
- NVIDIA UFM Enterprise User Manual — Subnet Manager and UFM components.
- NVIDIA UFM Enterprise REST API Guide — telemetry and API examples.
- NVIDIA InfiniBand Setup Runbook — UFM/OpenSM configuration and security.
- NVIDIA InfiniBand Partitioning — P_Key and partition isolation.

For production deployments, always use the documentation matching the exact installed UFM Enterprise release.
