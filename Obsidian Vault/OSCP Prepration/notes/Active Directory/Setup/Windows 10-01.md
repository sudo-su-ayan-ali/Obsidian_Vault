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