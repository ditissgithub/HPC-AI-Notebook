# HPC-AI Infrastructure Automation — Bash Cheat Sheet

The goal of this cheat sheet is **not to teach all of Bash**. It is a practical library of reusable patterns you can combine when solving HPC-AI infrastructure automation problems.

Think:

> **Requirement → Flowchart → Choose patterns → Assemble script → Test → Report**

---

## 1. Script Skeleton

Use this as your default starting point.

```bash
#!/usr/bin/env bash

set -uo pipefail

usage()
{
    echo "Usage: $0 <input>"
    exit 1
}

main()
{
    ...
}

if [[ $# -ne 1 ]]; then
    usage
fi

main "$1"
exit $?
```

For scripts where unexpected command failures should terminate execution, consider:

```bash
set -euo pipefail
```

But **don't blindly use `set -e`** in health-check scripts where command failures are expected conditions that you want to classify and continue processing.

---

## 2. Root / Administrative Check

### Check UID

```bash
if [[ $EUID -ne 0 ]]; then
    echo "ERROR: Must run as root"
    exit 1
fi
```

### Alternative

```bash
if (( EUID != 0 )); then
    echo "ERROR: Root privileges required"
    exit 1
fi
```

### HPC use cases

- User creation
- Package installation
- systemd management
- `/etc` configuration
- SSH key deployment
- Network configuration
- GPU driver configuration

---

## 3. Command-Line Arguments

### Check number of arguments

```bash
if [[ $# -ne 2 ]]; then
    echo "Usage: $0 <service> <node_file>"
    exit 1
fi
```

### Store arguments

```bash
service="$1"
node_file="$2"
```

Always quote:

```bash
"$service"
"$node_file"
```

---

## 4. File / Directory Validation

### File exists

```bash
[[ -f "$file" ]] || {
    echo "ERROR: File not found: $file"
    return 1
}
```

### File readable

```bash
[[ -r "$file" ]] || {
    echo "ERROR: File is not readable: $file"
    return 1
}
```

### Directory exists

```bash
[[ -d "$dir" ]] || {
    echo "ERROR: Directory not found: $dir"
    return 1
}
```

### Directory writable

```bash
[[ -w "$dir" ]] || {
    echo "ERROR: Directory is not writable"
    return 1
}
```

### File is not empty

```bash
[[ -s "$file" ]] || {
    echo "ERROR: File is empty"
    return 1
}
```

---

## 5. Functions

Your standard automation function:

```bash
check_node()
{
    local node="$1"

    echo "Checking $node"

    ...
}
```

Multiple parameters:

```bash
health_check()
{
    local service="$1"
    local node_file="$2"

    ...
}
```

Return success:

```bash
return 0
```

Return failure:

```bash
return 1
```

### Important

Inside a function:

```bash
return 1
```

Usually means:

> Return failure to caller.

Whereas:

```bash
exit 1
```

means:

> Terminate the entire script.

---

## 6. Reading a File Safely

This is one of the **most important patterns** for HPC automation.

```bash
while IFS= read -r line
do
    echo "$line"
done < "$file"
```

Use this for:

- node lists
- service lists
- user lists
- package lists
- configuration files
- inventory files

### Skip blank lines

```bash
[[ -z "$line" ]] && continue
```

### Skip comments

```bash
[[ "$line" =~ ^[[:space:]]*# ]] && continue
```

Combined:

```bash
while IFS= read -r line
do
    [[ -z "$line" ]] && continue
    [[ "$line" =~ ^[[:space:]]*# ]] && continue

    ...
done < "$file"
```

---

## 7. Whitespace Cleanup

Remove leading whitespace:

```bash
line="${line#"${line%%[![:space:]]*}"}"
```

Remove trailing whitespace:

```bash
line="${line%"${line##*[![:space:]]}"}"
```

Both:

```bash
line="${line#"${line%%[![:space:]]*}"}"
line="${line%"${line##*[![:space:]]*}"}"
```

Useful for:

```text
    cn01
cn02    
```

---

## 8. `if` — Your Most Important Structure

### Command success

```bash
if command; then
    echo "Success"
else
    echo "Failed"
fi
```

Example:

```bash
if systemctl restart slurmd; then
    echo "Restart successful"
else
    echo "Restart failed"
fi
```

### Bash condition

```bash
if [[ "$status" == "active" ]]; then
    echo "Running"
fi
```

Remember:

```text
if command
    → execute command and test exit status

if [[ condition ]]
    → evaluate Bash condition
```

---

## 9. Common String Tests

```bash
[[ "$x" == "$y" ]]
```

Equal.

```bash
[[ "$x" != "$y" ]]
```

Not equal.

```bash
[[ -z "$x" ]]
```

Empty.

```bash
[[ -n "$x" ]]
```

Not empty.

---

## 10. Numeric Comparisons

```bash
[[ "$count" -eq 10 ]]
```

Equal.

```bash
[[ "$count" -ne 10 ]]
```

Not equal.

```bash
[[ "$count" -gt 10 ]]
```

Greater than.

```bash
[[ "$count" -ge 10 ]]
```

Greater/equal.

```bash
[[ "$count" -lt 10 ]]
```

Less than.

```bash
[[ "$count" -le 10 ]]
```

Less/equal.

For arithmetic, also use:

```bash
(( count > 10 ))
```

---

## 11. Increment Counters

```bash
((TOTAL++))
```

Examples:

```bash
((ACTIVE++))
((FAILED++))
((INVALID++))
((RESTARTED++))
((UNREACHABLE++))
```

Initialize:

```bash
TOTAL=0
ACTIVE=0
FAILED=0
```

---

## 12. `for` Loop

```bash
for node in "${nodes[@]}"
do
    echo "$node"
done
```

For known values:

```bash
for service in slurmd munge chronyd
do
    systemctl is-active "$service"
done
```

---

## 13. Arrays

Create:

```bash
nodes=("cn01" "cn02" "cn03")
```

Loop:

```bash
for node in "${nodes[@]}"
do
    echo "$node"
done
```

Number of elements:

```bash
echo "${#nodes[@]}"
```

Add:

```bash
nodes+=("cn04")
```

---

## 14. Associative Arrays — Deduplication

Very useful for cluster scripts.

```bash
declare -A SEEN
```

Check duplicate:

```bash
if [[ ${SEEN["$node"]+_} ]]; then
    echo "$node is duplicate"
    continue
fi
```

Mark as seen:

```bash
SEEN["$node"]=1
```

Excellent for:

- duplicate hostnames
- duplicate users
- duplicate services
- duplicate IP addresses

---

## 15. Check Command Availability

```bash
if ! command -v ssh >/dev/null 2>&1; then
    echo "ERROR: ssh command not found"
    exit 1
fi
```

Multiple dependencies:

```bash
for cmd in ssh systemctl awk sed
do
    command -v "$cmd" >/dev/null 2>&1 || {
        echo "ERROR: $cmd not installed"
        exit 1
    }
done
```

---

## 16. SSH — HPC Cluster Automation

### Basic

```bash
ssh "$node" hostname
```

### Execute remote command

```bash
ssh "$node" systemctl is-active slurmd
```

### Non-interactive SSH

Very important for automation:

```bash
ssh \
    -o BatchMode=yes \
    -o ConnectTimeout=5 \
    "$node" hostname
```

### Check SSH connectivity

```bash
if ssh \
    -o BatchMode=yes \
    -o ConnectTimeout=5 \
    "$node" true &>/dev/null
then
    echo "$node : SSH OK"
else
    echo "$node : SSH FAILED"
fi
```

This is generally more useful than only checking port 22.

---

## 17. `nc` Network Check

Check SSH port:

```bash
nc -z -w 3 "$node" 22
```

With condition:

```bash
if nc -z -w 3 "$node" 22 &>/dev/null; then
    echo "Port 22 reachable"
else
    echo "Port 22 unreachable"
fi
```

Use it when you specifically want to test **TCP connectivity**.

---

## 18. Ping

```bash
if ping -c 2 -W 2 "$node" &>/dev/null; then
    echo "$node : reachable"
else
    echo "$node : unreachable"
fi
```

Remember:

```text
ping failure
    ≠
SSH failure
```

A node may block ICMP while SSH works.

---

## 19. systemd Service Checks

### Service exists

```bash
systemctl cat "$service" &>/dev/null
```

### Current state

```bash
systemctl is-active "$service"
```

### Check active

```bash
if systemctl is-active --quiet "$service"; then
    echo "Active"
else
    echo "Inactive"
fi
```

### Enabled?

```bash
systemctl is-enabled "$service"
```

### Restart

```bash
systemctl restart "$service"
```

### Start

```bash
systemctl start "$service"
```

### Stop

```bash
systemctl stop "$service"
```

### Reload

```bash
systemctl reload "$service"
```

### Enable at boot

```bash
systemctl enable "$service"
```

### Restart + verify

```bash
if systemctl restart "$service"; then

    if systemctl is-active --quiet "$service"; then
        echo "$service : restart successful"
    else
        echo "$service : restart completed but service is not active"
    fi

else
    echo "$service : restart failed"
fi
```

---

## 20. HPC Service Health Pattern

Worth memorizing:

```bash
status=$(ssh "$node" systemctl is-active slurmd 2>/dev/null || true)

if [[ "$status" == "active" ]]; then

    echo "$node : slurmd ACTIVE"

else

    echo "$node : slurmd $status"

    if ssh "$node" systemctl restart slurmd; then

        new_status=$(ssh "$node" systemctl is-active slurmd 2>/dev/null || true)

        if [[ "$new_status" == "active" ]]; then
            echo "$node : slurmd recovered"
        else
            echo "$node : slurmd FAILED"
        fi

    fi
fi
```

The architecture is:

```text
CHECK
  ↓
CLASSIFY
  ↓
REMEDIATE
  ↓
VERIFY
  ↓
REPORT
```

---

## 21. Filesystem Automation

### Find `.log` files

```bash
find "$dir" -type f -name "*.log"
```

### Older than N days

```bash
find "$dir" -type f -name "*.log" -mtime +"$days"
```

### Delete

```bash
find "$dir" -type f -name "*.log" -mtime +"$days" -delete
```

### Dry run

```bash
find "$dir" -type f -name "*.log" -mtime +"$days" -print
```

**Always dry-run destructive operations first.**

---

## 22. Safe File Processing

For filenames containing spaces:

```bash
find "$dir" -type f -print0 |
while IFS= read -r -d '' file
do
    echo "$file"
done
```

Even better:

```bash
while IFS= read -r -d '' file
do
    ...
done < <(
    find "$dir" -type f -print0
)
```

---

## 23. Disk Usage

```bash
df -h
```

Specific filesystem:

```bash
df -h /
```

Extract usage:

```bash
df -P / | awk 'NR==2 {print $5}'
```

Remove `%`:

```bash
usage=$(df -P / | awk 'NR==2 {gsub(/%/, "", $5); print $5}')
```

Check threshold:

```bash
if (( usage > threshold )); then
    echo "Disk usage high"
fi
```

---

## 24. Memory

```bash
free -h
```

Available memory:

```bash
free -m | awk '/Mem:/ {print $7}'
```

Memory percentage:

```bash
free | awk '/Mem:/ {
    printf "%.2f\n", ($3/$2)*100
}'
```

---

## 25. CPU

Number of CPUs:

```bash
nproc
```

Load:

```bash
uptime
```

Machine-readable:

```bash
awk '{print $1}' /proc/loadavg
```

Top CPU processes:

```bash
ps aux --sort=-%cpu | head
```

Top memory processes:

```bash
ps aux --sort=-%mem | head
```

Specific process:

```bash
pgrep -af slurmd
```

---

## 26. Process Health

Check process:

```bash
if pgrep -x slurmd >/dev/null; then
    echo "slurmd process running"
else
    echo "slurmd process not running"
fi
```

Get PID:

```bash
pid=$(pgrep -xo slurmd)
```

---

## 27. Network Interfaces

List interfaces:

```bash
ip -br link
```

IP addresses:

```bash
ip -br addr
```

Check interface:

```bash
ip link show "$interface"
```

Check UP:

```bash
ip link show "$interface" |
    grep -q "state UP"
```

---

## 28. InfiniBand

### Devices

```bash
ibstat
```

### Ports

```bash
ibdev2netdev
```

### Link state

```bash
cat /sys/class/infiniband/*/ports/*/state
```

### Check IB interfaces

```bash
ibdev2netdev | grep -i up
```

For HPC automation:

```text
Node
 ↓
IB device exists?
 ↓
Port active?
 ↓
Link speed?
 ↓
Report
```

---

## 29. NVIDIA GPU

### GPU visibility

```bash
nvidia-smi
```

### Count GPUs

```bash
nvidia-smi -L | wc -l
```

### Query specific information

```bash
nvidia-smi \
    --query-gpu=index,name,temperature.gpu,memory.total,memory.used,utilization.gpu \
    --format=csv,noheader
```

---

## 30. GPU Health Pattern

```bash
if nvidia-smi &>/dev/null; then
    echo "GPU driver healthy"
else
    echo "GPU driver unhealthy"
fi
```

Count:

```bash
gpu_count=$(nvidia-smi -L 2>/dev/null | wc -l)
```

Expected vs actual:

```bash
if (( gpu_count == expected_gpu_count )); then
    echo "GPU count OK"
else
    echo "GPU count mismatch"
fi
```

---

## 31. Slurm

Check controller:

```bash
systemctl is-active slurmctld
```

Check compute daemon:

```bash
systemctl is-active slurmd
```

Node status:

```bash
sinfo
```

Detailed:

```bash
scontrol show node "$node"
```

Jobs:

```bash
squeue
```

Cluster health:

```bash
sinfo -N -l
```

Useful automation pattern:

```bash
sinfo -N -h -o "%N %T"
```

Then process with:

```bash
while read -r node state
do
    ...
done < <(sinfo -N -h -o "%N %T")
```

---

## 32. User Management

Check user:

```bash
id "$username"
```

Create:

```bash
useradd "$username"
```

Check group:

```bash
getent group "$group"
```

Create group:

```bash
groupadd "$group"
```

Add user:

```bash
usermod -aG "$group" "$username"
```

Check membership:

```bash
id "$username"
```

---

## 33. LDAP

Query:

```bash
ldapsearch -x \
    -H ldap://"$ldap_server" \
    -b "$base_dn"
```

Check user:

```bash
getent passwd "$username"
```

Check group:

```bash
getent group "$group"
```

For HPC clusters, distinguish:

```text
local user
    vs
LDAP user
```

Do not blindly create a local account when the identity is supposed to come from LDAP.

---

## 34. Package Management

Rocky/Alma/RHEL:

```bash
rpm -q "$package"
```

Check installed:

```bash
if rpm -q "$package" &>/dev/null; then
    echo "Installed"
else
    echo "Missing"
fi
```

Install:

```bash
dnf install -y "$package"
```

Update:

```bash
dnf update -y
```

---

## 35. Configuration Backup

Before modifying a production configuration:

```bash
cp -a "$file" "$file.bak.$(date +%Y%m%d_%H%M%S)"
```

Example:

```bash
cp -a /etc/sssd/sssd.conf \
    /etc/sssd/sssd.conf.bak.$(date +%Y%m%d_%H%M%S)
```

---

## 36. Logging

Simple:

```bash
echo "$(date '+%F %T') : message"
```

Reusable:

```bash
log()
{
    echo "$(date '+%F %T') : $*"
}
```

Use:

```bash
log "Checking node $node"
```

Error:

```bash
log "ERROR: SSH failed for $node"
```

---

## 37. Summary File

Initialize:

```bash
: > "$summary_file"
```

Append:

```bash
echo "$node : ACTIVE" >> "$summary_file"
```

Print:

```bash
cat "$summary_file"
```

---

## 38. Temporary Files

Use:

```bash
tmp_file=$(mktemp)
```

Clean automatically:

```bash
trap 'rm -f "$tmp_file"' EXIT
```

This is much safer than:

```bash
tmp_file="/tmp/myfile.txt"
```

---

## 39. `trap` for Cleanup

```bash
cleanup()
{
    rm -f "$tmp_file"
}

trap cleanup EXIT
```

Useful for:

- temporary files
- locks
- mounted resources
- temporary directories

---

## 40. Exit Codes

Success:

```bash
exit 0
```

Generic failure:

```bash
exit 1
```

Function failure:

```bash
return 1
```

Typical automation:

```text
0 → success
1 → general failure
2 → partial failure / validation issue
```

Example:

```bash
if (( failed > 0 )); then
    exit 2
fi

exit 0
```

---

## 41. Capture Command Output

```bash
result=$(command)
```

Example:

```bash
hostname=$(hostname)
```

Remote:

```bash
kernel=$(ssh "$node" uname -r)
```

---

## 42. Capture Command Exit Status

Preferred:

```bash
if command; then
    echo "Success"
else
    echo "Failure"
fi
```

Or:

```bash
command

rc=$?

if (( rc != 0 )); then
    echo "Command failed: $rc"
fi
```

Remember:

```bash
$?
```

is the exit status of the **immediately preceding command**.

---

## 43. `grep` Checks

```bash
if grep -q "pattern" "$file"; then
    echo "Found"
fi
```

Case insensitive:

```bash
grep -qi "pattern" "$file"
```

Invert:

```bash
if ! grep -q "pattern" "$file"; then
    echo "Not found"
fi
```

---

## 44. `awk` — Infrastructure Data Extraction

Get column:

```bash
awk '{print $1}' file
```

CSV:

```bash
awk -F',' '{print $1}' file.csv
```

Skip header:

```bash
awk 'NR > 1'
```

Condition:

```bash
awk '$3 > 80'
```

Format:

```bash
awk '{printf "%-20s %s\n", $1, $2}'
```

Remove `%`:

```bash
awk '{gsub(/%/, "", $5); print $5}'
```

---

## 45. `sed`

Replace:

```bash
sed 's/old/new/g' file
```

Delete blank lines:

```bash
sed '/^[[:space:]]*$/d'
```

Delete comments:

```bash
sed '/^[[:space:]]*#/d'
```

---

## 46. `sort` + `uniq`

Unique:

```bash
sort file | uniq
```

Count duplicates:

```bash
sort file | uniq -c
```

Sort numerically:

```bash
sort -n
```

Reverse:

```bash
sort -nr
```

---

## 47. Useful Pipeline Pattern

A common infrastructure pattern:

```bash
command |
    awk '...' |
    sort |
    uniq
```

But don't automatically use pipelines everywhere.

Prefer simple Bash when the logic is simple.

---

## 48. Parallel SSH — Basic HPC Pattern

For a small cluster:

```bash
while IFS= read -r node
do
    ssh -o BatchMode=yes "$node" hostname &
done < nodes.txt

wait
```

Architecture:

```text
node1 ─┐
node2 ─┤
node3 ─┼── parallel SSH
node4 ─┤
node5 ─┘
        ↓
       wait
```

For large production clusters, consider whether **Ansible, Slurm, pdsh, clustershell, or another distributed execution mechanism** is more appropriate than spawning hundreds/thousands of SSH processes.

---

## 49. Parallel Job Limiting

Don't blindly launch 1,000 SSH sessions.

A simple Bash semaphore pattern:

```bash
MAX_JOBS=10

while IFS= read -r node
do
    while (( $(jobs -rp | wc -l) >= MAX_JOBS )); do
        wait -n
    done

    ssh "$node" hostname &

done < nodes.txt

wait
```

This gives you a basic concurrency limit.

---

## 50. Retry Pattern

Very useful for HPC nodes.

```bash
for ((attempt=1; attempt<=3; attempt++))
do
    if ssh -o ConnectTimeout=5 "$node" hostname &>/dev/null; then
        echo "$node : SUCCESS"
        break
    fi

    echo "$node : attempt $attempt failed"
    sleep 2
done
```

---

## 51. Timeout

Command:

```bash
timeout 10 command
```

Example:

```bash
if timeout 10 ssh "$node" hostname; then
    echo "Success"
else
    echo "Timeout/failure"
fi
```

Excellent for cluster automation.

---

## 52. Dry-Run Pattern

```bash
DRY_RUN=false
```

Then:

```bash
if [[ "$DRY_RUN" == true ]]; then
    echo "[DRY-RUN] systemctl restart $service"
else
    systemctl restart "$service"
fi
```

For dangerous operations:

```text
DELETE
RESTART
USERMOD
DNF REMOVE
CONFIG CHANGE
```

always consider dry-run mode.

---

## 53. Confirmation for Dangerous Operations

```bash
read -r -p "Continue? [y/N]: " answer

if [[ "$answer" != "y" ]]; then
    echo "Aborted"
    exit 1
fi
```

For HPC production automation, however, **non-interactive scripts** are generally preferable, so a `--dry-run` mode is often better than interactive confirmation.

---

## 54. Locking — Prevent Two Copies Running

Basic:

```bash
LOCK_FILE="/var/run/my_script.lock"

if [[ -e "$LOCK_FILE" ]]; then
    echo "Another instance is running"
    exit 1
fi

touch "$LOCK_FILE"

trap 'rm -f "$LOCK_FILE"' EXIT
```

For production, prefer `flock`:

```bash
exec 200>/var/run/my_script.lock

flock -n 200 || {
    echo "Another instance is running"
    exit 1
}
```

---

## 55. Common HPC Health-Check Architecture

Memorize this pattern.

```text
                 INPUT
                   │
                   ▼
             Validate Input
                   │
                   ▼
              Read Node
                   │
                   ▼
             Duplicate?
              /       \
            Yes       No
             │         │
           Skip        ▼
                 Connectivity
                      │
                ┌─────┴─────┐
              FAIL          OK
                │             │
             Report           ▼
                         Remote Check
                              │
                              ▼
                           Healthy?
                         /         \
                       Yes         No
                        │           │
                     Report     Remediate
                                    │
                                    ▼
                                 Verify
                                    │
                                    ▼
                                  Report
                                    │
                                    ▼
                               Next Node
                                    │
                                    ▼
                                  SUMMARY
```

This pattern can solve dozens of automation problems.

---

## 56. Node Health Matrix

For cluster automation, aim for this type of output:

```text
NODE    SSH    SERVICE    STATE       ACTION
------------------------------------------------
cn01    OK     slurmd     ACTIVE      NONE
cn02    OK     slurmd     INACTIVE    RESTART
cn03    FAIL   UNKNOWN    UNKNOWN     ALERT
cn04    OK     MISSING    UNKNOWN     ALERT
cn05    OK     slurmd     ACTIVE      NONE
```

Think of your Bash script as a **state collection engine**, not just a command runner.

---

## 57. Recommended Counters

For cluster health scripts:

```bash
TOTAL=0
DUPLICATE=0
REACHABLE=0
UNREACHABLE=0
SSH_FAILED=0
VALID=0
INVALID=0
ACTIVE=0
INACTIVE=0
RESTART_ATTEMPTED=0
RESTART_SUCCESS=0
RESTART_FAILED=0
FINAL_FAILED=0
```

Don't necessarily use all of them in every script. Select the ones relevant to the problem.

---

## 58. Standard Summary

Use this structure:

```bash
{
    echo
    echo "========================================"
    echo "           FINAL SUMMARY"
    echo "========================================"
    echo "Total Nodes        : $TOTAL"
    echo "Duplicate Nodes    : $DUPLICATE"
    echo "Unreachable Nodes  : $UNREACHABLE"
    echo "Valid Nodes        : $VALID"
    echo "Invalid Nodes      : $INVALID"
    echo "Active             : $ACTIVE"
    echo "Inactive           : $INACTIVE"
    echo "Restart Attempted  : $RESTART_ATTEMPTED"
    echo "Restart Successful : $RESTART_SUCCESS"
    echo "Restart Failed     : $RESTART_FAILED"
    echo "========================================"
} >> "$SUMMARY_FILE"
```

---

## 59. Bash Debugging

### Syntax check

```bash
bash -n script.sh
```

### Trace execution

```bash
bash -x script.sh
```

### More controlled debugging

```bash
set -x
...
set +x
```

### Check variables

```bash
declare -p variable
```

### Check exit status

```bash
echo "$?"
```

---

# 60. Your 15 Most Important Patterns

If you want to memorize only **15**, memorize these:

```bash
# 1. Root
[[ $EUID -eq 0 ]] || exit 1

# 2. File
[[ -f "$file" ]] || exit 1

# 3. Directory
[[ -d "$dir" ]] || exit 1

# 4. Read file
while IFS= read -r line; do
    ...
done < "$file"

# 5. Skip blank
[[ -z "$line" ]] && continue

# 6. Skip comment
[[ "$line" =~ ^[[:space:]]*# ]] && continue

# 7. Command success
if command; then
    ...
fi

# 8. String comparison
[[ "$status" == "active" ]]

# 9. Counter
((COUNT++))

# 10. SSH
ssh -o BatchMode=yes -o ConnectTimeout=5 "$node" command

# 11. systemd
systemctl is-active "$service"

# 12. Find
find "$dir" -type f -name "*.log" -mtime +"$days"

# 13. Capture output
result=$(command)

# 14. Temporary file
tmp=$(mktemp)

# 15. Cleanup
trap 'rm -f "$tmp"' EXIT
```

---

# Mental Cheat Sheet

When you receive an HPC automation problem, don't think:

> **"Which Bash command do I need?"**

Think:

```text
                REQUIREMENT
                     ↓
                FLOWCHART
                     ↓
        ┌────────────┼────────────┐
        ↓            ↓            ↓
     INPUT        DECISION      ACTION
        ↓            ↓            ↓
     read        if / [[ ]]    command
        ↓            ↓            ↓
     validate     classify     execute
        └────────────┼────────────┘
                     ↓
                   VERIFY
                     ↓
                  COUNTER
                     ↓
                  REPORT
                     ↓
               EXIT STATUS
```

## HPC-AI Problem → Bash Pattern

| Requirement | Bash pattern |
|---|---|
| Many nodes | `while read` |
| Remote execution | `ssh` |
| Network test | `ping` / `nc` |
| Service check | `systemctl is-active` |
| Process check | `pgrep` |
| GPU check | `nvidia-smi` |
| IB check | `ibstat` / `ibdev2netdev` |
| Slurm check | `sinfo` / `scontrol` |
| Disk check | `df` / `find` |
| User management | `id` / `useradd` / `usermod` |
| Package check | `rpm -q` |
| Config parsing | `awk` / `sed` |
| Duplicate detection | associative array |
| Failure tracking | counters |
| Safe execution | `if command; then` |
| Large cluster | concurrency/retry/timeout |
| Dangerous operation | `--dry-run` |
| Auditability | timestamp + summary |
| Production safety | validation + rollback + exit status |

---

# Practice Strategy

Don't memorize this entire page.

For HPC-AI automation practice, keep this cheat sheet beside you and force yourself to write scripts from the **flowchart first**, then look only for the Bash pattern you need.

The progression should be:

**Flowchart → identify 5–10 patterns → write script → `bash -n` → `bash -x` → test failure cases → summary.**

The target is to make these patterns muscle memory so you can solve HPC infrastructure automation problems without relying on an AI assistant.
