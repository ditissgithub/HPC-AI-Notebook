## Part 2 – Authentication Flow, User Management & HPC Integration

* [8.13 LDAP User Lookup](#813-ldap-user-lookup)
* [8.14 LDAP User and Group Management](#814-ldap-user-and-group-management)
* [8.15 UID and GID Consistency](#815-uid-and-gid-consistency)
* [8.16 SSH Authentication Flow](#816-ssh-authentication-flow)
* [8.17 Home Directories in HPC](#817-home-directories-in-hpc)
* [8.18 LDAP and Shared Storage](#818-ldap-and-shared-storage)
* [8.19 LDAP Replication](#819-ldap-replication)
* [8.20 Client-Side Troubleshooting](#820-client-side-troubleshooting)
* [8.21 Authentication Troubleshooting Workflow](#821-authentication-troubleshooting-workflow)
* [8.22 Part 2 Quick Revision](#822-part-2-quick-revision)

---

# 8.13 LDAP User Lookup

A Linux application normally uses NSS rather than directly querying LDAP.

Test:

```bash
getent passwd user01
```

Example:

```text
user01:x:10001:10001:Research User:/home/user01:/bin/bash
```

Check UID/GID:

```bash
id user01
```

Example:

```text
uid=10001(user01) gid=10001(user01) groups=10001(user01),10010(hpcusers)
```

Direct LDAP query:

```bash
ldapsearch -x \
-H ldap://ldap01 \
-b "dc=nsm,dc=in" \
"(uid=user01)"
```

### Important distinction

```text
getent
   ↓
Linux NSS configuration
   ↓
SSSD
   ↓
LDAP

ldapsearch
   ↓
LDAP directly
```

If `ldapsearch` works but `getent` fails, the LDAP server itself may be healthy while the Linux client integration is broken.

---

# 8.14 LDAP User and Group Management

LDAP commonly stores users and groups centrally.

Example:

```text
ou=People
 ├── user01
 ├── user02
 └── user03

ou=Groups
 ├── hpcusers
 ├── gpuusers
 └── research
```

A user may belong to several groups.

Check:

```bash
id user01
```

Check group:

```bash
getent group hpcusers
```

Direct search:

```bash
ldapsearch -x \
-H ldap://ldap01 \
-b "ou=Groups,dc=nsm,dc=in" \
"(cn=hpcusers)"
```

### HPC Use

Groups can be used for:

```text
Storage permissions
Application access
GPU access
Project membership
Administrative privileges
Slurm accounts/projects
```

LDAP itself does not determine Slurm scheduling policy; Slurm's account/QoS configuration handles workload policy.

---

# 8.15 UID and GID Consistency

This is one of the most important concepts in HPC.

Suppose:

```text
user01
UID = 10001
```

The same UID should resolve consistently across cluster nodes.

```text
Login01   → UID 10001
Compute01 → UID 10001
Compute02 → UID 10001
GPU01     → UID 10001
```

Why?

Because Linux filesystem permissions are based primarily on numeric UID/GID values.

Example:

```bash
ls -ln /home/user01
```

Output may show:

```text
10001 10001
```

If one node incorrectly maps `user01` to another UID, file ownership can appear wrong.

### HPC identity principle

```text
Central Identity
      ↓
Consistent UID/GID
      ↓
Consistent Permissions
      ↓
Correct Shared Storage Access
```

---

# 8.16 SSH Authentication Flow

Consider:

```bash
ssh user01@login01
```

Simplified flow:

```text
                SSH Client
                    │
                    ▼
                  sshd
                    │
                    ▼
                   PAM
                    │
                    ▼
                  SSSD
                    │
                    ▼
                  LDAP
                    │
             ┌──────┴──────┐
             ▼             ▼
          Identity      Password
          Lookup        Validation
             │             │
             └──────┬──────┘
                    ▼
              Authentication
                    │
                    ▼
                 Session
```

After authentication, Linux needs identity information:

```bash
id user01
```

NSS/SSSD supplies:

```text
UID
GID
Groups
Home directory
Shell
```

---

# 8.17 Home Directories in HPC

LDAP can provide the user's home-directory path:

```text
/home/user01
```

But LDAP does **not** itself provide the filesystem.

A common HPC architecture is:

```text
LDAP
 │
 └── User identity
        │
        ▼
      Login Node
        │
        ▼
   Shared Filesystem
        │
        ▼
    /home/user01
```

The home directory may be provided through NFS, Lustre, or another shared storage system depending on cluster design.

Check:

```bash
getent passwd user01
```

Then:

```bash
ls -ld /home/user01
```

Check mounts:

```bash
mount | grep /home
```

---

# 8.18 LDAP and Shared Storage

LDAP and shared storage solve different problems.

```text
LDAP
 ↓
Who is the user?

Storage
 ↓
Where is the user's data?
```

Example:

```text
                    LDAP
                      │
               UID/GID Identity
                      │
                      ▼
                Compute Node
                      │
                      ▼
                  Lustre
                      │
                      ▼
                User Files
```

For correct access:

```text
LDAP identity
      +
Consistent UID/GID
      +
Filesystem permissions
      =
Correct data access
```

This is especially important for Lustre and other shared filesystems.

---

# 8.19 LDAP Replication

Production HPC environments may use multiple LDAP servers.

Example:

```text
             LDAP Clients
                   │
          ┌────────┴────────┐
          ▼                 ▼
      LDAP01             LDAP02
      Master              Replica
```

Or a multi-provider design:

```text
          ┌──────────────┐
          │ LDAP Cluster │
          └──────┬───────┘
                 │
        ┌────────┼────────┐
        ▼        ▼        ▼
      LDAP01   LDAP02   LDAP03
```

Benefits:

* High availability
* Load distribution
* Reduced single point of failure
* Better resilience

### Production consideration

Replication must be monitored.

A user may appear on one server but not another if replication is unhealthy.

Useful checks:

```bash
ldapsearch -x -H ldap://ldap01 \
-b "dc=nsm,dc=in" "(uid=user01)"
```

and compare:

```bash
ldapsearch -x -H ldap://ldap02 \
-b "dc=nsm,dc=in" "(uid=user01)"
```

---

# 8.20 Client-Side Troubleshooting

## Problem: User Not Found

Test:

```bash
getent passwd user01
```

If nothing is returned:

```bash
id user01
```

Then check:

```bash
systemctl status sssd
```

Check:

```bash
cat /etc/nsswitch.conf
```

Then:

```bash
journalctl -u sssd
```

---

## Problem: LDAP Direct Query Works but `getent` Fails

Example:

```text
ldapsearch → works
getent passwd user01 → fails
```

Focus on:

```text
NSS
 ↓
SSSD
 ↓
SSSD configuration
```

Check:

```bash
sssctl config-check
```

If supported on the installed SSSD version:

```bash
sssctl domain-list
```

---

## Problem: Authentication Fails

Check:

```bash
systemctl status sssd
```

```bash
journalctl -u sssd
```

```bash
journalctl -u sshd
```

Check user:

```bash
id user01
```

If identity lookup itself fails, authentication troubleshooting should start with SSSD/LDAP rather than the password.

---

# 8.21 Authentication Troubleshooting Workflow

Use this sequence:

```text
User Cannot Login
       │
       ▼
Can SSH Reach Server?
       │
       ▼
Is sshd Running?
       │
       ▼
Does `id user` Work?
       │
       ├── No
       │    ↓
       │   NSS / SSSD / LDAP
       │
       └── Yes
            ↓
         PAM / Authentication
            ↓
         Home Directory
            ↓
         Shell / Session
```

### Step 1 – Network

```bash
ping ldap01
```

Better for service connectivity:

```bash
nc -zv ldap01 389
```

For LDAPS:

```bash
nc -zv ldap01 636
```

### Step 2 – LDAP

```bash
ldapsearch -x \
-H ldap://ldap01 \
-b "dc=nsm,dc=in" \
"(uid=user01)"
```

### Step 3 – SSSD

```bash
systemctl status sssd
```

```bash
sssctl config-check
```

### Step 4 – NSS

```bash
getent passwd user01
```

### Step 5 – Identity

```bash
id user01
```

### Step 6 – SSH/PAM

```bash
journalctl -u sshd
```

---

# 8.22 Part 2 Quick Revision

## Identity

```text
LDAP
 ↓
User / Group
 ↓
UID / GID
```

## Linux Integration

```text
NSS → Identity Lookup
PAM → Authentication
SSSD → Client Integration
```

## HPC Storage

```text
LDAP
 ↓
Consistent UID/GID
 ↓
Filesystem Permissions
 ↓
Shared Storage
```

## Troubleshooting

```text
Network
 ↓
LDAP
 ↓
SSSD
 ↓
NSS
 ↓
PAM
 ↓
SSH
 ↓
Home Directory
```

## Essential Commands

```bash
ldapsearch
getent passwd <user>
getent group <group>
id <user>

systemctl status sssd
sssctl config-check

journalctl -u sssd
journalctl -u sshd

cat /etc/nsswitch.conf
cat /etc/sssd/sssd.conf
```

---

# HPC Engineer Mental Model

When an HPC user says:

> **"I cannot log in."**

Do not immediately reset the password.

Think:

```text
Can I reach the login node?
        ↓
Is SSH working?
        ↓
Can Linux find the user?
        ↓
Is SSSD working?
        ↓
Can SSSD reach LDAP?
        ↓
Does LDAP contain the user?
        ↓
Does PAM authenticate?
        ↓
Is the home directory available?
```

That layered troubleshooting approach is much more valuable in production than memorizing individual LDAP commands.

---

# End of Part 2
