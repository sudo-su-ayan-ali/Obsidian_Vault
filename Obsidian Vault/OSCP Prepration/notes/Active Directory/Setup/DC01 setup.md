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

Next, create **WIN10-01**, point its DNS to `192.168.100.10`, and join it to the `corp.local` domain.