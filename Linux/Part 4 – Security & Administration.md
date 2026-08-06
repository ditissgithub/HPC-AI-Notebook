# Part 4 – Security & Administration

- [2.29 Linux User & Group Management](#229-linux-user--group-management)
- [2.30 File Ownership & Permissions](#230-file-ownership--permissions)
- [2.31 Access Control Lists (ACL)](#231-access-control-lists-acl)
- [2.32 SELinux](#232-selinux)
- [2.33 SSH Administration](#233-ssh-administration)
- [2.34 Package Management](#234-package-management)
- [2.35 Log Management](#235-log-management)
- [2.36 Cron & systemd Timers](#236-cron--systemd-timers)
- [2.37 Linux Administration Checklist](#237-linux-administration-checklist)
- [Production Insight](#production-insight)
- [Key Takeaways](#key-takeaways)

---

# 2.29 Linux User & Group Management

## Overview

Linux is a multi-user operating system.

Every user belongs to one or more groups.

Authentication and authorization are based on:

- User ID (UID)
- Group ID (GID)
- File permissions
- Access policies

In HPC environments, users are typically managed centrally using **LDAP/SSSD**, while local accounts are reserved for system administration and emergency access.

---

## User Architecture

```
                Linux System

                      │

      ┌───────────────┼───────────────┐

      ▼               ▼               ▼

   User          Primary Group    Home Directory
```

---

## Important Files

| File | Purpose |
|------|----------|
| `/etc/passwd` | User information |
| `/etc/shadow` | Encrypted passwords |
| `/etc/group` | Group information |
| `/etc/gshadow` | Group passwords |

---

## Common User Commands

Create user

```bash
useradd satish
```

Set password

```bash
passwd satish
```

Delete user

```bash
userdel satish
```

Delete user with home directory

```bash
userdel -r satish
```

Modify user

```bash
usermod
```

Lock account

```bash
usermod -L satish
```

Unlock account

```bash
usermod -U satish
```

---

## Group Commands

Create group

```bash
groupadd developers
```

Delete group

```bash
groupdel developers
```

Add user to group

```bash
usermod -aG developers satish
```

View user groups

```bash
groups satish
```

Current user

```bash
id
```

Another user

```bash
id satish
```

---

## Best Practices

- Avoid direct root login.
- Use groups to manage permissions.
- Disable inactive accounts.
- Use centralized authentication (LDAP/SSSD) in production.

---

# 2.30 File Ownership & Permissions

Every file has:

- Owner
- Group
- Permissions

Example

```bash
ls -l
```

Output

```text
-rwxr-x--- 1 root developers 1024 app.sh
```

---

## Permission Format

```
-rwxr-xr--

│

├── File Type

├── Owner

├── Group

└── Others
```

---

## Permission Values

| Permission | Value |
|------------|------:|
| Read (r) | 4 |
| Write (w) | 2 |
| Execute (x) | 1 |

Examples

| Octal | Meaning |
|-------|----------|
| 777 | Full access |
| 755 | Standard executable |
| 750 | Owner full, group read/execute |
| 700 | Owner only |
| 644 | Standard file |
| 600 | Private file |

---

## Change Permissions

```bash
chmod 755 file.sh
```

Symbolic mode

```bash
chmod u+x file.sh
```

Remove write

```bash
chmod g-w file.sh
```

---

## Change Ownership

Owner

```bash
chown satish file.txt
```

Owner and group

```bash
chown satish:developers file.txt
```

Group

```bash
chgrp developers file.txt
```

---

## Special Permissions

### SUID

```
-rwsr-xr-x
```

User executes with owner's privileges.

Example

```bash
passwd
```

---

### SGID

```
-rwxr-sr-x
```

Common on shared project directories.

---

### Sticky Bit

```
drwxrwxrwt
```

Example

```
/tmp
```

Users can only delete their own files.

---

# 2.31 Access Control Lists (ACL)

Traditional permissions are sometimes insufficient.

ACLs provide fine-grained access control.

---

## Example

```
Owner

↓

Group

↓

Other

↓

Additional Users
```

---

## View ACL

```bash
getfacl file.txt
```

Set ACL

```bash
setfacl -m u:alice:rwx file.txt
```

Remove ACL

```bash
setfacl -x u:alice file.txt
```

Remove all ACLs

```bash
setfacl -b file.txt
```

---

## HPC Use Case

Research projects often require temporary access to shared datasets without changing the primary ownership of files. ACLs allow administrators to grant permissions to individual users while preserving existing ownership and group settings.

---

# 2.32 SELinux

## What is SELinux?

Security-Enhanced Linux (SELinux) provides mandatory access control (MAC).

Even if standard Linux permissions allow access, SELinux policies can still deny operations.

---

## Security Layers

```
User

↓

Linux Permissions

↓

SELinux Policy

↓

Resource
```

---

## SELinux Modes

| Mode | Description |
|------|-------------|
| Enforcing | Policies are enforced |
| Permissive | Violations logged only |
| Disabled | SELinux inactive |

---

## Check Status

```bash
getenforce
```

Detailed information

```bash
sestatus
```

---

## Change Mode (Temporary)

Permissive

```bash
setenforce 0
```

Enforcing

```bash
setenforce 1
```

---

## View File Context

```bash
ls -Z
```

Restore context

```bash
restorecon -Rv /path
```

---

## Best Practice

Never disable SELinux permanently to fix an issue.

Instead:

- Read audit logs.
- Identify the policy violation.
- Correct labels or policies.

---

# 2.33 SSH Administration

SSH is the primary remote administration protocol for Linux servers.

---

## SSH Workflow

```
Administrator

↓

SSH Client

↓

SSH Server (sshd)

↓

Linux System
```

---

## Important Configuration

```
/etc/ssh/sshd_config
```

---

## Common Commands

Connect

```bash
ssh user@server
```

Copy file

```bash
scp file.txt server:/tmp
```

Copy directory

```bash
scp -r project server:/data
```

Remote execution

```bash
ssh server hostname
```

Generate key

```bash
ssh-keygen
```

Copy key

```bash
ssh-copy-id user@server
```

---

## Restart SSH

```bash
systemctl restart sshd
```

---

## Verify Configuration

```bash
sshd -t
```

---

## Security Best Practices

- Disable password authentication where possible.
- Use SSH keys.
- Disable direct root login.
- Restrict access with firewalls.
- Monitor authentication logs.

---

# 2.34 Package Management

Enterprise Linux distributions use RPM packages.

---

## DNF

Install package

```bash
dnf install vim
```

Remove package

```bash
dnf remove vim
```

Update

```bash
dnf update
```

Search

```bash
dnf search slurm
```

Package information

```bash
dnf info openssh
```

Installed packages

```bash
dnf list installed
```

---

## RPM

Install

```bash
rpm -ivh package.rpm
```

Upgrade

```bash
rpm -Uvh package.rpm
```

Query

```bash
rpm -qa
```

Verify

```bash
rpm -V package
```

Find package owning file

```bash
rpm -qf /usr/bin/bash
```

---

# 2.35 Log Management

Logs are essential for troubleshooting.

---

## Common Log Locations

| Directory | Purpose |
|-----------|----------|
| `/var/log/messages` | General system logs |
| `/var/log/secure` | Authentication |
| `/var/log/cron` | Scheduled jobs |
| `/var/log/dmesg` | Kernel messages |
| `/var/log/audit/` | SELinux and audit logs |

> **Note:** On systems using `systemd`, many logs are also stored in the system journal and accessed with `journalctl`.

---

## Useful Commands

View logs

```bash
less /var/log/messages
```

Follow logs

```bash
tail -f /var/log/messages
```

Kernel logs

```bash
dmesg
```

Search logs

```bash
grep ERROR /var/log/messages
```

Journal

```bash
journalctl
```

---

## Log Rotation

Linux uses **logrotate** to prevent log files from consuming excessive disk space.

Configuration

```
/etc/logrotate.conf
```

Directory

```
/etc/logrotate.d/
```

---

# 2.36 Cron & systemd Timers

Automation is fundamental in Linux administration.

---

## Cron

Edit current user's crontab

```bash
crontab -e
```

View

```bash
crontab -l
```

System cron

```
/etc/crontab
```

---

## Cron Format

```
* * * * *

│ │ │ │ │

│ │ │ │ └── Day of Week

│ │ │ └──── Month

│ │ └────── Day

│ └──────── Hour

└────────── Minute
```

Example

```cron
0 2 * * * /opt/scripts/backup.sh
```

Runs daily at 02:00.

---

## systemd Timers

Modern alternative to cron.

Advantages:

- Better logging
- Dependency management
- Missed job handling
- Integration with systemd

Useful commands

```bash
systemctl list-timers
```

View timer

```bash
systemctl status backup.timer
```

---

# 2.37 Linux Administration Checklist

Daily checks for production systems.

---

## System Health

```bash
uptime

free -h

df -h

top
```

---

## Services

```bash
systemctl --failed

systemctl status sshd
```

---

## Networking

```bash
ip addr

ss -tulpn
```

---

## Storage

```bash
lsblk

mount

findmnt
```

---

## Logs

```bash
journalctl -p err

dmesg
```

---

## Security

```bash
getenforce

last

lastlog
```

---

## Hardware

```bash
lscpu

lspci

dmidecode
```

---

# Production Insight

Linux administration in HPC environments extends far beyond managing a single server. Administrators are responsible for hundreds or thousands of systems that support provisioning, scheduling, authentication, storage, and networking services. Consistent user management, secure remote access, package maintenance, log analysis, and automation form the operational backbone of every production HPC cluster.

---

# Key Takeaways

- Linux is a multi-user operating system with structured user and group management.
- File permissions and ownership provide the first layer of access control.
- ACLs offer fine-grained permission management beyond traditional UNIX permissions.
- SELinux provides mandatory access control and should be configured rather than disabled.
- SSH is the standard method for secure remote administration.
- DNF and RPM are the primary package management tools on enterprise Linux systems.
- Logs and the system journal are the primary sources for troubleshooting.
- Cron and systemd timers automate recurring administrative tasks.
- Routine system health checks help identify issues before they impact production workloads.

---

## Next Part

**Part 5 – Production Operations**

Topics covered:

- Linux Networking Basics
- Production Troubleshooting Methodology
- HPC Linux Best Practices
- Essential Linux Commands
- Linux Interview Questions
- Chapter Summary
