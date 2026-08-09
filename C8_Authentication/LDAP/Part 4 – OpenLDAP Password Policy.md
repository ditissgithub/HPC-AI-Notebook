## Part 4 – OpenLDAP Password Policy (ppolicy)

> **Notebook focus:** Understand the OpenLDAP Password Policy overlay and the operational controls commonly used in production HPC environments.

---

# 8.35 OpenLDAP ppolicy

OpenLDAP **ppolicy** provides password-policy controls for LDAP accounts.

It can enforce policies such as:

* Password expiration
* Minimum password age
* Password history
* Maximum failed login attempts
* Account lockout
* Password quality requirements
* Grace logins after password expiration

Conceptually:

```text
                    LDAP
                     │
                  slapd
                     │
              ppolicy overlay
                     │
        ┌────────────┼────────────┐
        ▼            ▼            ▼
   Password Age   Failures     Expiration
        │            │            │
        └────────────┼────────────┘
                     ▼
              Authentication
```

---

# 8.36 Why ppolicy Matters in HPC

An HPC cluster may have thousands of user accounts.

Without centralized password policy:

```text
User01 → Policy A
User02 → Policy B
User03 → No Policy
```

With LDAP ppolicy:

```text
                    LDAP
                     │
              Central Policy
                     │
       ┌─────────────┼─────────────┐
       ▼             ▼             ▼
   Login Nodes   Compute Nodes   GPU Nodes
```

The important principle is:

> **Authentication policy should be centralized rather than manually maintained on individual HPC nodes.**

---

# 8.37 Common ppolicy Controls

Typical policy attributes include:

| Policy               | Purpose                                 |
| -------------------- | --------------------------------------- |
| `pwdMaxAge`          | Maximum password lifetime               |
| `pwdMinAge`          | Minimum password lifetime               |
| `pwdInHistory`       | Number of previous passwords remembered |
| `pwdMaxFailure`      | Failed attempts before lockout          |
| `pwdLockout`         | Enables account lockout                 |
| `pwdLockoutDuration` | Lockout duration                        |
| `pwdGraceAuthnLimit` | Allowed grace authentications           |
| `pwdCheckQuality`    | Password quality checking               |

Exact behavior depends on the OpenLDAP version and configured overlay/policy.

---

# 8.38 ppolicy Architecture

The ppolicy overlay is attached to the LDAP server.

Conceptually:

```text
LDAP Client
     │
     ▼
   SSSD
     │
     ▼
   slapd
     │
     ▼
 ppolicy overlay
     │
     ▼
LDAP Directory
```

The policy is enforced by the LDAP server rather than independently by every HPC node.

---

# 8.39 Check ppolicy Configuration

The exact configuration depends on the OpenLDAP deployment.

A common approach is to inspect the dynamic configuration:

```bash
ldapsearch -Y EXTERNAL -H ldapi:/// \
-b cn=config \
'(objectClass=olcOverlayConfig)'
```

Look specifically for:

```text
olcOverlay=ppolicy
```

Example conceptual structure:

```text
cn=config
 └── olcDatabase
      └── olcOverlay=ppolicy
```

---

# 8.40 Check Password Policy

A policy object can be queried using `ldapsearch`.

For example:

```bash
ldapsearch -x \
-H ldap://ldap01 \
-b "ou=Policies,dc=nsm,dc=in"
```

The actual DN and schema used depend on the deployment.

Do **not** assume that every OpenLDAP installation uses the same policy DN.

---

# 8.41 Account Lockout

A common policy is:

```text
Maximum failed attempts
        ↓
Account locked
        ↓
Administrator / policy-based recovery
```

Example conceptual configuration:

```text
pwdMaxFailure: 5
pwdLockout: TRUE
pwdLockoutDuration: 900
```

Meaning conceptually:

```text
5 failed attempts
        ↓
15-minute lockout
```

The actual values should be defined according to organizational security policy.

---

# 8.42 Password Expiration

A password can have a maximum lifetime.

Conceptually:

```text
Password Created
       │
       ▼
   Valid Period
       │
       ▼
 Password Expires
       │
       ▼
Password Change Required
```

A common policy attribute is:

```text
pwdMaxAge
```

Check user password-policy state using the appropriate LDAP attributes/tools for the deployed schema.

---

# 8.43 HPC Authentication Flow with ppolicy

The complete authentication model becomes:

```text
User
 │
 ▼
SSH
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
 ▼
slapd
 │
 ▼
ppolicy
 │
 ├── Password Valid?
 ├── Password Expired?
 ├── Account Locked?
 └── Policy Violation?
 │
 ▼
Authentication Result
```

This is useful to remember during troubleshooting.

---

# 8.44 Troubleshooting ppolicy

### User cannot authenticate

First determine whether the problem is:

```text
Identity lookup
       OR
Authentication
       OR
Password policy
```

Check:

```bash
id user01
```

If this works, identity lookup is probably functioning.

Then inspect:

```bash
journalctl -u sssd
journalctl -u sshd
```

On the LDAP server:

```bash
journalctl -u slapd
```

Look for indications of:

```text
Password expired
Account locked
Invalid credentials
Policy violation
```

---

# 8.45 Important Production Consideration

Do not immediately unlock an account just because authentication fails.

First determine:

```text
Why was it locked?
        ↓
Repeated failed login?
        ↓
Password expiration?
        ↓
Possible security incident?
```

Then apply the organization's account-recovery procedure.

---

# 8.46 ppolicy + SSSD Mental Model

Remember the separation:

```text
OpenLDAP ppolicy
        │
        ▼
Server-side password policy

SSSD
        │
        ▼
Linux client integration

PAM
        │
        ▼
Authentication framework

NSS
        │
        ▼
Identity lookup
```

They perform different jobs.

---

# 8.47 Advanced Interview Questions

### Q1. What is OpenLDAP ppolicy?

A password-policy overlay for OpenLDAP that provides controls such as password expiration, history and account lockout.

### Q2. Where is ppolicy enforced?

On the LDAP server through the OpenLDAP ppolicy overlay.

### Q3. Does SSSD replace LDAP ppolicy?

No.

SSSD is primarily the Linux client-side identity/authentication integration layer; LDAP ppolicy provides server-side password-policy controls.

### Q4. A user exists in LDAP but cannot log in. What do you check?

```text
getent passwd user01
        ↓
SSSD
        ↓
PAM
        ↓
LDAP
        ↓
ppolicy
        ↓
Password expiration / lockout
```

### Q5. Why is ppolicy useful in HPC?

Because centralized password policy avoids inconsistent authentication rules across a large number of cluster nodes.

---

# 8.48 ppolicy Quick Revision

```text
ppolicy
   │
   ├── Password expiration
   ├── Password history
   ├── Failed-login limits
   ├── Account lockout
   └── Password quality
```

### Remember

```text
slapd   → LDAP Server
ppolicy → Password Policy
SSSD    → Linux Client Integration
PAM     → Authentication Framework
NSS     → Identity Lookup
```

---

# Chapter 8 – Advanced Checklist

* [x] LDAP architecture
* [x] slapd
* [x] NSS
* [x] PAM
* [x] SSSD
* [x] User/group management
* [x] UID/GID consistency
* [x] LDAP replication
* [x] Authentication troubleshooting
* [x] High availability
* [x] OpenLDAP ppolicy
* [x] Password expiration
* [x] Password history
* [x] Account lockout
* [x] Production considerations

---

> **HPC Engineer takeaway:** LDAP provides centralized identity, SSSD integrates that identity with Linux, PAM handles authentication, NSS handles identity lookups, and **OpenLDAP ppolicy controls password-policy behavior at the directory layer**.

# End of Chapter 8 – Part 4
