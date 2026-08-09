## Part 3 – Automation, Remote Operations & Command Patterns

> **Notebook focus:** Combine Linux and HPC commands into repeatable operational checks instead of running isolated commands manually.

* [16.24 Remote Operations](#1624-remote-operations)
* [16.25 SSH Operations](#1625-ssh-operations)
* [16.26 Parallel Node Checks](#1626-parallel-node-checks)
* [16.27 Command Chaining](#1627-command-chaining)
* [16.28 Pipes and Filters](#1628-pipes-and-filters)
* [16.29 Useful AWK Patterns](#1629-useful-awk-patterns)
* [16.30 Useful Grep Patterns](#1630-useful-grep-patterns)
* [16.31 Finding HPC Problems Quickly](#1631-finding-hpc-problems-quickly)
* [16.32 Basic Health-Check Script](#1632-basic-health-check-script)
* [16.33 Production Command Discipline](#1633-production-command-discipline)
* [16.34 Quick Revision](#1634-quick-revision)


# 16.24 Remote Operations

HPC administration frequently requires checking many nodes.

Basic SSH:

```bash
ssh compute01 hostname
```

Execute a command:

```bash
ssh compute01 uptime
```

Multiple commands:

```bash
ssh compute01 'uptime; free -h; df -hT'
```

---

# 16.25 SSH Operations

## Copy File

```bash
scp file.txt compute01:/tmp/
```

Copy from node:

```bash
scp compute01:/tmp/file.txt .
```

For directories:

```bash
scp -r config/ compute01:/tmp/
```

For large transfers, `rsync` is often more useful:

```bash
rsync -av config/ compute01:/tmp/config/
```

---

# 16.26 Parallel Node Checks

Suppose:

```text
compute01
compute02
compute03
compute04
```

Simple loop:

```bash
for node in compute01 compute02 compute03 compute04
do
    echo "===== $node ====="
    ssh "$node" hostname
    ssh "$node" uptime
done
```

A better operational pattern:

```bash
for node in compute01 compute02 compute03 compute04
do
    echo "===== $node ====="
    ssh "$node" 'hostname; uptime; free -h'
done
```

For larger clusters, use cluster-management/parallel execution tools such as xCAT `xdsh` or Ansible rather than uncontrolled SSH loops.

Example xCAT:

```bash
xdsh compute 'uptime'
```

---

# 16.27 Command Chaining

Commands can be combined using:

### `;`

Run the next command regardless of previous result:

```bash
command1 ; command2
```

### `&&`

Run the next command only if the first succeeds:

```bash
command1 && command2
```

Example:

```bash
systemctl is-active slurmd && echo "Slurmd OK"
```

### `||`

Run the next command if the first fails:

```bash
systemctl is-active slurmd || echo "Slurmd FAILED"
```

Useful health-check pattern:

```bash
systemctl is-active --quiet slurmd \
    && echo "OK" \
    || echo "FAILED"
```

---

# 16.28 Pipes and Filters

Pipes send output from one command to another.

```bash
ps -ef | grep slurmd
```

Example:

```bash
df -hT | grep -E '/home|/scratch'
```

Count results:

```bash
squeue | wc -l
```

Sort:

```bash
ps -eo pid,%cpu,comm --sort=-%cpu | head
```

Pipeline mental model:

```text
Command
   │
   ▼
Filter
   │
   ▼
Transform
   │
   ▼
Useful Result
```

---

# 16.29 Useful AWK Patterns

AWK is especially useful in HPC automation.

### Extract a Column

```bash
sinfo -N -h | awk '{print $1}'
```

### Filter

```bash
df -hT | awk '$6 == "/home"'
```

### CPU Usage

```bash
ps -eo pid,comm,%cpu --no-headers |
awk '$3 > 80'
```

### Remove `%`

```bash
awk '{gsub(/%/, "", $3); print $3}'
```

### Calculate/Format

```bash
awk '{printf "%-20s %s\n", $1, $2}'
```

Common pattern:

```text
Input
 ↓
awk
 ├── Select
 ├── Filter
 ├── Transform
 └── Format
```

---

# 16.30 Useful Grep Patterns

Search case-insensitively:

```bash
grep -i error logfile
```

Search recursively:

```bash
grep -R "failed" /var/log/
```

Show line numbers:

```bash
grep -n "ERROR" logfile
```

Invert match:

```bash
grep -v "^#" config
```

Multiple patterns:

```bash
grep -E "error|failed|timeout" logfile
```

Useful for HPC logs:

```bash
journalctl -u slurmd | grep -Ei "error|fail|timeout"
```

---

# 16.31 Finding HPC Problems Quickly

A practical diagnostic sequence:

```text
             Problem
                │
                ▼
             Hostname
                │
                ▼
          CPU / Memory
                │
                ▼
       Disk / Filesystem
                │
                ▼
            Network
                │
       ┌────────┼────────┐
       ▼        ▼        ▼
     Slurm      IB       GPU
       │        │        │
       └────────┼────────┘
                ▼
              Lustre
                │
                ▼
              LDAP
```

Example: GPU job fails.

Start with:

```bash
hostname
nvidia-smi
systemctl status slurmd
scontrol show node <node>
```

Then:

```bash
dmesg | grep -i nvidia
```

If the workload accesses Lustre:

```bash
lfs df -h
```

---

# 16.32 Basic Health-Check Script

A simple node health-check pattern:

```bash
#!/bin/bash

echo "===== NODE ====="
hostname

echo "===== UPTIME ====="
uptime

echo "===== MEMORY ====="
free -h

echo "===== FILESYSTEM ====="
df -hT

echo "===== FAILED SERVICES ====="
systemctl --failed

echo "===== SLURMD ====="
systemctl is-active slurmd

echo "===== GPU ====="
if command -v nvidia-smi >/dev/null 2>&1; then
    nvidia-smi --query-gpu=index,name,temperature.gpu,utilization.gpu \
        --format=csv,noheader
fi

echo "===== INFINIBAND ====="
if command -v ibstat >/dev/null 2>&1; then
    ibstat
fi
```

Run:

```bash
chmod +x node_health.sh
./node_health.sh
```

This is a starting point; production automation should add proper error handling, timeouts and structured output.

---

# 16.33 Production Command Discipline

In production HPC environments:

### 1. Verify Before Changing

```bash
systemctl status slurmd
```

before:

```bash
systemctl restart slurmd
```

### 2. Check Scope

Know whether the command affects:

```text
One process
One node
One rack
One partition
Entire cluster
```

### 3. Avoid Blind `kill -9`

Prefer:

```bash
kill <PID>
```

and investigate before using:

```bash
kill -9 <PID>
```

### 4. Record Evidence

Before making changes:

```bash
date
hostname
uptime
systemctl status <service>
journalctl -u <service> -n 100
```

### 5. Validate After Change

```text
Before
  ↓
Change
  ↓
Validate
  ↓
Monitor
```

---

# 16.34 Quick Revision

## Remote

```bash
ssh node command
scp file node:/path/
rsync -av source/ node:/path/
```

## Automation

```bash
for node in ...
do
    ssh "$node" ...
done
```

## Filtering

```bash
grep
awk
sort
head
tail
cut
wc
```

## Command Logic

```bash
cmd1 && cmd2
cmd1 || cmd2
cmd1 ; cmd2
```

## HPC Diagnostics

```bash
scontrol show node
nvidia-smi
ibstat
lfs df -h
systemctl status slurmd
journalctl -u slurmd
```

---

# Daily Operations Mental Model

```text
                Observe
                   │
                   ▼
               Identify
                   │
                   ▼
              Diagnose
                   │
                   ▼
                Change
                   │
                   ▼
               Validate
                   │
                   ▼
               Document
```

> **HPC Engineer rule:** Commands are not the skill by themselves. The important skill is knowing **which command to run, why you are running it, what evidence it provides, and what to check next**.

# End of Chapter 16 – Part 3
