## Setting Up WIN10-01 (Windows 10 Domain Client)

### VM Configuration

| Setting      | Value            |
| ------------ | ---------------- |
| Name         | WIN10-01         |
| OS           | Windows 10       |
| RAM          | 4 GB             |
| CPU          | 2 vCPUs          |
| Disk         | 40 GB            |
| Network      | Internal Network |
| Network Name | ADLAB            |

---

## 1. Install Windows 10

Install Windows 10 normally and create a local account:

```text
Username: localadmin
Password: Admin@123
```

---

## 2. Configure Network

Set a static IP:

|Setting|Value|
|---|---|
|IP Address|192.168.100.20|
|Subnet Mask|255.255.255.0|
|Gateway|(Optional)|
|Preferred DNS|192.168.100.10|

**Important:** DNS must point to your domain controller (DC01).

Verify:

```cmd
ipconfig /all
```

You should see:

```text
DNS Servers . . . . . . : 192.168.100.10
```

---

## 3. Test Connectivity

Ping the Domain Controller:

```cmd
ping 192.168.100.10
```

Test DNS:

```cmd
nslookup corp.local
```

Expected result:

```text
Name:    corp.local
Address: 192.168.100.10
```

---

## 4. Join the Domain

Open:

```text
Settings
→ System
→ About
→ Rename this PC (Advanced)
→ Change
```

Select:

```text
Domain
```

Enter:

```text
corp.local
```

When prompted, enter:

```text
Username: CORP\Administrator
Password: <your domain admin password>
```

You should receive:

```text
Welcome to the corp.local domain
```

Restart the computer.

---

## 5. Log In with a Domain User

After reboot:

```text
Other User
```

Log in as:

```text
CORP\john.smith
```

or

```text
CORP\alice.jones
```

---

## 6. Verify Domain Membership

Open Command Prompt:

```cmd
whoami
```

Expected:

```text
corp\john.smith
```

Check the domain:

```cmd
echo %USERDOMAIN%
```

Expected:

```text
CORP
```

---

## 7. Verify in Active Directory

On DC01, open:

```text
Active Directory Users and Computers
```

Navigate to:

```text
Computers
```

You should see:

```text
WIN10-01
```

Move it to your **Workstations** OU if desired.


---

### WIN10-01 Vulnerability Plan

#### What we're working with (baseline from your manual)

```
OS        : Windows 10
Machine   : WIN10-01
IP        : 192.168.100.20
Domain    : corp.local
Local User: localadmin / Admin@123
DNS       : 192.168.100.10
Roles     : Domain Workstation only
```

---

### 5 Things We'll Configure on WIN10-01

---

#### 1. Enable RDP

**What:** Turn on Remote Desktop, allow through firewall

**Why first:** RDP is the most realistic entry point once creds are obtained from DC01-side attacks. Without it, you have creds but no interactive access to the workstation. Nearly every real engagement involves RDP lateral movement.

**Affects:** WIN10-01 only, port 3389

**Attack it enables:** `xfreerdp` from Kali once any valid domain cred is sprayed/roasted → interactive session on workstation

---

#### 2. Enable WinRM

**What:** Run `Enable-PSRemoting`, open port 5985

**Why:** Gives a second lateral movement path beyond RDP. More realistic for modern environments where admins use PowerShell remoting. Also enables `Evil-WinRM` practice which is a staple post-exploitation tool.

**Affects:** WIN10-01 only, port 5985

**Attack it enables:** `Evil-WinRM` from Kali with valid creds → remote PowerShell session

---

#### 3. Local Admin Password Reuse

**What:** Set `localadmin` password identical to a domain account password

**Example:**

```
localadmin password → P@ssw0rd123
```

Same as `john.smith` / `alice.jones` domain passwords

**Why:** Single most common real-world finding in internal assessments. Teaches pass-the-hash and credential reuse lateral movement. Without this, local admin is isolated — with this, compromising one account cascades.

**Affects:** Local SAM on WIN10-01 only

**Attack it enables:**

```
CrackMapExec smb → local admin spray
Pass-the-hash via obtained NTLM hash
Lateral movement without needing domain creds
```

---

#### 4. Disable Windows Defender / Firewall

**What:**

```
Disable Windows Defender real-time protection
Disable Windows Firewall on all profiles
```

**Why:** Realistic lab shortcut — without this, Mimikatz and other post-exploitation tools get caught immediately before you even practice the technique. Purpose is to learn the attack chain, not AV evasion (that's a separate lab topic).

**Affects:** WIN10-01 endpoint protection only

**Attack it enables:**

```
Mimikatz → lsadump, sekurlsa (credential dumping)
Any post-exploitation tool without AV interference
```

---

#### 5. Unconstrained Delegation

**What:** Enable `TrustedForDelegation` on WIN10-01 computer object

**Why last:** Most complex attack chain in the WIN10-01 plan. Requires:

- WIN10-01 to have delegation enabled
- A coercion technique (PetitPotam/PrinterBug) to force DC01 to authenticate to WIN10-01
- Rubeus or similar to capture the DC's TGT from WIN10-01's cache
- Pass-the-ticket to get Domain Admin

Teaches the full coercion → ticket capture → impersonation chain which is extremely realistic in modern AD environments.

**Affects:** WIN10-01 computer object in AD (configured from DC01 but attack lands on WIN10-01)

**Attack it enables:**

```
PetitPotam → coerce DC01 auth to WIN10-01
Rubeus monitor → capture DC01 TGT
Pass-the-ticket → Domain Admin
```

---

### Order of Configuration

```
Step 1 → Enable RDP                     (immediate access after cred obtained)
Step 2 → Enable WinRM                   (second access path)
Step 3 → Local admin password reuse     (lateral movement + PTH)
Step 4 → Disable Defender + Firewall    (unblocks post-exploitation tools)
Step 5 → Unconstrained delegation       (most complex, save for last)
```

---

### Snapshot Plan for WIN10-01

```
Snapshot 1 → Fresh-Domain-Join              ← already exists from your manual
Snapshot 2 → After Step 1 + 2              (access layer — RDP + WinRM)
Snapshot 3 → After Step 3 + 4              (lateral movement + post-ex layer)
Snapshot 4 → After Step 5                  (delegation layer — full lab)
```

---

### How WIN10-01 Fits Into the Full Attack Chain

```
Kali
  │
  ├── Password spray DC01 → get john.smith creds
  │
  ├── RDP into WIN10-01 as john.smith         ← Step 1
  │
  ├── Evil-WinRM into WIN10-01                ← Step 2
  │
  ├── Discover localadmin reuse → PTH         ← Step 3
  │
  ├── Drop Mimikatz → dump creds              ← Step 4
  │
  └── Trigger coercion → capture DC01 TGT    ← Step 5
        └── Pass-the-ticket → Domain Admin
```

---

### What This Doesn't Touch (already planned for DC01)

```
- Password spray target    → DC01 (AD auth)
- AS-REP / Kerberoast     → DC01 (AD objects)
- GPP cpassword            → DC01 (SYSVOL)
- ACL abuse                → DC01 (AD ACL)
- ADCS ESC1                → DC01 (CA)
```

---

### Complete Lab Picture (both machines)

|Attack|Machine Configured|Attacked From|
|---|---|---|
|Password spray|DC01|Kali → DC01|
|AS-REP roast|DC01|Kali → DC01|
|Kerberoast|DC01|Kali → DC01|
|GPP cpassword|DC01|Kali → DC01 SYSVOL|
|ACL abuse|DC01|Kali → DC01|
|ADCS ESC1|DC01|Kali → DC01|
|RDP foothold|WIN10-01|Kali → WIN10-01|
|WinRM foothold|WIN10-01|Kali → WIN10-01|
|Local admin reuse + PTH|WIN10-01|Kali → WIN10-01|
|Mimikatz cred dump|WIN10-01|On WIN10-01|
|Unconstrained delegation|WIN10-01|Kali coerces DC01 → WIN10-01|


---

## Common Problems

|Problem|Fix|
|---|---|
|Cannot find domain|Check DNS is `192.168.100.10`|
|Ping fails|Verify both VMs use the same VirtualBox Internal Network (`ADLAB`)|
|Domain join denied|Use `CORP\Administrator` credentials|
|Time mismatch|Sync Windows time on both machines|

---

## Final Lab

```text
ADLAB
│
├── DC01
│   ├── Active Directory
│   ├── DNS
│   └── Users/Groups
│
├── WIN10-01
│   └── Domain Client
│
└── KALI
    └── Security Testing VM
```




With just **DC01 + WIN10-01 + Kali**, you'll have a complete basic AD lab for learning domain administration, Group Policy, DNS, authentication concepts, logging, and authorized security testing in an isolated environment.