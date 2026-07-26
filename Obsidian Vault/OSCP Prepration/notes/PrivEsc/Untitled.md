Windows Privilege Escalation techniques:

# Kernel & OS

## ☣️ Kernel Exploits

### 1. The Kernel Landscape: Rings & Privileges

In Windows, the CPU operates in different **Privilege Rings**. User applications run in **Ring 3**, while the Kernel runs in **Ring 0**. A kernel exploit is essentially a bridge that allows code from Ring 3 to execute in Ring 0.

- **User Mode (Ring 3):** Limited access to hardware; uses APIs to talk to the kernel.
    
- **Kernel Mode (Ring 0):** Unrestricted access to system memory and hardware.
    
- **The Goal:** Gain `NT AUTHORITY\SYSTEM` by exploiting a vulnerability in `ntoskrnl.exe` (the kernel) or a **Kernel Mode Driver** (.sys files).
    
### 2. Common Kernel Vulnerability Classes

For the OSCP+, you should be familiar with these three main types of "bugs":

- **Buffer Overflows:** Writing more data to a kernel-space buffer than it can hold, overwriting the return address to redirect execution.
    
- **Arbitrary Write (Write-What-Where):** The ability to write a specific value to a specific memory address in the kernel. This is often used to overwrite the **Token** of your current process with the Token of the System process.
    
- **Use-After-Free (UAF):** Exploiting a program that continues to use a pointer after the memory it points to has been "freed." An attacker can fill that "free" spot with malicious data.
    

### 3. Modern Protections (The "Boss" Mechanics)

Modern Windows (10/11) has built-in defenses that make "old-school" exploits fail. You need to know how to identify these:

|**Mitigation**|**What it does**|**How to bypass (Overview)**|
|---|---|---|
|**kASLR**|Randomizes where the kernel is loaded in memory.|Need a "memory leak" vulnerability to find the base address.|
|**SMEP**|Prevents the Kernel from executing code in User pages.|Use **ROP (Return Oriented Programming)** to disable SMEP or flip bits.|
|**KPTI**|Isolates kernel and user-mode page tables.|More complex side-channel or specific data-only attacks.|

---

## 🔗 DLL Hijacking

### 1. The Windows DLL Search Order (Standard)

When an application calls a DLL without a "Fully Qualified Path" (e.g., `LoadLibrary("example.dll")`), Windows hunts for it in this specific priority:

1. **The Directory of the Executable:** Where the `.exe` lives.
    
2. **The System Directory:** `C:\Windows\System32`.
    
3. **The 16-bit System Directory:** `C:\Windows\System` (Rarely relevant now).
    
4. **The Windows Directory:** `C:\Windows`.
    
5. **The Current Working Directory (CWD):** The folder the user is currently "in" within the shell.
    
6. **The PATH Environment Variable:** Directories listed in system/user PATH.
    

> **The Vulnerability:** If an application expects a DLL in `System32` (Step 2) but you have write access to the Application Directory (Step 1), you can "intercept" the request by placing your malicious DLL there.



### 2. Hunting with Process Monitor (ProcMon)

This is the most critical skill for the OSCP+ labs. You aren't guessing; you are observing.

**The ProcMon Filter Setup:** To find "missing" DLLs that you can hijack, set these filters:

- **Process Name** is `target_app.exe`
    
- **Result** is `NAME NOT FOUND`
    
- **Path** ends with `.dll`
    

**What to look for:** Look for entries where the application tries to load a DLL from a **user-writable directory** (like `C:\Users\Public` or a custom App folder with weak permissions) and fails (`NAME NOT FOUND`). That is your entry point.



### 3. Advanced Techniques: Side-loading vs. Proxying

In a modern pentest, simply replacing a DLL often crashes the program, alerting the user. You need to be "surgical."

#### **DLL Side-loading**

You find a **legitimate, signed executable** (like an old version of `GUP.exe` or `VLC.exe`) and place it in a writable folder along with your malicious DLL. Because the EXE is trusted, AV/EDR is less likely to block the initial execution.

#### **DLL Proxying (The "Pro" Move)**

Instead of just stealing the execution, your malicious DLL **forwards** all legitimate function calls to the _real_ DLL.

- **How it works:** You rename the original `real.dll` to `real_orig.dll`. Your malicious `real.dll` executes your shellcode and then tells the app, "Hey, for everything else, talk to `real_orig.dll`."
    
- **Benefit:** The application continues to run perfectly, making your presence nearly invisible to the user.

---


## 💉 DLL Injection

### 1. The Core Concept: Forced Loading

In DLL Injection, you (the attacker) already have code execution on the box (usually as a low-privilege user). Your goal is to migrate your code into a **more stable** or **higher-privileged** process (like `explorer.exe` or `spoolsv.exe`).

### 2. The Classic "CreateRemoteThread" Method

This is the "textbook" method you must memorize for the OSCP+. It follows a 5-step API dance:

1. **`OpenProcess`**: Get a "handle" to the target process (e.g., Notepad).
    
2. **`VirtualAllocEx`**: Carve out a small piece of memory _inside_ that target process.
    
3. **`WriteProcessMemory`**: Write the **path** to your malicious DLL (e.g., `C:\temp\evil.dll`) into that carved-out space.
    
4. **`GetProcAddress`**: Find the memory address of `LoadLibraryA` inside `kernel32.dll`. (Since `kernel32.dll` is mapped at the same address for almost all processes, this works perfectly).
    
5. **`CreateRemoteThread`**: Tell the target process to start a new thread, but tell it the "start function" is `LoadLibraryA` and the "argument" is the path you wrote in step 3.
    

> **Why this works:** The target process thinks it’s just loading a legitimate library, but as soon as `LoadLibraryA` runs, your DLL's `DllMain` function executes your reverse shell.

### 3. Advanced Injection: Reflective DLL Injection

Modern EDRs (Endpoint Detection and Response) look for `CreateRemoteThread` and `LoadLibraryA` calls like hawks. To bypass them, you use **Reflective Injection**.

- **The Difference:** Instead of pointing to a file on disk, you write the **entire DLL bytes** directly into the target process's memory.
    
- **The "Magic":** The DLL must have its own "Reflective Loader" function that mimics what the Windows OS loader does (mapping sections, fixing relocations).
    
- **OSCP+ Relevance:** This is how tools like **Metasploit's Meterpreter** and **Cobalt Strike** work. They never touch the disk, making them "Fileless."
    

### 4. Process Hollowing (The "Imposter" Technique)

If you can't inject into a running process, you can create a new one in a "suspended" state and swap its brains.

1. Start a legitimate process (like `svchost.exe`) in a **Suspended** state.
    
2. Unmap (hollow out) the legitimate code from its memory.
    
3. Replace it with your malicious code.
    
4. **Resume** the thread.
    

- **Result:** To a defender looking at Task Manager, it looks like a legitimate Windows service, but it's actually your malware.



---

## Unquoted Service Paths







---

**Services & Scheduled Tasks**

- Weak Service Permissions
- Service Binary Hijacking
- Scheduled Task Abuse
- Always Install Elevated

**Credentials & Tokens**

- Token Impersonation / Theft
- Pass-the-Hash
- Pass-the-Ticket
- Credential Dumping (LSASS)
- SAM/NTDS.dit Extraction
- Cached Credentials (MSCache)
- DPAPI Abuse
- Kerberoasting
- AS-REP Roasting
- SeImpersonatePrivilege (Potato attacks)
- SeDebugPrivilege Abuse
- SeTakeOwnershipPrivilege

**Registry & File System**

- Weak Registry Permissions
- Autologon Credentials in Registry
- AlwaysInstallElevated (Registry)
- Writable PATH Directories
- Weak File/Folder Permissions

**Active Directory**

- DCSync Attack
- ACL/ACE Abuse
- Group Policy Object (GPO) Abuse
- PrintSpooler / PrintNightmare
- Zerologon
- noPac / SamAccountName Spoofing

**UAC Bypass**

- UAC Bypass via fodhelper
- UAC Bypass via eventvwr
- Auto-Elevate COM Objects

**Misc**

- Named Pipe Impersonation
- Insecure GUI Applications
- Startup Folder Persistence
- WSL Privilege Escalation
- Stored Credentials (cmdkey / Windows Vault)