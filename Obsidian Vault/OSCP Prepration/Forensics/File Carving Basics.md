# File Carving Basics (Pentester-Focused Deep Dive)

File carving is a **forensic and offensive technique used to extract files from raw data without relying on a filesystem structure**.

In simple terms:

> 🪓 File carving recovers files from raw disk, memory, or firmware images by identifying file signatures (magic bytes).

It is heavily used in:

- Digital forensics
    
- Incident response
    
- Firmware analysis
    
- Disk exploitation
    
- Memory analysis
    
- CTFs
    
- Data recovery
    

---

# 1. Why File Carving Is Important in Pentesting

During engagements you may encounter:

- Raw disk images (`.dd`, `.img`)
    
- Memory dumps
    
- Corrupted filesystems
    
- Embedded firmware blobs
    
- Deleted files
    
- Slack space artifacts
    
- Unallocated disk space
    

File carving allows you to:

✔ Recover deleted documents  
✔ Extract credentials from unallocated space  
✔ Recover embedded payloads  
✔ Extract hidden data from firmware  
✔ Recover malware from memory dumps

---

# 2. Core Concept: How File Carving Works

File carving does **not** rely on:

- File allocation tables
    
- Inodes
    
- MFT (Master File Table)
    
- Directory structure
    

Instead, it relies on:

### 🔎 1. File Signatures (Magic Bytes)

Every file type starts with a unique byte pattern.

Examples:

|File Type|Header (Hex)|Footer (Hex)|
|---|---|---|
|JPEG|FF D8 FF|FF D9|
|PNG|89 50 4E 47|IEND|
|PDF|25 50 44 46|%%EOF|
|ZIP|50 4B 03 04|50 4B 05 06|
|ELF|7F 45 4C 46|(no fixed footer)|

Carving engines:

1. Scan raw data for header
    
2. Identify footer (if exists)
    
3. Extract everything in between
    

---

# 3. When File Carving Is Used

## 3.1 Deleted File Recovery

When a file is deleted:

- Data often remains on disk
    
- Only metadata pointers are removed
    
- Carving can recover the content
    

---

## 3.2 Memory Forensics

Example:

- Dumped RAM
    
- Malware injected into memory
    
- Extract PE files or shellcode
    

---

## 3.3 Firmware Analysis

Firmware may contain:

- Embedded PNGs
    
- Config backups
    
- Web server files
    
- Hidden archives
    

Carving extracts them even without known filesystem structures.

---

## 3.4 Malware Analysis

Packed malware often embeds:

- Secondary payloads
    
- Encrypted config
    
- Hidden PE files
    

Carving helps extract them.

---

# 4. Basic Carving Workflow

## Step 1 – Obtain Raw Data

Examples:

```bash
dd if=/dev/sda of=disk.img bs=4M
```

Or:

- Memory dump (`.raw`)
    
- Firmware blob (`firmware.bin`)
    
- Network capture converted to raw stream
    

---

## Step 2 – Identify File Signatures

Use:

```bash
binwalk disk.img
```

or

```bash
strings disk.img
```

---

## Step 3 – Carve Files

### Using Foremost

```bash
foremost -i disk.img -o output/
```

---

### Using Scalpel

```bash
scalpel disk.img -o output/
```

---

### Using Binwalk (firmware-focused)

```bash
binwalk -e disk.img
```

---

# 5. Manual File Carving (Understanding the Internals)

Suppose you find a JPEG header at offset 20480.

Extract manually:

```bash
dd if=disk.img of=image.jpg bs=1 skip=20480 count=50000
```

This method is useful when:

- Tools fail
    
- Signatures are corrupted
    
- You need precision extraction
    

---

# 6. Common File Carving Tools

## 🔹 Foremost

- Header/footer based
    
- Configurable signatures
    
- Good for disk images
    

---

## 🔹 Scalpel

- Improved version of Foremost
    
- More configurable
    
- Faster
    

---

## 🔹 Binwalk

- Best for firmware
    
- Handles embedded filesystems
    
- Recursive extraction
    

---

## 🔹 Bulk Extractor

- Extracts emails, URLs, credit cards
    
- Does not rely on file structure
    
- Great for IR
    

---

## 🔹 PhotoRec

- Strong recovery tool
    
- Ignores filesystem
    
- Works on many file types
    

---

# 7. Challenges in File Carving

## 7.1 Fragmented Files

If a file is fragmented:

- Header and footer may not be contiguous
    
- Carving may fail
    
- Partial recovery possible
    

---

## 7.2 False Positives

Random data may match a signature.

Example:

- Random bytes match JPEG header pattern
    

Always verify carved files.

---

## 7.3 Encrypted or Compressed Data

High entropy regions may:

- Hide files
    
- Prevent carving
    
- Require decompression first
    

---

# 8. Advanced Techniques

## 8.1 Entropy Analysis

Detect suspicious areas:

```bash
binwalk -E disk.img
```

High entropy = encrypted/compressed payload

---

## 8.2 Custom Signature Creation

Scalpel config example:

```
jpg     y       2000000     \xff\xd8\xff     \xff\xd9
```

You can define:

- Header
    
- Footer
    
- Max file size
    

---

## 8.3 XOR / Obfuscation Recovery

If data is XOR’d:

- Detect repeating patterns
    
- Use CyberChef or scripts
    
- Deobfuscate before carving
    

---

# 9. File Carving vs Filesystem Recovery

|Filesystem Recovery|File Carving|
|---|---|
|Uses metadata|Ignores metadata|
|Faster|Slower|
|Needs intact FS|Works on damaged disks|
|Accurate filenames|Usually generic names|

---

# 10. Red Team vs Blue Team Use

## 🔴 Red Team

- Recover credentials from images
    
- Extract deleted configs
    
- Recover staged malware
    
- Pull keys from memory dumps
    

---

## 🔵 Blue Team

- Incident response
    
- Recover deleted attacker tools
    
- Extract malware artifacts
    
- Forensic analysis
    

---

# 11. Real-World Pentest Example

### Scenario: Compromised Linux Server

You obtain:

```
/dev/sda
```

You image disk.

Run:

```bash
foremost -i disk.img -o recovered/
```

You recover:

- `/recovered/jpg/00000012.jpg`
    
- `/recovered/pdf/00000003.pdf`
    
- `/recovered/zip/00000008.zip`
    

Inside zip:

- `.ssh/id_rsa`
    
- Database backups
    

This can escalate impact significantly.

---

# 12. Indicators of Interesting Findings

Look for:

- `.pem`
    
- `.key`
    
- `shadow`
    
- `config.php`
    
- `.env`
    
- Wallet files
    
- SSH keys
    
- Browser password DBs
    

---

# 13. Practical Lab Exercise Idea

1. Create a file
    
2. Delete it
    
3. Image disk
    
4. Carve it back
    

You’ll understand carving deeply by doing it.

---

# 14. Summary

File carving:

- Recovers files from raw data
    
- Uses magic bytes
    
- Ignores filesystem metadata
    
- Critical for forensics & firmware analysis
    
- Essential skill for advanced pentesters
    

---

If you'd like next, I can explain:

- 🔬 Carving from memory dumps
    
- ⚙️ Building your own carving tool
    
- 🧠 Detecting fragmented file recovery
    
- 🛡️ Anti-forensics and how attackers evade carving
    

Let me know which direction you want to go.