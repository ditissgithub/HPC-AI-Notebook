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


# 17.39 LDAP and SSSD Troubleshooting

Authentication path:

```text id="z3qv3h"
User
 │
 ▼
SSH / Login
 │
 ▼
PAM
 │
 ▼
NSS / SSSD
 │
 ▼
LDAP
 │
 ▼
Directory
```

Start with:

```bash id="qv7zj6"
id user01
```

Then:

```bash id="skx9ft"
getent passwd user01
```

If both fail, investigate identity lookup before troubleshooting SSH itself.

---

## SSSD

Check service:

```bash id="j8d7h9"
systemctl status sssd
```

Check configuration:

```bash id="5k4h8c"
sssctl config-check
```

List domains:

```bash id="t7x4hy"
sssctl domain-list
```

Check domain:

```bash id="9f3r0s"
sssctl domain-status <domain>
```

Logs:

```bash id="6m5d5p"
journalctl -u sssd -n 100
```

---

# 17.40 Authentication Failure Flow

If:

```bash id="v9d3z0"
id user01
```

fails:

```text id="3v3hqp"
id user01
   ↓
getent passwd user01
   │
   ├── Works
   │      ↓
   │   Check SSH/PAM
   │
   └── Fails
          ↓
       Check SSSD
          ↓
       Check LDAP
```

Test LDAP directly:

```bash id="4nyrzi"
ldapsearch -x \
-H ldap://ldap01 \
-b "dc=nsm,dc=in" \
"(uid=user01)"
```

Possible causes:

```text id="3ax5l8"
LDAP unavailable
SSSD stopped
DNS failure
TLS/certificate problem
Incorrect base DN
LDAP access failure
Expired/locked account
NSS configuration issue
```

---

# 17.41 SSH Login Problems

First test identity:

```bash id="0t8j8c"
id user01
```

Then test SSH:

```bash id="r9lqtd"
ssh user01@compute01
```

Check SSH service:

```bash id="y9y1l7"
systemctl status sshd
```

Logs:

```bash id="fz8m6b"
journalctl -u sshd -n 100
```

On systems using traditional auth logs, also inspect the appropriate authentication log under `/var/log/`.

Check SSH configuration:

```bash id="o5o1vn"
sshd -T
```

Common causes:

```text id="67q0eq"
User lookup failure
PAM failure
SSSD failure
Home directory problem
SSH configuration
Permissions
Account restrictions
```

---

# 17.42 xCAT Provisioning Troubleshooting

Typical provisioning flow:

```text id="3zdbf7"
Node
 ↓
DHCP
 ↓
PXE/iPXE
 ↓
Bootloader
 ↓
OS Image
 ↓
Install
 ↓
Post-install
 ↓
Node Configuration
```

When provisioning fails, identify **which stage** failed.

Check node definition:

```bash id="9fj0mm"
lsdef <node>
```

Check node status:

```bash id="0m4x90"
nodestat <node>
```

Check network information:

```bash id="i9g9s4"
lsdef <node> -i
```

Check xCAT tables:

```bash id="0c3x5d"
tabdump
```

Useful tables include:

```text id="n89hzi"
nodes
nodetype
hosts
networks
site
passwd
```

---

# 17.43 PXE and DHCP Problems

If a node does not start network boot:

```text id="2s3m8c"
Node
 ↓
NIC
 ↓
DHCP
 ↓
PXE
 ↓
TFTP/HTTP
 ↓
Bootloader
```

Check DHCP service:

```bash id="o6z3jb"
systemctl status dhcpd
```

Check logs:

```bash id="l2b5am"
journalctl -u dhcpd
```

Check listening port:

```bash id="i6c3pq"
ss -lunp | grep ':67'
```

Check node network:

```bash id="c9a4s2"
ip link
```

If DHCP works but bootloader does not load, investigate the next provisioning stage rather than repeatedly restarting DHCP.

---

# 17.44 Node Provisioning Failure

A node may successfully PXE boot but fail during installation.

Break the process into:

```text id="v4w9y1"
PXE
 ↓
Installer
 ↓
Storage
 ↓
Package installation
 ↓
Network configuration
 ↓
Post-install
 ↓
Slurm/xCAT configuration
```

Typical causes:

```text id="dd3k9p"
Incorrect image
Package repository failure
Disk problem
Network failure
DNS failure
Post-install script failure
Configuration mismatch
```

After provisioning, validate:

```bash id="3k2q9y"
hostname
ip -br addr
df -hT
systemctl --failed
```

Then:

```bash id="oz2x7m"
systemctl status slurmd
```

And, where applicable:

```bash id="br5z99"
nvidia-smi
ibstat
mount | grep lustre
```

---

# 17.45 Cluster-Wide Failure

If many nodes fail simultaneously, **do not troubleshoot each node independently first**.

Look for shared dependencies:

```text id="o1v3fq"
              Cluster Failure
                    │
        ┌───────────┼───────────┐
        ▼           ▼           ▼
      DNS          LDAP       Network
        │           │           │
        ▼           ▼           ▼
      Slurm       Storage     DHCP
        │           │
        └──────┬────┘
               ▼
          Shared Service
```

Examples:

### Many users cannot log in

Investigate:

```text id="k8i1g4"
LDAP
SSSD
DNS
Authentication services
```

### Many jobs cannot start

Investigate:

```text id="q9j0c5"
Slurmctld
Slurm database/accounting
Partitions
Shared filesystem
```

### Many nodes lose connectivity

Investigate:

```text id="6v5xg8"
Switch
Routing
Bond
VLAN
MTU
Fabric
```

---

# 17.46 Recent Change Analysis

A very useful production question:

> **What changed before the problem started?**

Check:

```bash id="y4c8c6"
journalctl --since "2 hours ago"
```

Package changes:

```bash id="x5b7a3"
rpm -qa --last | head
```

Kernel:

```bash id="4l7d4f"
uname -r
```

System boot:

```bash id="u5o6e0"
last reboot
```

Compare:

```text id="wmh90h"
Working Node
     vs
Failed Node
```

Check:

```text id="5d1k5e"
OS
Kernel
Driver
Configuration
Packages
Network
Services
```

Configuration drift is a common cause of node-specific failures.

---

# 17.47 Incident Handling

During a production incident:

### 1. Confirm

```text id="1f1r8u"
What is broken?
```

### 2. Contain

Prevent the problem from spreading.

For example, isolate an unhealthy node from scheduling rather than allowing jobs to continue failing.

```bash id="p9o2k8"
scontrol update NodeName=<node> State=DRAIN Reason="investigation"
```

### 3. Collect

Record:

```text id="s2b0h1"
Time
Node
Job
Error
Logs
Recent changes
Affected users
```

### 4. Recover

Restore service using the smallest safe change.

### 5. Validate

Run an actual workload or health check.

### 6. Document

Record the root cause and corrective action.

---

# 17.48 Root Cause Analysis

Do not stop at:

> "Slurm node was down."

That describes the symptom.

Better:

```text id="2aj1gt"
Symptom:
Node entered DOWN state.

Evidence:
slurmd stopped.

Root Cause:
Configuration mismatch after package update.

Corrective Action:
Restored compatible configuration.

Validation:
slurmd registered successfully and test job completed.
```

---

## 5-Why Example

```text id="g1l7nq"
Why did the job fail?
        ↓
GPU unavailable.

Why?
        ↓
nvidia-smi failed.

Why?
        ↓
NVIDIA kernel module was not loaded.

Why?
        ↓
Driver module did not match running kernel.

Why?
        ↓
Kernel was updated without rebuilding/revalidating
the NVIDIA driver stack.
```

This identifies a systemic improvement rather than only repairing one node.

---

# 17.49 Production Recovery Checklist

Before returning a failed node:

```text id="3l1w2a"
☐ Linux healthy
☐ CPU healthy
☐ Memory healthy
☐ Disk healthy
☐ Network healthy
☐ InfiniBand ACTIVE
☐ GPU healthy
☐ Lustre accessible
☐ LDAP lookup works
☐ SSSD healthy
☐ slurmd running
☐ Slurm node registered
☐ No critical logs
☐ Test workload successful
```

Then:

```bash id="8b0g8m"
scontrol update NodeName=<node> State=RESUME
```

Finally:

```bash id="e9g7k1"
sinfo -N -l
```

---

# 17.50 Quick Revision

### Authentication

```bash id="f5r6o4"
id user
getent passwd user
systemctl status sssd
sssctl domain-status <domain>
ldapsearch
```

### xCAT

```bash id="k1y4v8"
lsdef <node>
nodestat <node>
tabdump
systemctl status dhcpd
```

### Incident

```text id="w3l7r9"
Detect
 ↓
Scope
 ↓
Contain
 ↓
Collect Evidence
 ↓
Diagnose
 ↓
Recover
 ↓
Validate
 ↓
Document
```

### Root Cause

```text id="z8u2n1"
Symptom ≠ Root Cause
```

> **Production rule:** When multiple nodes fail at the same time, always investigate **shared infrastructure and dependencies** before assuming independent node failures.

# End of Chapter 17 – Part 4
