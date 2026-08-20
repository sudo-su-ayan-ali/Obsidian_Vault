## Setting Up DC01 (Windows Server Domain Controller)

### Step 1: Create the VM in VirtualBox

| Setting         | Value                              |
| --------------- | ---------------------------------- |
| Name            | DC01                               |
| OS Type         | Windows Server 2022                |
| RAM             | 4096 MB                            |
| CPU             | 2 vCPUs                            |
| Disk            | 40 GB (VDI, Dynamically Allocated) |
| Network Adapter | Internal Network                   |
| Network Name    | ADLAB                              |

---

### Step 2: Install Windows Server

1. Mount the Windows Server ISO.
    
2. Start the VM.
    
3. Install:
    
    - Windows Server 2022 Standard Evaluation (Desktop Experience)
        
4. Set Administrator password.
		password = DCAdmin@123
    

After installation, log in as:

```text
Administrator
```

---

### Step 3: Configure Static IP

Open:

```text
Settings → Network & Internet → Ethernet
```

Or:

```text
Control Panel → Network and Sharing Center
```

Configure:

| Setting       | Value                  |
| ------------- | ---------------------- |
| IP Address    | 192.168.100.10         |
| Subnet Mask   | 255.255.255.0          |
| Gateway       | Leave Blank (Lab Only) |
| Preferred DNS | 192.168.100.10         |

Verify:

```powershell
ipconfig
```

---

### Step 4: Rename Server

Open PowerShell as Administrator:

```powershell
Rename-Computer -NewName DC01 -Restart
```

Wait for reboot.

Verify:

```powershell
hostname
```

Expected:

```text
DC01
```

---

### Step 5: Install Active Directory Domain Services

Open PowerShell:

```powershell
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools
```

Verify:

```powershell
Get-WindowsFeature AD-Domain-Services
```

Status should show:

```text
Installed
```

---

### Step 6: Promote Server to Domain Controller

Run:

```powershell
Install-ADDSForest `
-DomainName "corp.local" `
-DomainNetbiosName "CORP" `
-InstallDNS
```

You'll be prompted to:

- Set Directory Services Restore Mode (DSRM) password.
	    Password is DCAdmin@123

After confirmation:

```text
Server will reboot automatically
```

---

### Step 7: Verify Domain Creation

Login as:

```text
CORP\Administrator
```

Check:

```powershell
echo %USERDOMAIN%
```

Expected:

```text
CORP
```

---

### Step 8: Create Organizational Units (OUs)

Open:

```text
Server Manager
→ Tools
→ Active Directory Users and Computers
```

Create:

|OU Name|
|---|
|Users|
|Workstations|
|Servers|
|IT|
|HR|
|Finance|

---

### Step 9: Create Test Users

Example:

|Username|Password|
|---|---|
|john.smith|P@ssw0rd123|
|alice.jones|P@ssw0rd123|
|helpdesk.user|P@ssw0rd123|

_(Use stronger passwords in real environments.)_

---

### Step 10: Configure DNS

Verify DNS service:

```powershell
Get-Service DNS
```

Expected:

```text
Running
```

Test:

```powershell
nslookup corp.local
```

Should resolve to:

```text
192.168.100.10
```

---

### Step 11: Snapshot the VM

In VirtualBox:

```text
DC01
→ Snapshots
→ Take Snapshot
```

Name:

```text
Fresh-Domain-Controller
```

This gives you a clean restore point before adding workstations and experimenting.

---

## Result

|Service|Status|
|---|---|
|Windows Server Installed|✅|
|Static IP Set|✅|
|Renamed to DC01|✅|
|AD DS Installed|✅|
|DNS Installed|✅|
|Domain Created (corp.local)|✅|
|Users/OUs Created|✅|

---

### DC01 Vulnerability Plan

#### What we're working with (baseline from your manual)

```
OS        : Windows Server 2022
Domain    : corp.local
DC Name   : DC01
IP        : 192.168.100.10
Roles     : AD DS + DNS (only)
Users     : john.smith, alice.jones, helpdesk.user, Administrator
```

---

### 6 Things We'll Configure on DC01

---

#### 1. Disable Account Lockout

**What:** Set lockout threshold to 0 (unlimited attempts)

**Why first:** Every other attack below involves password guessing or spraying — if lockout is active, you'll lock accounts before you can practice the actual technique. This unblocks everything else.

**Affects:** Entire domain via Default Domain Policy

**Attack it enables:** Password spraying without consequences

---

#### 2. AS-REP Roasting

**What:** Disable Kerberos preauth on `john.smith`

**Why:** This is the simplest roasting attack — requires zero creds to pull a crackable hash. Good starting point before Kerberoasting because it's pre-authentication, meaning you don't even need a valid session first.

**Affects:** `john.smith` user object only

**Attack it enables:** `GetNPUsers.py` → offline hash crack

---

#### 3. Kerberoasting

**What:**

```
Create svc_sql account
Assign weak password
Register SPN against it
```

**Why:** Realistic — service accounts with SPNs are in virtually every real AD environment. Any authenticated domain user can request the TGS and crack it offline. Teaches the SPN → TGS → hashcat workflow.

**Affects:** New `svc_sql` user object in AD

**Attack it enables:** `GetUserSPNs.py` → offline TGS crack

---

#### 4. GPP cpassword

**What:** Create a GPO containing an embedded credential (local admin preference with password) stored in SYSVOL

**Why:** Classic finding, still common in orgs that built their AD pre-2014. Teaches SYSVOL enumeration and why storing creds in Group Policy is permanently broken regardless of MS14-025 patch status (patch stops _creating_ new ones, doesn't remove existing ones).

**Affects:** SYSVOL share, readable by all domain users

**Attack it enables:** `Get-GPPPassword` / `gpp-decrypt` → plaintext credential recovery

---

#### 5. Excessive ACL — helpdesk.user

**What:** Grant `helpdesk.user` `GenericAll` over the `Users` OU

**Why:** Most realistic misconfiguration on this list — over-permissioned helpdesk accounts exist in almost every real org. Teaches ACL-based privilege escalation which is BloodHound's core use case. No exploit needed, just abusing legitimate AD permissions.

**Affects:** ACL on `Users` OU object

**Attack it enables:** BloodHound path discovery → `dacledit` / `Set-DomainUserPassword` → escalation to any user in that OU including Domain Admin

---

#### 6. ADCS — ESC1 Misconfiguration

**What:**

```
Install ADCS role
Deploy Enterprise Root CA
Misconfigure certificate template:
  - ENROLLEE_SUPPLIES_SUBJECT = true
  - Domain Users can enroll
  - No manager approval
```

**Why last:** Most complex to set up and attack. ADCS ESC1 is currently one of the highest-value real-world vectors — any domain user can request a cert as Domain Admin and use it for full domain compromise. Teaches certificate-based attacks which are separate from Kerberos/NTLM chains.

**Affects:** New ADCS role on DC01, new certificate template

**Attack it enables:** `Certipy find` → `Certipy req` as DA → `Certipy auth` → full compromise

---

### Order of Configuration

```
Step 1 → Disable account lockout        (unblocks everything)
Step 2 → AS-REP roast john.smith        (no creds needed to attack)
Step 3 → Kerberoast via svc_sql         (needs domain user to attack)
Step 4 → GPP cpassword in SYSVOL       (needs domain user to attack)
Step 5 → ACL abuse via helpdesk.user   (needs BloodHound to find)
Step 6 → ADCS ESC1                     (most complex, save for last)
```

---

### Snapshot Plan for DC01

```
Snapshot 1 → Fresh-Domain-Controller        ← already exists from your manual
Snapshot 2 → After Step 1 + 2 + 3          (spray + roasting layer)
Snapshot 3 → After Step 4 + 5              (SYSVOL + ACL layer)
Snapshot 4 → After Step 6                  (ADCS layer — full lab)
```

---

### What This Doesn't Touch (keeping for WIN10-01 plan)

```
- SMB relay        → WIN10-01 side
- Unconstrained delegation → WIN10-01 computer object
- RDP / WinRM      → WIN10-01 side
- Local admin reuse → WIN10-01 side
```

---

Next, create **WIN10-01**, point its DNS to `192.168.100.10`, and join it to the `corp.local` domain.