## Part 3 – Quotas, Administration & Troubleshooting

> **Notebook focus:** Manage user/group quotas on Lustre and troubleshoot common capacity and access problems.

* [9.21 Lustre Quotas](#921-lustre-quotas)
* [9.22 User Quota on Home](#922-user-quota-on-home)
* [9.23 User Quota on Scratch](#923-user-quota-on-scratch)
* [9.24 Soft and Hard Limits](#924-soft-and-hard-limits)
* [9.25 Verify Quotas](#925-verify-quotas)
* [9.26 Group Quotas](#926-group-quotas)
* [9.27 Quota Troubleshooting](#927-quota-troubleshooting)
* [9.28 Production Quota Example](#928-production-quota-example)
* [9.29 Important Commands](#929-important-commands)
* [9.30 Quick Revision](#930-quick-revision)

# 9.21 Lustre Quotas

Lustre supports quotas to control how much storage a user or group can consume.

Conceptually:

```text
                    Lustre
                       │
             ┌─────────┴─────────┐
             ▼                   ▼
          /home               /scratch
             │                   │
             ▼                   ▼
          Quotas              Quotas
```

Quotas can be applied to:

* Users
* Groups

They can control:

* Block/storage usage
* File/inode usage

> **Important:** Quotas must be enabled and properly configured on the Lustre filesystem before `lfs setquota` can be used.

---

# 9.22 User Quota on Home

First identify the filesystem:

```bash
df -hT /home
```

Check the current quota:

```bash
lfs quota -u satish /home
```

For UID:

```bash
lfs quota -u 10001 /home
```

A common administrative example is a **500 GB hard limit**:

```bash
lfs setquota -u satish -b 500G /home
```

However, exact `lfs setquota` syntax can vary by Lustre version.

Always check:

```bash
lfs setquota --help
```

---

# 9.23 User Quota on Scratch

Check the filesystem:

```bash
df -hT /scratch
```

Check current quota:

```bash
lfs quota -u satish /scratch
```

Example: **2 TB hard limit**:

```bash
lfs setquota -u satish -b 2T /scratch
```

Again, verify the syntax supported by the installed Lustre version:

```bash
lfs setquota --help
```

### Important

`/home` and `/scratch` may be separate Lustre filesystems.

Therefore:

```text
/home
  ↓
Quota A

/scratch
  ↓
Quota B
```

Setting a quota on `/home` does not automatically configure `/scratch`.

---

# 9.24 Soft and Hard Limits

Production environments commonly use both soft and hard limits.

Example:

```text
/home

Soft Limit  → 450 GB
Hard Limit  → 500 GB
```

Conceptually:

```text
0 GB ─────────── 450 GB ──────── 500 GB
                 │                │
              Warning          Maximum
```

A common syntax on Lustre versions supporting explicit limit options is:

```bash
lfs setquota -u satish \
--block-softlimit 450G \
--block-hardlimit 500G \
/home
```

For `/scratch`:

```bash
lfs setquota -u satish \
--block-softlimit 1800G \
--block-hardlimit 2T \
/scratch
```

Check the installed version's supported options:

```bash
lfs setquota --help
```

> **Do not blindly copy quota syntax between Lustre versions.** Verify the command options on the actual cluster.

---

# 9.25 Verify Quotas

After setting a quota:

```bash
lfs quota -u satish /home
```

```bash
lfs quota -u satish /scratch
```

Human-readable output:

```bash
lfs quota -h -u satish /home
```

```bash
lfs quota -h -u satish /scratch
```

You should verify:

```text
Filesystem
User
Used blocks
Soft limit
Hard limit
File/inode usage
```

Example conceptual result:

```text
User      Used      Soft       Hard
satish    320G      450G       500G
```

---

# 9.26 Group Quotas

Quotas can also be applied to groups.

Example:

```bash
lfs quota -g hpcusers /scratch
```

Set a group quota using the syntax supported by your Lustre version:

```bash
lfs setquota -g hpcusers -b 10T /scratch
```

Conceptually:

```text
                    /scratch
                       │
                 Group quota
                       │
                    10 TB
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
       user01       user02       user03
```

User and group quota policies should be designed carefully so that limits do not conflict with project requirements.

---

# 9.27 Quota Troubleshooting

## Problem 1 – Quota Command Fails

Check:

```bash
lfs quota -u satish /home
```

If it fails, verify:

```bash
df -hT /home
```

and:

```bash
lfs df -h
```

Then check Lustre services and client logs:

```bash
dmesg | grep -i lustre
```

---

## Problem 2 – User Says They Cannot Write

Check:

```bash
lfs quota -h -u satish /home
```

Look for:

```text
Used >= Soft Limit
Used >= Hard Limit
```

Also check normal filesystem permissions:

```bash
ls -ld /home
ls -ld /home/satish
```

Remember:

```text
Quota problem
      ≠
Permission problem
```

Both must be checked.

---

## Problem 3 – Quota Looks Correct but Write Fails

Check disk/filesystem capacity:

```bash
lfs df -h
```

Check inode/file usage:

```bash
lfs quota -h -u satish /home
```

Also investigate:

```text
Filesystem capacity
OST availability
MDT availability
Client errors
Network connectivity
Filesystem read-only state
```

---

# 9.28 Production Quota Example

Suppose the HPC policy is:

```text
User: satish

/home
  Soft: 450 GB
  Hard: 500 GB

/scratch
  Soft: 1.8 TB
  Hard: 2 TB
```

Conceptual configuration:

```text
                   satish
                     │
          ┌──────────┴──────────┐
          ▼                     ▼
        /home                 /scratch
          │                     │
      450 GB soft            1.8 TB soft
      500 GB hard             2 TB hard
```

Verification:

```bash
lfs quota -h -u satish /home
lfs quota -h -u satish /scratch
```

---

# 9.29 Important Commands

## Check Filesystem

```bash
df -hT /home
df -hT /scratch
lfs df -h
```

## User Quota

```bash
lfs quota -u satish /home
lfs quota -u satish /scratch
```

## Human-readable

```bash
lfs quota -h -u satish /home
lfs quota -h -u satish /scratch
```

## Group Quota

```bash
lfs quota -g hpcusers /scratch
```

## Set Quota

```bash
lfs setquota -u satish -b 500G /home
lfs setquota -u satish -b 2T /scratch
```

## Check Syntax

```bash
lfs setquota --help
```

## Lustre Status

```bash
lfs df -h
mount | grep lustre
dmesg | grep -i lustre
```

---

# 9.30 Quick Revision

### User quota

```text
User
 │
 ├── /home    → independent quota
 │
 └── /scratch → independent quota
```

### Important commands

```bash
lfs quota
lfs setquota
lfs df
df -hT
```

### Troubleshooting order

```text
Quota
  ↓
Filesystem capacity
  ↓
Permissions
  ↓
OST/MDT health
  ↓
Client
  ↓
Network
```

### HPC Engineer Rule

> **Always distinguish between quota, filesystem capacity, and Unix permissions. A user being unable to write does not automatically mean the quota is the problem.**

---

# End of Chapter 9 – Part 3
