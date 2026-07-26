## Binwalk – In-Depth Explanation (Pentester Perspective)

**Binwalk** is a firmware analysis and reverse engineering tool primarily used to **identify, extract, and analyze embedded files and executable code inside binary blobs**. It is heavily used in IoT, embedded device, router, and appliance security testing.

In short:

> 🔎 **Binwalk scans binary files for embedded file signatures and structures.**

---

# 1. Why Binwalk Matters in Pentesting

Modern embedded devices (routers, IP cameras, printers, PLCs, smart TVs) run custom firmware. Vendors often distribute firmware updates as `.bin`, `.img`, or `.fw` files.

These firmware files usually contain:

- Bootloaders
    
- Linux kernels
    
- Root filesystems
    
- Config files
    
- Hardcoded credentials
    
- Web server code
    
- Certificates
    
- Encryption keys
    

Binwalk allows you to:

- Identify internal components
    
- Extract embedded filesystems
    
- Locate hardcoded credentials
    
- Analyze web applications
    
- Discover outdated libraries
    
- Find backdoors
    

It is especially useful during:

- IoT pentests
    
- Hardware hacking
    
- Red team firmware analysis
    
- Vulnerability research
    

---

# 2. How Binwalk Works Internally

Binwalk works primarily through:

## 2.1 Signature-Based Scanning

It scans a binary file for **magic byte signatures** (similar to how the `file` command works).

Example:

|File Type|Magic Bytes|
|---|---|
|gzip|1F 8B|
|ZIP|50 4B 03 04|
|SquashFS|hsqs|
|ELF|7F 45 4C 46|

Binwalk compares byte sequences against its signature database to detect:

- Compressed data
    
- File systems
    
- Executables
    
- Images
    
- Archives
    

---

## 2.2 Entropy Analysis

Binwalk can measure entropy to detect:

- Encrypted data
    
- Compressed regions
    
- Packed firmware
    
- Obfuscation
    

High entropy = likely compressed or encrypted.

Low entropy = likely plaintext or structured data.

---

## 2.3 Extraction Engine

Binwalk can automatically:

- Carve embedded files
    
- Decompress archives
    
- Extract file systems
    
- Recursively analyze extracted data
    

It uses internal and external tools like:

- `unsquashfs`
    
- `tar`
    
- `gzip`
    
- `dd`
    
- `7z`
    

---

# 3. Core Binwalk Usage

### Basic scan

```bash
binwalk firmware.bin
```

Output example:

```
DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             U-Boot bootloader
131072        0x20000         Linux kernel
3145728       0x300000        Squashfs filesystem
```

This tells you:

- Offset locations
    
- What type of data exists
    
- Structure layout
    

---

# 4. Important Flags (Deep Dive)

## 4.1 Automatic Extraction

```bash
binwalk -e firmware.bin
```

- `-e` = extract known file types
    
- Creates `_firmware.bin.extracted/`
    
- Carves components using offsets
    

---

## 4.2 Recursive Extraction

```bash
binwalk -Me firmware.bin
```

- `-M` = recursive scan
    
- `-e` = extract
    
- Automatically drills down multiple layers
    

This is extremely useful for:

Firmware → SquashFS → tar → gzip → config files

---

## 4.3 Entropy Graph

```bash
binwalk -E firmware.bin
```

Helps identify:

- Encrypted sections
    
- Packed payloads
    
- Hidden partitions
    

Useful when signature detection fails.

---

## 4.4 Raw Extraction by Offset

If you know the offset:

```bash
dd if=firmware.bin of=kernel.bin bs=1 skip=131072
```

But Binwalk can automate this.

---

## 4.5 Custom Signatures

You can add custom magic signatures for:

- Proprietary firmware formats
    
- Vendor-specific containers
    

Located typically in:

```
/usr/share/binwalk/magic/
```

---

# 5. Typical Pentesting Workflow (Real-World Scenario)

## Step 1 – Obtain Firmware

- Vendor website
    
- OTA update server
    
- Extract from flash using SPI dump
    

Example tools:

- flashrom
    
- bus pirate
    
- JTAG
    

---

## Step 2 – Scan Firmware

```bash
binwalk -Me firmware.bin
```

---

## Step 3 – Identify Filesystem

Common embedded filesystems:

- SquashFS
    
- CramFS
    
- JFFS2
    
- UBIFS
    

---

## Step 4 – Analyze Extracted Filesystem

Look for:

### 🔐 Hardcoded credentials

```bash
grep -Ri password .
grep -Ri admin .
```

---

### 🔑 Private keys

```bash
find . -name "*.key"
```

---

### 🌐 Web app source code

Common paths:

```
/www/
/htdocs/
/var/www/
```

Check for:

- Command injection
    
- Path traversal
    
- Hardcoded secrets
    

---

### ⚙️ Init scripts

```
/etc/init.d/
/etc/rc.d/
```

Look for:

- Backdoors
    
- Weak permissions
    
- Debug modes
    

---

### 🧠 Outdated libraries

```bash
strings libc.so | grep GLIBC
```

Check versions → search for CVEs.

---

# 6. Advanced Use Cases

## 6.1 Firmware Backdoor Hunting

Look for:

```bash
nc
telnetd
dropbear
busybox
```

Sometimes hidden services run on alternate ports.

---

## 6.2 Extracting Encrypted Firmware

If entropy is high:

- Firmware may be encrypted
    
- Look for bootloader decryption routines
    
- Reverse engineer with:
    
    - Ghidra
        
    - IDA
        
    - Radare2
        

---

## 6.3 Modifying Firmware (Offensive Research)

Extract → modify → repack → reflash

Example:

1. Extract SquashFS
    
2. Add reverse shell
    
3. Repack filesystem
    
4. Flash device
    

⚠️ Only in controlled research environments.

---

# 7. Limitations of Binwalk

Binwalk struggles with:

- Fully encrypted firmware
    
- Custom vendor packing
    
- XOR obfuscation
    
- Encrypted partitions
    

In those cases, combine with:

- Firmware emulation
    
- Static RE
    
- Hardware dumping
    
- UART debugging
    

---

# 8. Common Embedded Filesystems Explained

## SquashFS

Compressed read-only filesystem.  
Most common in routers.

## JFFS2

Writable flash filesystem.

## UBIFS

Modern NAND flash filesystem.

---

# 9. Combining Binwalk with Other Tools

|Tool|Purpose|
|---|---|
|strings|Extract ASCII content|
|foremost|File carving|
|ghidra|Reverse engineering|
|qemu|Firmware emulation|
|firmadyne|Automated firmware emulation|
|checksec|Binary hardening checks|

---

# 10. Practical Attack Example

Let’s say you're testing an IP camera:

1. Download firmware
    
2. `binwalk -Me camera.bin`
    
3. Extract SquashFS
    
4. Locate `/etc/passwd`
    
5. Crack hashes
    
6. Find web interface CGI scripts
    
7. Discover command injection
    
8. Gain remote shell
    
9. Escalate privileges
    

---

# 11. Red Team vs Blue Team Usage

### 🔴 Red Team

- Identify hardcoded creds
    
- Discover hidden services
    
- Find old OpenSSL versions
    
- Extract API tokens
    

### 🔵 Blue Team

- Audit firmware before release
    
- Ensure no secrets embedded
    
- Detect supply chain tampering
    

---

# 12. Indicators of Interesting Findings

You should get excited when you see:

- `shadow`
    
- `authorized_keys`
    
- `.pem`
    
- `openssl`
    
- `dropbear`
    
- `lighttpd`
    
- `busybox`
    
- `iptables`
    
- `telnetd`
    

These usually mean exploitable attack surface.

---

# 13. Installation

On Kali:

```bash
sudo apt install binwalk
```

---

# 14. Summary

Binwalk is:

- A binary firmware analysis tool
    
- Signature-based
    
- Capable of recursive extraction
    
- Critical for IoT pentesting
    

It allows you to:

✔ Map firmware structure  
✔ Extract embedded files  
✔ Discover secrets  
✔ Identify outdated components  
✔ Prepare for reverse engineering

---
