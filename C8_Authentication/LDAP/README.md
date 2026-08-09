# Chapter 8 – LDAP Authentication

## Part 1 – LDAP Architecture, slapd, NSS & SSSD

> **Notebook focus:** Understand how centralized identity and authentication work across an HPC cluster.

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

## Part 2 – Authentication Flow, User Management & HPC Integration

> **Notebook focus:** Understand how LDAP identities become usable Linux accounts across login, compute, GPU, and storage nodes.

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

