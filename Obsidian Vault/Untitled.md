# Vulnerability Assessment Report - VulnMail (Roundcube Mail Server)

**Target:** `10.176.121.111` (`vulnmail.local`)  
**Assessment Type:** Authorized CTF / Lab Enumeration  
**Date:** 04-Aug-2026

---

# Objective

Perform reconnaissance and enumeration against the target mail server to identify:

- Running services
- Web technologies
- Mail services
- Valid users
- Potential attack vectors
- Sensitive information exposure

---

# Phase 1 - Service Enumeration

## Nmap Scan

Command used:

```bash
nmap -Pn -sV -sC -T4 -oN nmap.txt 10.176.121.111
```

### Results

| Port | Service | Version |
|------|---------|---------|
| 22 | SSH | OpenSSH 9.1 |
| 25 | SMTP | Postfix |
| 80 | HTTP | Apache 2.4.41 |
| 143 | IMAP | Dovecot |
| 993 | IMAPS | tcpwrapped |
| 3306 | MariaDB | 10.3.39 |

---

## Observations

- Mail server is running Postfix + Dovecot.
- Webmail is hosted on Apache.
- MySQL/MariaDB is accessible externally.
- Hostname discovered:

```
vulnmail.local
```

---

# Phase 2 - Web Enumeration

Browsing:

```
http://10.176.121.111/roundcube/
```

The application was identified as:

```
Roundcube Webmail
```

JavaScript exposed:

```
rcversion = 10509
```

Identified version:

```
Roundcube 1.5.9
```

---

# Configuration Findings

Observed PHP warning:

```
Use of undefined constant RCUBE_INSTALL_PATH
```

Also noticed:

```
config.inc.php
```

appeared empty/misconfigured during testing.

This suggests:

- incomplete installation
- configuration issue
- intentional CTF misconfiguration

---

# Directory Enumeration

Gobuster discovered SQL directories:

```
/SQL/mysql/
/SQL/postgres/
/SQL/oracle/
/SQL/mssql/
```

These returned HTTP 403.

Additional files:

```
.htaccess
.htpasswd
.htaccess.old
.htaccess.bak
```

also returned 403 Forbidden.

No direct file disclosure was observed.

---

# Log File Discovery

Accessible log:

```
/roundcube/logs/errors.log
```

Contents included:

- IMAP authentication failures
- Database errors
- Session creation errors

---

# IMAP Enumeration

Connected manually:

```bash
nc 10.176.121.111 143
```

Banner:

```
Dovecot ready.
```

Manual authentication attempt:

```
LOGIN admin admin
```

Response:

```
AUTHENTICATIONFAILED
```

Result:

- IMAP service reachable
- Authentication enforced
- Credentials invalid

---

# SMTP Enumeration

Banner:

```
220 vulnmail.local ESMTP Postfix
```

EHLO revealed:

```
PIPELINING
SIZE
VRFY
AUTH PLAIN
AUTH LOGIN
STARTTLS
```

Important finding:

```
VRFY enabled
```

---

# User Enumeration

Tool:

```bash
smtp-user-enum
```

Mode:

```
VRFY
```

Discovered valid users:

```
adam
eve
root
postmaster
```

This confirms SMTP user enumeration is possible.

---

# Password Testing

Tool:

```bash
Hydra
```

Command:

```bash
hydra -L user.txt -p test 10.176.121.111 imap
```

Result:

```
0 valid password found
```

Observation:

Only password **test** was attempted.

No valid credentials were recovered.

---

# Roundcube Error Log Analysis

Observed entries:

```
IMAP Error:
AUTHENTICATE PLAIN
Authentication failed
```

Meaning:

- IMAP is functional.
- Authentication backend is active.
- Login attempts failed due to incorrect credentials.

---

Database errors also observed:

```
Duplicate entry
```

for

```
session
```

table.

Possible causes:

- Session handling issue
- Application misconfiguration
- Existing session collision

No evidence of SQL injection was observed from these logs.

---

# Technologies Identified

| Component | Version |
|------------|----------|
| Apache | 2.4.41 |
| Roundcube | 1.5.9 |
| Dovecot | IMAP |
| Postfix | SMTP |
| MariaDB | 10.3.39 |
| OpenSSH | 9.1 |

---

# Information Disclosure

The assessment identified exposure of:

- Roundcube version
- Internal hostname
- Valid SMTP users
- Application error logs
- Database session errors

---

# Attack Surface Summary

| Service | Risk |
|----------|------|
| SMTP VRFY | User Enumeration |
| Roundcube | Public Login Portal |
| IMAP | Password Authentication |
| MariaDB | External Access |
| SSH | Credential Target |

---

# Findings

## Finding 1

**SMTP VRFY Enabled**

Impact:

Allows remote attackers to enumerate valid usernames.

Status:

Confirmed.

---

## Finding 2

**Roundcube Version Disclosure**

Impact:

Application version is publicly disclosed.

Status:

Confirmed.

---

## Finding 3

**Accessible Error Logs**

Impact:

Internal application paths, authentication failures, and database errors are exposed.

Status:

Confirmed.

---

## Finding 4

**Valid User Enumeration**

Discovered users:

- adam
- eve
- root
- postmaster

Status:

Confirmed.

---

## Finding 5

**External MariaDB Service**

MariaDB is reachable over the network.

Further authentication testing would be required to determine whether unauthorized access is possible.

Status:

Observed.

---

# What Was NOT Found

The assessment did **not** identify evidence of:

- SQL Injection
- Remote Code Execution
- Local File Inclusion
- Authentication bypass
- Valid IMAP credentials
- Exposed backup files
- Directory listing
- Public configuration files

---

# Conclusion

The enumeration successfully identified the target as a Roundcube-based mail server using Apache, Postfix, Dovecot, and MariaDB.

The most significant findings were:

- SMTP VRFY enabled
- Valid user enumeration
- Accessible Roundcube error logs
- Roundcube version disclosure
- External MariaDB exposure

At the conclusion of this assessment, no authenticated access or code execution had been achieved. Additional testing would require valid credentials or further investigation into application- or service-specific vulnerabilities within the authorized scope.

---
**End of Report**