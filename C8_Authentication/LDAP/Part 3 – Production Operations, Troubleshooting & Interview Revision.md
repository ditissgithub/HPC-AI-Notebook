## Part 3 – Production Operations, Troubleshooting & Interview Revision

* [8.23 LDAP Production Architecture](#823-ldap-production-architecture)
* [8.24 LDAP High Availability](#824-ldap-high-availability)
* [8.25 SSSD Cache and Offline Authentication](#825-sssd-cache-and-offline-authentication)
* [8.26 Common LDAP Problems](#826-common-ldap-problems)
* [8.27 Common SSSD Problems](#827-common-sssd-problems)
* [8.28 UID/GID Mismatch Scenario](#828-uidgid-mismatch-scenario)
* [8.29 LDAP Replication Troubleshooting](#829-ldap-replication-troubleshooting)
* [8.30 HPC Authentication Troubleshooting Workflow](#830-hpc-authentication-troubleshooting-workflow)
* [8.31 Production Best Practices](#831-production-best-practices)
* [8.32 LDAP Interview Questions](#832-ldap-interview-questions)
* [8.33 Essential LDAP Commands](#833-essential-ldap-commands)
* [8.34 Chapter 8 Final Revision](#834-chapter-8-final-revision)

---

# 8.23 LDAP Production Architecture

A production HPC cluster may contain hundreds or thousands of Linux systems.

A typical architecture:

```text
                         Users
                           │
                           ▼
                      Login Nodes
                           │
                     SSSD / PAM / NSS
                           │
                           ▼
                  ┌──────────────────┐
                  │ LDAP Infrastructure │
                  └─────────┬────────┘
                            │
                 ┌──────────┼──────────┐
                 ▼          ▼          ▼
              LDAP01     LDAP02     LDAP03
                 │          │          │
                 └──────────┼──────────┘
                            │
                       Directory
                            │
              ┌─────────────┼─────────────┐
              ▼             ▼             ▼
          Compute       GPU Nodes      Storage
```

The objective is:

```text
Central Identity
      +
High Availability
      +
Consistent UID/GID
      +
Reliable Authentication
```

---

# 8.24 LDAP High Availability

LDAP authentication becomes a critical cluster service.

If there is only one LDAP server:

```text
All Nodes
    │
    ▼
 LDAP01
    │
    X
 Failure
    │
    ▼
Authentication Problems
```

With multiple servers:

```text
                 Clients
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
       LDAP01               LDAP02
          │                   │
          └─────────┬─────────┘
                    ▼
              LDAP Directory
```

Client configuration can provide multiple LDAP servers or use an appropriate load-balancing/high-availability design.

### Production goal

Avoid:

```text
Single LDAP Server
        =
Single Point of Failure
```

---

# 8.25 SSSD Cache and Offline Authentication

SSSD can cache identity information and, depending on configuration, authentication credentials.

This can help when the LDAP server is temporarily unavailable.

Conceptually:

```text
Normal
User → SSSD → LDAP

LDAP unavailable
User → SSSD Cache
```

Check SSSD status:

```bash id="kj5z0m"
systemctl status sssd
```

Useful command:

```bash id="f6f6lo"
sssctl domain-status
```

Check cached users where supported:

```bash id="6o4jv3"
sssctl user-checks user01
```

Caching should not be treated as a replacement for a highly available LDAP architecture.

---

# 8.26 Common LDAP Problems

## Problem 1 – LDAP Service Down

Check:

```bash id="wy5m8y"
systemctl status slapd
```

Logs:

```bash id="78dz3x"
journalctl -u slapd
```

Check listening port:

```bash id="ynp0h0"
ss -lntp | grep 389
```

---

## Problem 2 – LDAP Port Unreachable

From client:

```bash id="0u6d3m"
nc -zv ldap01 389
```

For LDAPS:

```bash id="44t4pc"
nc -zv ldap01 636
```

If connection fails, investigate:

```text id="1b9wsg"
Network
Firewall
LDAP service
Routing
DNS
```

---

## Problem 3 – LDAP User Missing

Search directly:

```bash id="c7op4e"
ldapsearch -x \
-H ldap://ldap01 \
-b "dc=nsm,dc=in" \
"(uid=user01)"
```

If the user does not exist, investigate the directory or provisioning process.

If it exists, continue to SSSD/NSS troubleshooting.

---

# 8.27 Common SSSD Problems

## SSSD Not Running

```bash id="v4yqyf"
systemctl status sssd
```

Restart after correcting configuration:

```bash id="d4im2c"
systemctl restart sssd
```

---

## Configuration Error

```bash id="nq1nj8"
sssctl config-check
```

Check permissions:

```bash id="brc9fg"
ls -l /etc/sssd/sssd.conf
```

Configuration should be protected because it can contain sensitive information.

---

## User Lookup Failure

```bash id="5o2p0k"
getent passwd user01
```

Then:

```bash id="96z30f"
id user01
```

Check logs:

```bash id="w6h9x0"
journalctl -u sssd
```

Check domain status:

```bash id="m9i0t3"
sssctl domain-status
```

---

# 8.28 UID/GID Mismatch Scenario

### Situation

A user creates a file on `compute01`:

```bash id="d4f6eu"
ls -ln test.dat
```

Output:

```text id="zbyjqp"
-rw------- 1 10001 10001 test.dat
```

On another node, the same username may unexpectedly resolve to another UID.

Check:

```bash id="trg8o5"
id user01
```

Compare:

```bash id="l2d5kc"
getent passwd user01
```

on both nodes.

Expected:

```text id="qz0qec"
compute01 → UID 10001
compute02 → UID 10001
```

If different:

```text id="6u0k4w"
LDAP / SSSD / NSS
        ↓
Identity Resolution Problem
```

This can cause serious shared-storage permission problems.

---

# 8.29 LDAP Replication Troubleshooting

Suppose:

```text id="u8r6m5"
LDAP01 → 175 users
LDAP02 → 179 users
```

Do not assume the application is wrong.

Compare the directory contents.

Example:

```bash id="i5n7i4"
ldapsearch -x -H ldap://ldap01 \
-b "dc=nsm,dc=in" "(objectClass=posixAccount)"
```

Then:

```bash id="qj9k0y"
ldapsearch -x -H ldap://ldap02 \
-b "dc=nsm,dc=in" "(objectClass=posixAccount)"
```

Investigate:

```text id="j4v2u5"
Replication status
Replication agreements
Directory logs
Network connectivity
DNS
Authentication
```

### Production principle

```text
LDAP01
  │
  ├── Directory Data
  │
  ▼
Replication
  │
  ▼
LDAP02
  │
  └── Same Expected State
```

---

# 8.30 HPC Authentication Troubleshooting Workflow

When many users suddenly cannot authenticate:

```text id="f1z2c3"
          Users Cannot Login
                  │
                  ▼
          Check Login Node
                  │
                  ▼
            Check sshd
                  │
                  ▼
             Check SSSD
                  │
                  ▼
             Check LDAP
                  │
                  ▼
         Check LDAP Network
                  │
                  ▼
        Check Replication / DB
```

---

## Scenario: All Nodes Affected

If hundreds of nodes fail simultaneously:

```text id="8u2fbr"
Many Nodes
    ↓
Same Failure
    ↓
Likely Central Service
```

Investigate:

```text id="i4n0l2"
LDAP
DNS
Network
Authentication infrastructure
```

---

## Scenario: One Node Affected

If only one compute node fails:

```text id="v2q0r8"
One Node
   ↓
Likely Local Configuration
```

Check:

```bash id="5vqf43"
systemctl status sssd
cat /etc/nsswitch.conf
cat /etc/sssd/sssd.conf
getent passwd user01
```

---

# 8.31 Production Best Practices

## 1. Centralize Identity

Avoid maintaining independent local users on every cluster node.

Use centralized identity where appropriate.

---

## 2. Maintain UID/GID Consistency

This is essential for shared storage.

```text id="i7dyu7"
Same User
   ↓
Same UID
   ↓
Same GID
```

---

## 3. Use Multiple LDAP Servers

Avoid a single point of failure.

---

## 4. Protect SSSD Configuration

```bash id="d91r2a"
chmod 600 /etc/sssd/sssd.conf
```

Use the permissions appropriate to the installed distribution and authentication setup.

---

## 5. Monitor Authentication Services

Monitor:

```text id="d6wq2x"
slapd
sssd
LDAP connectivity
LDAP replication
DNS
Authentication failures
```

---

## 6. Automate Client Configuration

For large HPC clusters, use automation to maintain:

```text id="w0d9nd"
SSSD configuration
NSS configuration
CA certificates
LDAP server addresses
Authentication configuration
```

This is especially important when managing hundreds or thousands of nodes.

---

## 7. Secure LDAP Communication

Where supported by the environment, use TLS/LDAPS or StartTLS according to the organization's security architecture.

Do not send credentials over an unprotected connection.

---

# 8.32 LDAP Interview Questions

### 1. Why is LDAP used in HPC?

To provide centralized identity and authentication across large numbers of cluster nodes.

### 2. What is `slapd`?

The OpenLDAP server daemon.

### 3. What is SSSD?

A Linux client-side service that integrates identity and authentication providers such as LDAP.

### 4. What is NSS?

The Linux Name Service Switch mechanism used for retrieving users, groups and other identity information.

### 5. What is PAM?

A modular Linux authentication framework.

### 6. How do you verify that an LDAP user is visible to Linux?

```bash id="2stqvk"
getent passwd user01
```

### 7. How do you verify the user identity?

```bash id="5g6b79"
id user01
```

### 8. How do you query LDAP directly?

```bash id="l9kq1g"
ldapsearch -x -H ldap://ldap01 \
-b "dc=nsm,dc=in" "(uid=user01)"
```

### 9. User exists in LDAP but `getent` fails. What do you check?

```text id="e3c6y6"
NSS
SSSD
SSSD configuration
SSSD logs
LDAP connectivity
```

### 10. Why are UID/GID important in HPC?

Because shared filesystem permissions rely on numeric identities.

### 11. How do you troubleshoot an authentication failure?

```text id="9r7x7m"
SSH
 ↓
PAM
 ↓
SSSD
 ↓
NSS
 ↓
LDAP
 ↓
Network
```

Troubleshoot layer by layer.

### 12. Why use LDAP replication?

For availability, resilience and potentially load distribution.

---

# 8.33 Essential LDAP Commands

## Server

```bash id="a7xj3e"
systemctl status slapd
journalctl -u slapd
ss -lntp | grep 389
```

## LDAP Query

```bash id="v4n5yu"
ldapsearch -x \
-H ldap://ldap01 \
-b "dc=nsm,dc=in"
```

## User

```bash id="5r0yp7"
ldapsearch -x \
-H ldap://ldap01 \
-b "dc=nsm,dc=in" \
"(uid=user01)"
```

## Linux Identity

```bash id="4n1t9m"
id user01
getent passwd user01
getent group hpcusers
```

## SSSD

```bash id="nq31cs"
systemctl status sssd
sssctl config-check
sssctl domain-status
```

## Logs

```bash id="6uq2o8"
journalctl -u sssd
journalctl -u sshd
journalctl -u slapd
```

## Network

```bash id="n8r0x7"
nc -zv ldap01 389
nc -zv ldap01 636
```

---

# 8.34 Chapter 8 Final Revision

## LDAP Stack

```text id="qk7g0a"
                 LDAP
                  │
                slapd
                  │
           Directory Data
                  │
                  ▼
                SSSD
             ┌────┴────┐
             ▼         ▼
            NSS       PAM
             │         │
        Identity    Authentication
             │         │
             └────┬────┘
                  ▼
                Linux
```

---

## HPC Authentication Path

```text id="0u7p2c"
User
 ↓
SSH
 ↓
sshd
 ↓
PAM
 ↓
SSSD
 ↓
LDAP
 ↓
Authentication
 ↓
NSS
 ↓
UID/GID
 ↓
Home / Storage
```

---

## Most Important Concepts

```text id="y4z2wq"
LDAP     → Central directory
slapd    → LDAP server
SSSD     → Linux identity client
NSS      → Identity lookup
PAM      → Authentication
UID/GID  → Linux identity
```

---

## Troubleshooting Principle

> **If one node fails, investigate the node. If hundreds of nodes fail simultaneously, investigate the shared infrastructure.**

```text id="3m6n5e"
One Node
   ↓
Local SSSD / NSS / Network

Many Nodes
   ↓
LDAP / DNS / Network / Authentication Infrastructure
```

---

## Chapter 8 Engineer Checklist

* [x] LDAP architecture
* [x] `slapd`
* [x] LDAP directory structure
* [x] Users and groups
* [x] UID/GID
* [x] NSS
* [x] PAM
* [x] SSSD
* [x] SSH authentication
* [x] Shared storage integration
* [x] LDAP replication
* [x] High availability
* [x] SSSD cache
* [x] Troubleshooting
* [x] Production best practices
* [x] Interview questions
* [x] Essential commands

---

# Chapter 8 Complete

**Core mental model:**

```text id="j9w3k4"
LDAP
  ↓
Identity
  ↓
SSSD
  ↓
NSS / PAM
  ↓
Linux
  ↓
HPC Compute / GPU / Storage
```

> **Centralized identity is a foundational service in an HPC cluster because users must have consistent identities and permissions across login nodes, compute nodes, GPU nodes and shared storage.**

---

# End of Chapter 8
