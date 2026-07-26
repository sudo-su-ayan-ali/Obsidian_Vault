# 🔹 Part 1 — Processes (Lifecycle & Management)

### What is a Process?

- A **process** = a running program (instance of an executable).
    
- Every process has:
    
    - **PID** (Process ID)
        
    - **PPID** (Parent PID)
        
    - **UID/GID** (User & Group IDs)
        
    - **Priority/Nice Value**
        
    - **State** (Running, Sleeping, Zombie, Stopped)
        

---

### 📌 Inspecting Processes

```bash
ps aux         # BSD style (all processes) ps -ef         # System V style
```

Key fields:

- `USER` → owner
    
- `PID` → process ID
    
- `%CPU / %MEM` → usage
    
- `STAT` → state
    
    - `R` Running
        
    - `S` Sleeping
        
    - `D` Uninterruptible sleep
        
    - `Z` Zombie
        
    - `T` Stopped (suspended)
        

Interactive tools:

```bash
top            # default process viewer htop           # colorful, interactive (scroll, kill, sort)
```

---

### 📌 Managing Processes

- **Start**: `./program &` → run in background
    
- **List jobs**: `jobs -l`
    
- **Suspend**: `Ctrl+Z`
    
- **Background**: `bg %1`
    
- **Foreground**: `fg %1`
    

Signals:

```bash

kill -SIGTERM 1234      # graceful termination (default) kill -9 1234            # force kill (SIGKILL) 
kill -STOP 1234         # pause process 
kill -CONT 1234         # resume process
```


🔑 Tip: Always try `TERM` before `KILL` — `KILL` doesn’t allow cleanup.

---

### 📌 Priority & Scheduling

- **Niceness** ranges from `-20` (highest priority) to `19` (lowest).
    
- Default is `0`.
    

Examples:

```bash
nice -n 10 myscript.sh         # start with lower priority 
renice -n 5 -p 2345            # adjust running process
```

⚠️ Only **root** can set negative nice values.

---

### 📌 Orphan & Zombie Processes

- **Orphan** → parent dies, process gets adopted by `init` (`systemd`).
    
- **Zombie** → process finished, but parent didn’t collect exit code.
    
    - Shows up as `Z` in `ps`.
        
    - Fixed when parent calls `wait()`. If parent is dead, `systemd` reaps it.
        

---

# 🔹 Part 2 — Permissions (Access Control)

Linux permission system is **user-based**, with three categories:

- **Owner (u)**, **Group (g)**, **Others (o)**
    

---

### 📌 Viewing Permissions

```bash
ls -l file.txt 
-rwxr-xr--  1 alice devs  120 Sep 14 10:00 file.txt
```

Breakdown:

- `-` → regular file (d=directory, l=symlink, c=char dev, b=block dev)
    
- `rwx` (owner)
    
- `r-x` (group)
    
- `r--` (others)
    

---

### 📌 Modifying Permissions

```bash
chmod u+x script.sh         # add execute for owner 
chmod go-rw secrets.txt     # remove read/write for group & others 
chmod 750 file              # rwx for owner, rx for group, none for others
```

Octal codes:

- `4` = read, `2` = write, `1` = execute
    
- `7` = rwx, `6` = rw-, etc.
    

---

### 📌 Ownership

```bash
chown bob:staff file.txt    # change owner & group 
chown :admins file.txt      # change group only
```

---

### 📌 Special Permission Bits

- **Setuid (s)** → run program as file owner
    
    - Example: `/usr/bin/passwd` → runs as root to update `/etc/shadow`.
        
    - `chmod u+s file`
        
- **Setgid (s)** → files created in dir inherit group
    
    - Useful for shared directories.
        
    - `chmod g+s dir`
        
- **Sticky bit (t)** → on a directory, only owner can delete their files
    
    - Example: `/tmp` has `drwxrwxrwt`.
        

---

### 📌 Default Permissions & `umask`

`umask` defines _which permissions are removed_ from new files.

Example:

```bash
umask 022 
touch test.txt 
ls -l test.txt 
-rw-r--r--  ...
```

Calculation:

- Default file perms = `666` (rw-rw-rw-)
    
- Default dir perms = `777` (rwxrwxrwx)
    
- Apply umask → remove perms.  
    `666 - 022 = 644` → rw-r--r--
    

---

# 🔹 Part 3 — Scheduling (Automation & Timing)

---

### 📌 At jobs (run once in future)

```bash
at now + 1 hour 
at> echo "Backup done at $(date)" >> /tmp/log.txt 
at> <Ctrl+D>
```

- List: `atq`
    
- Remove: `atrm <jobid>`
    

---

### 📌 Cron jobs (repeated scheduling)

- Cron syntax:

```
* * * * * command
│ │ │ │ │
│ │ │ │ └─ Day of week (0-6, Sunday=0)
│ │ │ └─── Month (1-12)
│ │ └───── Day of month (1-31)
│ └─────── Hour (0-23)
└───────── Minute (0-59)
```

    

Example:

```bash
# Edit user’s cron table 
crontab -e  # Backup every day at 2 AM 
0 2 * * * rsync -a /home /backup
```


Commands:

```bash
crontab -l   # list current cron jobs 
crontab -r   # remove all jobs
```

---

### 📌 Systemd Timers (modern alternative)

Instead of cron, systemd uses timers:

```bash
# /etc/systemd/system/backup.service 
[Unit] 
Description=Daily backup  
[Service] 
ExecStart=/usr/local/bin/backup.sh
```

```bash
# /etc/systemd/system/backup.timer
[Unit] 
Description=Run backup daily  
[Timer] 
OnCalendar=daily 
Persistent=true  
[Install] 
WantedBy=timers.target
```

Commands:

```bash 
sudo systemctl enable --now backup.timer systemctl list-timers
```

Benefits:

- Logs in `journalctl`
    
- Can persist missed runs
    
- Follows dependency rules
    

---

# 🔹 Quick Summary

- **Processes**: `ps`, `top`, `kill`, `nice`, `renice`, signals, zombies/orphans.
    
- **Permissions**: `chmod`, `chown`, umask, setuid/setgid/sticky bit.
    
- **Scheduling**: `at` for one-time, `cron` for repetitive, `systemd timers` for modern systems.
