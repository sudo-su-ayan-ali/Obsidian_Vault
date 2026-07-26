## 🌳 The Big Idea

In Linux, **everything starts at `/` (root)**.  
There are **no drive letters** like `C:` or `D:`—all files and devices live somewhere under `/`.

---

## 📁 Key Directories (the important ones)

### `/`

The **root directory**.  
Every other directory hangs off this one.

---

### `/bin`

Essential **user commands**.

- Examples: `ls`, `cp`, `mv`, `cat`
    
- Needed for the system to work even in emergency mode
    

---

### `/sbin`

Essential **system/admin commands**.

- Examples: `ip`, `mount`, `fsck`
    
- Mostly used by the root user
    

---

### `/etc`

**Configuration files** for the system.

- Examples:
    
    - `/etc/passwd` → user accounts
        
    - `/etc/fstab` → disk mounts
        
- No binaries here, just text configs
    

---

### `/home`

**User home directories**.

- Example: `/home/alex`
    
- This is where personal files, downloads, configs, etc. live
    

---

### `/root`

Home directory for the **root user** (not the same as `/`)

---

### `/lib` and `/lib64`

Essential **shared libraries** needed by `/bin` and `/sbin`.

- Similar to DLLs on Windows
    

---

### `/usr`

User-space programs and data (not “users”).

- `/usr/bin` → most normal commands
    
- `/usr/lib` → libraries
    
- `/usr/share` → docs, icons, man pages
    

👉 Think of `/usr` as “**installed software**”

---

### `/var`

**Variable data** that changes often.

- `/var/log` → system logs
    
- `/var/spool` → mail, print queues
    
- `/var/cache` → cached files
    

---

### `/tmp`

**Temporary files**.

- Often wiped on reboot
    
- Anyone can write here
    

---

### `/dev`

**Device files**.

- Hard drives, USBs, terminals, etc.
    
- Examples:
    
    - `/dev/sda` → disk
        
    - `/dev/null` → the black hole of Linux 😄
        

---

### `/proc`

Virtual filesystem with **system and process info**.

- Lives in RAM
    
- Example:
    
    - `/proc/cpuinfo`
        
    - `/proc/1234` → info about process 1234
        

---

### `/sys`

Another virtual filesystem for **hardware and kernel info**.

- Used a lot by modern tools and drivers
    

---

### `/boot`

Files needed to **boot the system**.

- Kernel (`vmlinuz`)
    
- Bootloader configs (GRUB)
    

---

### `/media` and `/mnt`

Mount points for filesystems.

- `/media` → auto-mounted USBs, CDs
    
- `/mnt` → manual mounts (often temporary)
    

---

### `/opt`

Optional / third-party software.

- Example: `/opt/google/chrome`
    

---

## 🧠 How to remember it

- **Commands** → `/bin`, `/sbin`, `/usr/bin`
    
- **Configs** → `/etc`
    
- **Users** → `/home`
    
- **Logs & changing data** → `/var`
    
- **Devices** → `/dev`
    
- **Kernel & system info** → `/proc`, `/sys`
    

---
