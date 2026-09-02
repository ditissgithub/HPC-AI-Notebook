# NVIDIA GPU MIG (Multi-Instance GPU) — Short Note

## 1. What is MIG?

**MIG (Multi-Instance GPU)** is an NVIDIA GPU virtualization/partitioning technology available on supported GPUs such as NVIDIA A100, H100, H200 and newer architectures.

MIG divides one physical GPU into multiple **isolated GPU Instances (GI)**. Each GPU Instance can then contain one or more **Compute Instances (CI)**.

The important idea is:

```text
Physical NVIDIA GPU
        |
        +-- GPU Instance (GI)
        |      |
        |      +-- Compute Instance (CI)
        |
        +-- GPU Instance (GI)
               |
               +-- Compute Instance (CI)
```

MIG provides dedicated portions of GPU resources such as GPU compute and memory to different workloads. This is useful when multiple users/jobs need to share a GPU while maintaining stronger resource isolation than ordinary time-sharing.

> **Note:** Exact MIG profiles and available partitions depend on the GPU model and driver/software version.

---

## 2. GPU Instance vs Compute Instance

### GPU Instance (GI)

A **GPU Instance** is the primary hardware resource partition.

It defines a portion of the physical GPU's resources, including GPU memory and GPU compute resources.

### Compute Instance (CI)

A **Compute Instance** is created inside a GPU Instance.

It represents a schedulable compute partition within that GPU Instance.

For most operational workflows:

```text
GPU
 |
 +-- GI
      |
      +-- CI
```

You normally create the **GPU Instance first**, then create the **Compute Instance** inside it.

---

## 3. Common MIG Profiles

MIG profile names vary by GPU generation. For example, on an **A100 80GB**, commonly available GPU-instance profiles include:

| Profile | Approx. GPU Memory | Approx. GPU Compute |
|---|---:|---:|
| `1g.10gb` | 10 GB | 1/7 GPU |
| `2g.20gb` | 20 GB | 2/7 GPU |
| `3g.40gb` | 40 GB | 3/7 GPU |
| `4g.40gb` | 40 GB | 4/7 GPU |
| `7g.80gb` | 80 GB | Full GPU |

Other GPU generations have different profiles. **Do not assume these profiles exist on every MIG-capable GPU. Always query the actual GPU.**

---

# 4. Check GPU and MIG Capability

Check the GPU:

```bash
nvidia-smi
```

Check MIG mode:

```bash
nvidia-smi -i 0 --query-gpu=name,mig.mode.current,mig.mode.pending --format=csv
```

Enable MIG mode if required:

```bash
nvidia-smi -i 0 -mig 1
```

Disable MIG mode:

```bash
nvidia-smi -i 0 -mig 0
```

Depending on the driver/GPU state, changing MIG mode may require a GPU reset or reboot.

---

# 5. List Supported GPU Instance Profiles

List all supported GPU Instance profiles:

```bash
nvidia-smi mig -lgip
```

For a specific GPU:

```bash
nvidia-smi mig -i 0 -lgip
```

This is the preferred way to determine which MIG profiles are actually supported on the installed GPU.

---

# 6. Check Possible GPU Instance Placements

```bash
nvidia-smi mig -i 0 -lgipp
```

This shows possible placements for GPU Instance profiles.

Example format:

```text
{Start}:{Size}
```

Placement matters because MIG resources are arranged in hardware slices. Some combinations of MIG instances are possible only when their slices can be placed together.

---

# 7. Create a GPU Instance

Example:

```bash
nvidia-smi mig -i 0 -cgi 1g.10gb
```

This creates a `1g.10gb` GPU Instance on GPU 0.

Multiple profiles can be specified when a valid placement exists:

```bash
nvidia-smi mig -i 0 -cgi 3g.40gb,2g.20gb
```

**Important:** The exact combination must be supported by the GPU's placement rules.

---

# 8. Create a Compute Instance

After creating the GPU Instance, list it:

```bash
nvidia-smi mig -i 0 -lgi
```

Then create a Compute Instance:

```bash
nvidia-smi mig -i 0 -gi <GPU_INSTANCE_ID> -cci
```

For example:

```bash
nvidia-smi mig -i 0 -gi 1 -cci
```

The default Compute Instance profile is used when no CI profile is explicitly specified.

You can also inspect supported CI profiles:

```bash
nvidia-smi mig -i 0 -gi 1 -lcip
```

---

# 9. Verify MIG Configuration

List GPU Instances:

```bash
nvidia-smi mig -lgi
```

List Compute Instances:

```bash
nvidia-smi mig -lci
```

A general check:

```bash
nvidia-smi
```

You should see MIG devices/resources exposed according to the current configuration.

---

# 10. Destroy a Compute Instance

A Compute Instance should normally be removed before its parent GPU Instance.

List CIs first:

```bash
nvidia-smi mig -i 0 -gi <GPU_INSTANCE_ID> -lci
```

Destroy a specific CI:

```bash
nvidia-smi mig -i 0 -gi <GPU_INSTANCE_ID> -ci <COMPUTE_INSTANCE_ID> -dci
```

Example:

```bash
nvidia-smi mig -i 0 -gi 1 -ci 0 -dci
```

---

# 11. Destroy a GPU Instance

After its Compute Instances have been removed:

```bash
nvidia-smi mig -i 0 -gi <GPU_INSTANCE_ID> -dgi
```

Example:

```bash
nvidia-smi mig -i 0 -gi 1 -dgi
```

You can also destroy GPU Instances associated with a GPU using the appropriate `-i` and `-gi` options.

---

# 12. Complete MIG Lifecycle

The basic operational workflow is:

```text
Physical GPU
     |
     v
Enable MIG Mode
     |
     v
List GPU Instance Profiles
     |
     v
Check Possible Placements
     |
     v
Create GPU Instance (GI)
     |
     v
List GPU Instances
     |
     v
Create Compute Instance (CI)
     |
     v
Workload uses MIG device
     |
     v
Destroy Compute Instance
     |
     v
Destroy GPU Instance
     |
     v
(Optional) Disable MIG Mode
```

---

# 13. Important `nvidia-smi mig` Commands

| Purpose | Command |
|---|---|
| Help | `nvidia-smi mig -h` |
| List GPU profiles | `nvidia-smi mig -lgip` |
| List GPU placements | `nvidia-smi mig -lgipp` |
| Create GPU Instance | `nvidia-smi mig -cgi <profile>` |
| List GPU Instances | `nvidia-smi mig -lgi` |
| Destroy GPU Instance | `nvidia-smi mig -dgi` |
| List CI profiles | `nvidia-smi mig -lcip` |
| List CI placements | `nvidia-smi mig -lcipp` |
| Create Compute Instance | `nvidia-smi mig -cci` |
| List Compute Instances | `nvidia-smi mig -lci` |
| Destroy Compute Instance | `nvidia-smi mig -dci` |

---

# 14. MIG in an HPC / Kubernetes Environment

In a production HPC-AI environment, MIG is normally not managed manually for every job.

A typical architecture is:

```text
                 User Job
                    |
                    v
             Scheduler / Control Plane
                    |
          +---------+---------+
          |                   |
        Slurm             Kubernetes
          |                   |
          +---------+---------+
                    |
                    v
             NVIDIA GPU Node
                    |
                    v
             NVIDIA Driver
                    |
                    v
              MIG Configuration
                    |
          +---------+---------+
          |                   |
       MIG Device 0        MIG Device 1
       1g.10gb             1g.10gb
          |                   |
          v                   v
       Job / Pod            Job / Pod
```

For Kubernetes, the NVIDIA device plugin / GPU Operator can expose MIG resources to workloads.

For Slurm, MIG resources can be integrated with GRES/resource scheduling so that jobs request specific GPU/MIG resources.

The important distinction is:

```text
MIG configuration
       !=
Job scheduling
       !=
GPU application
```

MIG creates the hardware-level GPU partitions. The scheduler/orchestrator decides **which workload gets which available partition**.

---

# 15. Operational Best Practices

1. **Always check the actual GPU model first.**
2. Use `nvidia-smi mig -lgip` instead of assuming profile names.
3. Check `-lgipp` before creating multiple GPU Instances.
4. Stop workloads before destroying their MIG instances.
5. Destroy **Compute Instances before GPU Instances**.
6. In production, avoid manual MIG changes while jobs are running.
7. Integrate MIG with Slurm or Kubernetes rather than managing partitions manually for every workload.
8. Keep NVIDIA driver, CUDA, GPU Operator/device-plugin, and scheduler configuration compatible.
9. Document the desired MIG layout for each node type.
10. Test MIG persistence and reboot behavior before deploying the configuration across a production cluster.

---

## Quick Reference

### Enable MIG

```bash
nvidia-smi -i 0 -mig 1
```

### Discover profiles

```bash
nvidia-smi mig -i 0 -lgip
nvidia-smi mig -i 0 -lgipp
```

### Create GI

```bash
nvidia-smi mig -i 0 -cgi 1g.10gb
```

### Create CI

```bash
nvidia-smi mig -i 0 -gi <GI_ID> -cci
```

### Verify

```bash
nvidia-smi mig -lgi
nvidia-smi mig -lci
nvidia-smi
```

### Destroy CI

```bash
nvidia-smi mig -i 0 -gi <GI_ID> -ci <CI_ID> -dci
```

### Destroy GI

```bash
nvidia-smi mig -i 0 -gi <GI_ID> -dgi
```

### Disable MIG

```bash
nvidia-smi -i 0 -mig 0
```

> **Production note:** Exact commands, profile names, placement rules, and reset requirements depend on the GPU architecture and NVIDIA driver version. Always validate commands on the target GPU with `nvidia-smi mig -h`, `-lgip`, and `-lgipp` before applying them to production nodes.
