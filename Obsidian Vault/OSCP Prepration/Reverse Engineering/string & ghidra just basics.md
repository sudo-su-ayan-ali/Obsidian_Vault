# 🔎 `strings` – Quick Binary Intelligence

`strings` extracts printable ASCII/Unicode sequences from binaries. It’s one of the fastest ways to get situational awareness before opening a disassembler.

---

## 🧠 Why It’s Useful in Pentesting

You can quickly identify:

- Hardcoded credentials
    
- IP addresses / domains
    
- File paths
    
- Debug messages
    
- Hidden endpoints
    
- Command execution strings (`cmd.exe`, `/bin/sh`, etc.)
    
- Interesting function names
    
- Encryption hints
    

---

## ⚙️ Basic Usage

strings binary

---

## 🎯 High-Value Usage Patterns

### 1️⃣ Show offset (important for correlation)

strings -t x binary

- `-t x` → show hexadecimal offset
    
- Useful when correlating with Ghidra / debugger
    

---

### 2️⃣ Unicode strings (VERY important on Windows)

strings -el binary

or:

strings -e l binary

Most Windows binaries store strings in UTF-16LE.

---

### 3️⃣ Filter interesting data

strings binary | grep -i pass  
strings binary | grep -i admin  
strings binary | grep -E "http|ftp|cmd|powershell"

---

### 4️⃣ Minimum length tuning

strings -n 6 binary

- Default is 4
    
- Increasing reduces noise
    

---

## 🛠 Practical Pentest Workflow

file target.exe  
checksec target.exe   # Linux  
strings -t x -el target.exe > strings.txt  
less strings.txt

Look for:

- `system`
    
- `strcpy`
    
- `gets`
    
- `WinExec`
    
- `CreateProcess`
    
- `cmd.exe`
    
- URLs
    
- Base64 blobs
    

---

## 🚨 Red Flags for Exploitation

If you see:

- `gets`
    
- `strcpy`
    
- `sprintf`
    
- `scanf("%s")`
    

You might have:

- Stack overflow potential
    
- Format string vulnerability
    

---

# 🧬 Ghidra Basics for Pentesters

Ghidra is a free reverse engineering suite developed by:

National Security Agency

It’s excellent for:

- Vulnerability research
    
- Malware analysis
    
- CTF reversing
    
- BOF prep
    
- Patch diffing
    

---

# 🏗 Basic Ghidra Workflow

---

## 1️⃣ Create Project

- File → New Project
    
- Non-Shared Project
    
- Import binary
    
- Analyze (accept defaults initially)
    

---

## 2️⃣ Key Windows You Must Know

### 🧩 Symbol Tree

Lists:

- Functions
    
- Imports
    
- Exports
    
- Strings
    
- Classes (C++)
    

---

### 🧠 Decompiler Window (Most Important)

Shows C-like pseudocode.

Example:

void vulnerable(char *input) {  
    char buffer[64];  
    strcpy(buffer, input);  
}

Immediate BOF candidate.

---

### 🔍 Listing View

Raw disassembly + annotations.

---

### 🔤 Strings Window

Search for:

- URLs
    
- Passwords
    
- Debug messages
    
- Commands
    

Right-click string → “References to”

This shows where it’s used in code.

---

# 🔥 Pentester Reverse Engineering Strategy

---

## 🎯 Step 1: Identify Entry Point

Look for:

- `main`
    
- `WinMain`
    
- `_start`
    

---

## 🎯 Step 2: Identify Dangerous Functions

Go to:

Symbol Tree → Imports

Look for:

**Linux**

- `system`
    
- `execve`
    
- `gets`
    
- `strcpy`
    

**Windows**

- `WinExec`
    
- `CreateProcessA`
    
- `ShellExecute`
    
- `strcpy`
    
- `lstrcpy`
    

Double-click → see cross-references.

---

## 🎯 Step 3: Trace User Input

Find:

- `scanf`
    
- `gets`
    
- `recv`
    
- `fgets`
    
- `ReadFile`
    

Follow input flow to:

- Copy functions
    
- Comparisons
    
- Auth logic
    

---

## 🎯 Step 4: Buffer Overflow Recon

Look for patterns:

char buf[128];  
strcpy(buf, user_input);

Decompiler shows stack variables clearly.

Check stack layout:

- Right-click function
    
- “Function Graph”
    

---

## 🎯 Step 5: Patch Testing (Advanced)

You can:

- Modify instructions
    
- Export patched binary
    
- Use for exploit dev
    

---

# 🧨 BOF-Specific Ghidra Tricks

---

## 📌 Find buffer size

In decompiler:

char local_48 [64];

That means 64-byte stack buffer.

---

## 📌 Identify EIP/RIP overwrite point

Switch to:

Window → Function Graph

Trace:

- Stack frame setup
    
- `sub rsp, XX`
    
- Buffer offset
    

---

# 🧠 Correlating `strings` with Ghidra

If `strings` shows:

0x402050 admin password incorrect

In Ghidra:

- Search → For Address
    
- Jump to offset
    
- Right-click → “References to”
    

You find auth logic instantly.

---

# 🔬 Real-World Engagement Usage

You’d use this combo when:

- Client gives you proprietary software
    
- No source code available
    
- Exploit doesn’t work reliably
    
- Reverse shell binary flagged by AV
    
- BOF lab prep (OSCP-style)
    

---

# ⚡ Speed Triage Technique (Very Effective)

1. `file`
    
2. `checksec`
    
3. `strings -t x -el`
    
4. `ghidra`
    
5. Check imports
    
6. Trace input
    
7. Identify vuln
    
8. Confirm with debugger (gdb / x64dbg)
    

---

# 🧩 When to Use Which Tool

|Scenario|Use `strings`|Use Ghidra|
|---|---|---|
|Quick triage|✅|❌|
|Credential hunting|✅|⚠|
|BOF dev|⚠|✅|
|Malware analysis|⚠|✅|
|Hardcoded config extraction|✅|⚠|
|Auth bypass reversing|❌|✅|

---

# 🛡 Pro Tip (Professional Level)

If binary is stripped:

- Use function signatures
    
- Look for common libc patterns
    
- Rename functions as you go
    
- Comment everything
    

Ghidra is powerful, but organization wins engagements.

---
