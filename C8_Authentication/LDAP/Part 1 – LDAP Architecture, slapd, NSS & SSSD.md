## Part 1 – LDAP Architecture, slapd, NSS & SSSD

* [8.1 Why LDAP Is Used in HPC](#81-why-ldap-is-used-in-hpc)
* [8.2 LDAP Architecture](#82-ldap-architecture)
* [8.3 LDAP Directory Structure](#83-ldap-directory-structure)
* [8.4 Important LDAP Objects](#84-important-ldap-objects)
* [8.5 slapd](#85-slapd)
* [8.6 LDAP Client Architecture](#86-ldap-client-architecture)
* [8.7 NSS](#87-nss)
* [8.8 PAM](#88-pam)
* [8.9 SSSD](#89-sssd)
* [8.10 LDAP Authentication Flow](#810-ldap-authentication-flow)
* [8.11 Important Commands](#811-important-commands)
* [8.12 HPC LDAP Mental Model](#812-hpc-ldap-mental-model)


# 8.1 Why LDAP Is Used in HPC

An HPC cluster may contain:

```text
Login Nodes
Compute Nodes
GPU Nodes
Management Nodes
Storage Nodes
```

Managing local users independently on every node is difficult.

Without centralized authentication:

```text
Node01 → user database
Node02 → user database
Node03 → user database
...
Node1000 → user database
```

LDAP provides a centralized directory:

```text
                    LDAP
                      │
       ┌──────────────┼──────────────┐
       ▼              ▼              ▼
   Login Node      Compute Node    GPU Node
```

The same user identity can therefore be recognized across the cluster.

---

# 8.2 LDAP Architecture

A simplified HPC architecture:

```text
                    Users
                      │
                      ▼
                 Login Node
                      │
              NSS / PAM / SSSD
                      │
                      ▼
                    LDAP
                      │
              ┌───────┴───────┐
              ▼               ▼
           slapd            LDAP DB
```

LDAP commonly provides:

* User information
* Group information
* UID/GID
* Authentication-related directory information
* Centralized identity management

---

# 8.3 LDAP Directory Structure

LDAP uses a hierarchical directory.

Example:

```text
dc=nsm,dc=in
│
├── ou=People
│   ├── uid=user01
│   ├── uid=user02
│   └── uid=user03
│
└── ou=Groups
    ├── cn=research
    ├── cn=hpcusers
    └── cn=admins
```

A user DN might be:

```text
uid=user01,ou=People,dc=nsm,dc=in
```

The DN uniquely identifies an LDAP object.

---

# 8.4 Important LDAP Objects

Common attributes include:

| Attribute       | Purpose                             |
| --------------- | ----------------------------------- |
| `uid`           | Username                            |
| `uidNumber`     | Linux UID                           |
| `gidNumber`     | Primary GID                         |
| `cn`            | Common name                         |
| `homeDirectory` | User home                           |
| `loginShell`    | Login shell                         |
| `userPassword`  | Password attribute where applicable |

Example conceptual user:

```text
uid: satish
uidNumber: 10001
gidNumber: 10001
homeDirectory: /home/satish
loginShell: /bin/bash
```

The UID/GID mapping is particularly important in HPC because shared filesystems depend heavily on consistent identities.

---

# 8.5 slapd

`slapd` is the OpenLDAP server daemon.

Check service:

```bash
systemctl status slapd
```

Start:

```bash
systemctl start slapd
```

Enable:

```bash
systemctl enable slapd
```

Check listening ports:

```bash
ss -lntp | grep -E '389|636'
```

Common LDAP ports:

```text
389 → LDAP
636 → LDAPS
```

---

## Check LDAP Server Logs

Depending on the distribution and logging configuration:

```bash
journalctl -u slapd
```

Follow logs:

```bash
journalctl -fu slapd
```

---

# 8.6 LDAP Client Architecture

A Linux HPC node normally does not communicate with LDAP directly for every application.

Instead:

```text
Application
    │
    ▼
NSS / PAM
    │
    ▼
SSSD
    │
    ▼
LDAP Server
```

This separation is important.

---

# 8.7 NSS

NSS means **Name Service Switch**.

It determines where Linux obtains information such as:

* Users
* Groups
* Hosts
* Services

Check:

```bash
cat /etc/nsswitch.conf
```

Example:

```text
passwd:     files sss
group:      files sss
shadow:     files
```

Meaning:

```text
files → Local /etc files
sss   → SSSD
```

Test user lookup:

```bash
getent passwd user01
```

Test groups:

```bash
getent group hpcusers
```

If LDAP users appear through `getent`, the NSS path is working.

---

# 8.8 PAM

PAM means **Pluggable Authentication Modules**.

PAM controls authentication-related operations such as login.

Conceptually:

```text
User
 ↓
SSH
 ↓
PAM
 ↓
SSSD
 ↓
LDAP
```

PAM configuration depends on the operating system and authentication stack.

On modern RHEL-family systems, inspect:

```bash
authselect current
```

Avoid manually modifying generated authentication configuration unless you understand the consequences.

---

# 8.9 SSSD

SSSD means **System Security Services Daemon**.

It commonly acts as the client-side identity and authentication layer.

Check:

```bash
systemctl status sssd
```

Configuration:

```bash
cat /etc/sssd/sssd.conf
```

Permissions are important:

```bash
ls -l /etc/sssd/sssd.conf
```

Typically:

```text
root:root
0600
```

Check logs:

```bash
ls /var/log/sssd/
```

---

## SSSD Mental Model

```text
Linux
 │
 ├── NSS ───────► User/Group Lookup
 │
 └── PAM ───────► Authentication
          │
          ▼
         SSSD
          │
          ▼
         LDAP
```

SSSD can also cache identity information, which can be important when LDAP connectivity is temporarily unavailable.

---

# 8.10 LDAP Authentication Flow

A simplified SSH login:

```text
User
 │
 │ ssh user01@login01
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
 ├── User exists?
 ├── Password valid?
 └── UID/GID information
 │
 ▼
Authentication Success
 │
 ▼
NSS
 │
 ▼
Home Directory / Group Information
 │
 ▼
Shell
```

For an HPC cluster:

```text
                 LDAP
                   │
       ┌───────────┼───────────┐
       ▼           ▼           ▼
    Login01     Compute01    GPU01
       │           │           │
      SSSD        SSSD        SSSD
```

This allows centralized identity across many nodes.

---

# 8.11 Important Commands

## LDAP Server

```bash
systemctl status slapd
journalctl -u slapd
ss -lntp | grep 389
```

## LDAP Query

```bash
ldapsearch -x -H ldap://ldap01 \
-b "dc=nsm,dc=in"
```

Search for a user:

```bash
ldapsearch -x \
-H ldap://ldap01 \
-b "dc=nsm,dc=in" \
"(uid=user01)"
```

---

## Client

```bash
systemctl status sssd
systemctl restart sssd
```

Check identity:

```bash
id user01
```

```bash
getent passwd user01
```

```bash
getent group hpcusers
```

Check NSS:

```bash
cat /etc/nsswitch.conf
```

---

## SSH Authentication

```bash
ssh user01@login01
```

If authentication fails, investigate:

```text
sshd
 ↓
PAM
 ↓
SSSD
 ↓
LDAP
```

Check:

```bash
journalctl -u sshd
journalctl -u sssd
```

---

# 8.12 HPC LDAP Mental Model

Remember the four important components:

```text
LDAP
 ↓
Directory / Identity Data

slapd
 ↓
LDAP Server

SSSD
 ↓
Linux LDAP Client

NSS / PAM
 ↓
Linux Identity + Authentication Integration
```

The complete HPC model:

```text
                     LDAP Server
                          │
                    Central Identity
                          │
             ┌────────────┼────────────┐
             ▼            ▼            ▼
          Login01      Compute01      GPU01
             │            │            │
            SSSD         SSSD         SSSD
             │            │            │
          NSS/PAM      NSS/PAM      NSS/PAM
             │            │            │
             ▼            ▼            ▼
          Linux         Linux        Linux
```

### Key Engineering Principle

> **LDAP provides centralized identity; SSSD connects Linux systems to that identity service; NSS provides identity lookups; PAM handles authentication.**

---

# Part 1 Quick Revision

```text
slapd  → LDAP server
SSSD   → Linux identity/authentication client
NSS    → User/group lookup
PAM    → Authentication framework
```

Essential commands:

```bash
systemctl status slapd
systemctl status sssd

ldapsearch
getent passwd <user>
getent group <group>
id <user>

cat /etc/nsswitch.conf
cat /etc/sssd/sssd.conf

journalctl -u slapd
journalctl -u sssd
journalctl -u sshd
```

---

# End of Part 1
