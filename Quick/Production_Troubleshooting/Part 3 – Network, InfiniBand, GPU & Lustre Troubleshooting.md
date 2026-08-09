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


# 17.26 Network Troubleshooting

Use a layered approach:

```text
Physical Link
     ↓
Interface
     ↓
IP Address
     ↓
Route
     ↓
DNS
     ↓
Port
     ↓
Application
```

Start with:

```bash
ip -br addr
```

Then:

```bash
ip link
ip route
```

Check statistics:

```bash
ip -s link
```

---

# 17.27 Interface Down

Check:

```bash
ip link show <interface>
```

Example:

```bash
ip link show bond0
```

Check NetworkManager:

```bash
nmcli device status
```

For an Ethernet interface:

```bash
ethtool eth0
```

Look for:

```text
Link detected: yes
```

Check errors:

```bash
ethtool -S eth0
```

For bonding:

```bash
cat /proc/net/bonding/bond0
```

Check:

```text
MII Status
Slave Interface
Link Failure Count
Active Slave
```

---

# 17.28 Connectivity Problems

Test basic connectivity:

```bash
ping <gateway>
```

Then:

```bash
ping <target>
```

Check routing:

```bash
ip route
```

Test a specific path:

```bash
tracepath <target>
```

Check port:

```bash
nc -zv <host> <port>
```

Check listening services:

```bash
ss -lntup
```

### Diagnostic pattern

```text
Cannot connect
     ↓
Interface?
     ↓
IP?
     ↓
Route?
     ↓
Gateway?
     ↓
DNS?
     ↓
Port?
     ↓
Service?
```

---

# 17.29 MTU Problems

HPC networks often use large MTUs, particularly for high-performance fabrics and storage networks.

Check:

```bash
ip link show <interface>
```

Example:

```bash
ip link show ib0
```

or:

```bash
ip link show bond0
```

Test packet size:

```bash
ping -M do -s 8972 <target>
```

The exact payload depends on the interface MTU being tested.

A mismatch can cause:

```text
Packet loss
Connection failures
Poor performance
Storage issues
RDMA problems
```

Compare both endpoints:

```bash
ip link show <interface>
```

---

# 17.30 InfiniBand Troubleshooting

Start with:

```bash
ibstat
```

Expected healthy state:

```text
State: Active
Physical state: LinkUp
```

Check devices:

```bash
ibdev2netdev
```

Check RDMA:

```bash
rdma link
```

Check:

```bash
ibv_devinfo
```

---

## Port Not Active

If:

```text
State: Down
```

investigate:

```text
Cable
 ↓
Switch port
 ↓
HCA port
 ↓
Subnet Manager
 ↓
OFED/driver
```

Check OpenSM:

```bash
systemctl status opensm
```

Logs:

```bash
journalctl -u opensm
```

Check kernel:

```bash
dmesg -T | grep -Ei "mlx|ib|rdma"
```

---

# 17.31 RDMA Troubleshooting

Check:

```bash
rdma link
```

```bash
ibv_devices
```

```bash
ibv_devinfo
```

Check RDMA modules:

```bash
lsmod | grep -E "rdma|mlx"
```

Check RDMA resources:

```bash
rdma resource show
```

If supported by the environment, use `perftest` utilities:

```bash
ib_write_bw
ib_read_bw
ib_send_bw
```

### Diagnostic flow

```text
HCA detected?
     ↓
Port ACTIVE?
     ↓
RDMA device visible?
     ↓
Subnet Manager healthy?
     ↓
Peer reachable?
     ↓
Bandwidth/latency test
```

---

# 17.32 NVIDIA GPU Troubleshooting

Start with:

```bash
nvidia-smi
```

If successful, check:

```bash
nvidia-smi -q
```

Check PCIe:

```bash
lspci | grep -i nvidia
```

Check driver:

```bash
lsmod | grep nvidia
```

Check kernel messages:

```bash
dmesg -T | grep -i nvidia
```

---

## GPU Not Detected

```text
lspci
   │
   ├── GPU visible
   │       ↓
   │   Driver problem
   │
   └── GPU absent
           ↓
       PCIe / hardware / BIOS
```

Do not immediately reinstall the driver.

First determine whether the operating system can see the PCIe device.

---

# 17.33 GPU Driver Problems

Check:

```bash
nvidia-smi
```

Possible error:

```text
NVIDIA-SMI has failed because it couldn't communicate with the NVIDIA driver.
```

Investigate:

```bash
lsmod | grep nvidia
```

```bash
cat /proc/driver/nvidia/version
```

```bash
uname -r
```

Check kernel logs:

```bash
dmesg -T | grep -i nvidia
```

Potential causes:

```text
Driver not loaded
Kernel module mismatch
Kernel update
Incorrect driver version
GPU hardware problem
Module dependency issue
```

### Important principle

```text
Kernel
   ↓
NVIDIA kernel module
   ↓
NVIDIA driver
   ↓
CUDA runtime
   ↓
Application
```

A CUDA application failure does not automatically mean CUDA is the root cause.

---

# 17.34 Slurm GPU Problems

A GPU may work with:

```bash
nvidia-smi
```

but still be unavailable to Slurm.

Check:

```bash
scontrol show node <node>
```

Look for:

```text
Gres=
CfgTRES=
AllocTRES=
```

Check configuration:

```bash
grep -i gres /etc/slurm/slurm.conf
```

Check GRES configuration:

```bash
cat /etc/slurm/gres.conf
```

Then:

```bash
systemctl status slurmd
journalctl -u slurmd
```

### Diagnostic model

```text
GPU hardware
     ↓
NVIDIA driver
     ↓
nvidia-smi
     ↓
Slurm GRES
     ↓
Slurm scheduler
     ↓
GPU job
```

---

# 17.35 Lustre Troubleshooting

Start from the client.

Check mount:

```bash
mount | grep lustre
```

Check filesystem:

```bash
df -hT | grep lustre
```

Check Lustre space:

```bash
lfs df -h
```

Check client logs:

```bash
dmesg -T | grep -i lustre
```

or:

```bash
journalctl -k | grep -i lustre
```

---

## Lustre Architecture Reminder

```text
Client
  │
  ├──── MGS
  │
  ├──── MDT
  │
  └──── OST
```

Typical failures:

```text
Mount failure
Slow I/O
No space
Quota exceeded
OST unavailable
Network problem
Client error
```

---

# 17.36 Lustre Quota Problems

Check user quota:

```bash
lfs quota -h -u user01 /home
```

```bash
lfs quota -h -u user01 /scratch
```

Check group quota:

```bash
lfs quota -h -g hpcusers /scratch
```

Typical symptoms:

```text
Disk has free space
but user cannot create files
```

Possible cause:

```text
User quota exceeded
```

Check:

```bash
lfs quota -h -u user01 /scratch
```

For a quota configuration problem, investigate the Lustre filesystem/quota configuration rather than changing client-side limits blindly.

---

# 17.37 Production Diagnostic Matrix

| Symptom              | First Checks                  | Next Layer            |
| -------------------- | ----------------------------- | --------------------- |
| Node DOWN            | `scontrol show node`          | Linux/slurmd          |
| Job PENDING          | `squeue`, `scontrol show job` | Resources/QoS         |
| Job FAILED           | `sacct`, job output           | Application/node      |
| GPU missing          | `lspci`, `nvidia-smi`         | Driver/PCIe           |
| GPU job unavailable  | `scontrol show node`          | GRES                  |
| IB down              | `ibstat`                      | OFED/OpenSM/fabric    |
| RDMA failure         | `rdma link`                   | Fabric/driver         |
| Lustre mount failure | `mount`, `dmesg`              | Network/Lustre        |
| Lustre slow          | `lfs df`, I/O metrics         | OST/network           |
| Quota exceeded       | `lfs quota`                   | User/filesystem quota |
| Network failure      | `ip`, `ss`, `ping`            | Routing/DNS/service   |

---

# 17.38 Quick Revision

### Network

```bash
ip -br addr
ip route
ip -s link
ethtool <interface>
ss -lntup
```

### InfiniBand

```bash
ibstat
ibdev2netdev
rdma link
ibv_devinfo
systemctl status opensm
```

### GPU

```bash
lspci | grep -i nvidia
nvidia-smi
nvidia-smi -q
lsmod | grep nvidia
```

### Slurm GPU

```bash
scontrol show node <node>
cat /etc/slurm/gres.conf
journalctl -u slurmd
```

### Lustre

```bash
mount | grep lustre
lfs df -h
lfs getstripe <path>
lfs quota -h -u <user> <filesystem>
dmesg -T | grep -i lustre
```

---

# Production Troubleshooting Rule

```text
                 SYMPTOM
                    │
                    ▼
                 SCOPE
                    │
                    ▼
                EVIDENCE
                    │
                    ▼
              INFRASTRUCTURE
                    │
       ┌────────────┼────────────┐
       ▼            ▼            ▼
     Linux        Network      Storage
       │            │            │
       ▼            ▼            ▼
     Slurm          IB          Lustre
       │
       ▼
      GPU
       │
       ▼
     Validate
```

> **Key principle:** In HPC, components are tightly coupled. A Slurm job failure can originate from the GPU, InfiniBand, Lustre, Linux kernel, authentication, or the scheduler itself. Always troubleshoot **across layers**, not inside a single tool.

# End of Chapter 17 – Part 3
