## 1. **Strings**

**What they are:**  
Human-readable text found inside files, memory dumps, executables, or disk images—even if the file is deleted or partially corrupted.

**Common uses in forensics:**

- 🔍 **Recover hidden information**
    
    - Usernames, passwords (sometimes), URLs, email addresses
        
- 🦠 **Malware analysis**
    
    - Command-and-control servers
        
    - Hard-coded IPs, file paths, registry keys
        
- 🗂️ **Deleted data discovery**
    
    - Text remnants from wiped documents or chats
        
- 🧠 **Memory forensics**
    
    - Commands typed by an attacker
        
    - Running processes or injected code
        

**Example:**  
Running a `strings` scan on malware might reveal:

`http://malicious-site.com/update C:\Windows\System32\svchost.exe`

→ instant clues about attacker behavior.

---

## 2. **Metadata**

**What it is:**  
“Data about data.” Information describing _how, when, where, and by whom_ a file was created or modified.

**Common metadata fields:**

- 📅 Created / Modified / Accessed timestamps
    
- 👤 Author or owner
    
- 💻 Device or software used
    
- 📍 GPS location (photos)
    
- 🧾 File size, type, hash values
    

**Uses in forensics:**

- ⏱️ **Timeline reconstruction**
    
    - What happened first, next, and last
        
- 🧑‍⚖️ **Attribution**
    
    - Linking files to users or devices
        
- 🔁 **Detecting tampering**
    
    - Modified timestamps not matching user activity
        
- 📷 **Image & document investigations**
    
    - EXIF data in photos
        
    - Author info in PDFs, Word files
        

**Example:**  
A document claiming to be written in 2022 shows metadata created in 2025 → 🚩 suspicious.

---

## 3. **Analysis**

**What it is:**  
The process of _interpreting_ evidence to form conclusions.

**Types of forensic analysis:**

### 🔬 Technical Analysis

- Disk, memory, and network traffic analysis
    
- Log correlation
    
- Malware reverse engineering
    

### 🕒 Timeline Analysis

- Combining metadata, logs, and events
    
- Identifying gaps or anomalies
    

### 🔗 Correlation Analysis

- Linking files, users, IPs, and actions
    
- Cross-device or cross-account evidence
    

### ⚖️ Evidentiary Analysis

- Ensuring evidence integrity (hashing, chain of custody)
    
- Presenting findings in court-acceptable format
    

**Why analysis matters:**  
Raw data ≠ evidence.  
Analysis turns artifacts into **proof**.

---