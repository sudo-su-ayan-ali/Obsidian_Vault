## Active Directory + Red Team Lab (Brief Overview)

### 1. Host Machine

|Component|Specification|
|---|---|
|OS|Arch Linux|
|CPU|Intel Core i7-3770 (4C/8T)|
|RAM|16 GB|
|Virtualization|VirtualBox|
|Free Disk Space|~184 GB|

---

## 2. Recommended Virtual Machines

|VM Name|OS|RAM|vCPU|Purpose|
|---|---|---|---|---|
|DC01|Windows Server 2022|4 GB|2|Domain Controller, DNS|
|WIN10-01|Windows 10/11|4 GB|2|Domain User Workstation|
|KALI|Kali Linux|2 GB|2|Security Testing|
|FILE01 (Optional)|Windows Server|2 GB|1|File Shares|

**Minimum Lab:** DC01 + WIN10-01 + KALI

---

## 3. Network Configuration

|Device|IP Address|
|---|---|
|DC01|192.168.100.10|
|WIN10-01|192.168.100.20|
|KALI|192.168.100.30|
|FILE01|192.168.100.40|

|Setting|Value|
|---|---|
|Network Type|Internal Network|
|Network Name|ADLAB|
|Domain Name|corp.local|

---

## 4. Active Directory Structure

|OU|Contains|
|---|---|
|Domain Controllers|DC01|
|Servers|FILE01|
|Workstations|WIN10-01|
|Users|Employee Accounts|
|Groups|IT, HR, Finance|

### Sample Users

|Username|Department|
|---|---|
|john.smith|IT|
|alice.jones|HR|
|finance.user|Finance|
|helpdesk.user|IT Support|

---

## 5. Learning Roadmap

|Phase|Learn|
|---|---|
|1|Active Directory Basics|
|2|DNS & Kerberos|
|3|LDAP Enumeration|
|4|BloodHound Mapping|
|5|Privilege Escalation Concepts|
|6|Detection & Logging|

---

## 6. Common Tools

|Tool|Purpose|
|---|---|
|VirtualBox|Virtual Machines|
|RSAT|AD Administration|
|BloodHound|AD Relationship Mapping|
|Neo4j|BloodHound Database|
|Nmap|Network Discovery|
|Sysmon|Windows Logging|
|Wireshark|Traffic Analysis|

---

## 7. Practice Activities

|Activity|Goal|
|---|---|
|Domain Join|Add PCs to AD|
|User Management|Create Users & Groups|
|GPO Creation|Apply Policies|
|Share Management|File Permissions|
|Network Scanning|Discover Hosts|
|LDAP Queries|Enumerate Directory|
|Log Analysis|Detect Activity|

---

## 8. Resource Usage

|VM|RAM|
|---|---|
|DC01|4 GB|
|WIN10-01|4 GB|
|KALI|2 GB|
|Host OS|~3–4 GB|

**Total Usage:** ~13–14 GB RAM

---

## 9. Final Lab Diagram

```text
                     ADLAB Network
                   (192.168.100.0/24)

               +------------------+
               |      DC01        |
               | 192.168.100.10   |
               | Domain Controller|
               +---------+--------+
                         |
          +--------------+--------------+
          |                             |
+---------+--------+         +----------+--------+
|     WIN10-01     |         |      FILE01       |
| 192.168.100.20   |         | 192.168.100.40    |
| Domain User PC   |         | File Server       |
+------------------+         +-------------------+

                         |
                 +-------+-------+
                 |     KALI      |
                 |192.168.100.30 |
                 | Testing VM    |
                 +---------------+
```

This 3–4 VM setup is a good balance for your 16 GB RAM system and is sufficient for learning Active Directory administration, Windows networking, directory services, logging, and authorized security testing in a contained lab.