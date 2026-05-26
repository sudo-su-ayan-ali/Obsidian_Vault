# 🔑 Files and Streams in Linux

Every process in Linux automatically has **three “streams”** open, known as file descriptors (FDs):

- **0 → stdin** (standard input) → where the process _reads_ from (keyboard by default).
- **1 → stdout** (standard output) → where the process _writes normal output_ (terminal by default).
- **2 → stderr** (standard error) → where the process _writes errors_ (also terminal by default).

👉 You, the hacker, can **redirect** these streams anywhere—files, other commands, null void.

---
# 📂 File Basics

- `cat file.txt` → view contents.
- `touch file.txt` → create empty file.
- `rm file.txt` → delete file (careful 🔥).
- `cp src dest` → copy.
- `mv old new` → move or rename.

But the true power is **redirection**.

---
# 🎯 Output Redirection

### 1. Redirect stdout to a file

```bash
echo "Hacker log entry" > log.txt
```

- `>` → creates/overwrites file.
- `>>` → appends instead of overwriting.

```bash
echo "Another entry" >> log.txt
```

Now `log.txt` accumulates logs like a data journal.

---
### 2. Redirect stderr

By default, errors show up in the terminal:

Bash

```
ls doesnotexist
```

Send errors to a file:

```bash
ls doesnotexist 2> errors.txt
```

Now `errors.txt` contains the complaint.

Append errors:

```bash
ls nope 2>> errors.txt
```

---
### 3. Redirect both stdout and stderr

- Redirect separately:

```bash
command > out.txt 2> err.txt
```

- Redirect BOTH into same file:

```bash
command > all.txt 2>&1
```

- Newer shortcut:

```bash
command &> all.txt
```

---
# 🎯 Input Redirection

Instead of typing input interactively, feed it from a file:

```bash
sort < unsorted.txt
```

This will _read_ unsorted.txt as stdin.

---
# 🔗 Pipes (`|`)

This is the **hacker’s duct tape**: chain commands like modular weapons.  
The stdout of one becomes the stdin of another.

Examples:

```bash
cat /etc/passwd | grep bash
```

Find only the lines containing `bash`.

```bash
ps aux | grep firefox | wc -l
```

Count how many firefox processes are running.

---
# 🎛 Combining Redirection with Pipes

You can mix and match! For instance:

```bash
grep "ERROR" app.log | sort | uniq > errors_sorted.txt
```

→ Extract errors, sort them, remove duplicates, write to file.

Or:

```bash
command 2>&1 | tee combined.log
```

→ Send both stdout and stderr to screen **and** save them to `combined.log`.

(`tee` is like a “T‑junction”—output goes down two paths at once.)

---
# 🕳️ The Black Hole of Linux: `/dev/null`

This file is the **void**—data written here disappears forever. Useful when you don’t want noise cluttering your screen.

Examples:

```bash
command > /dev/null        # silence output
command 2> /dev/null       # silence errors
command &> /dev/null       # silence everything
```

---
# ⚔️ Fun Hacker‑ish Examples

### Redirect command output to a timestamped logfile:

```bash
dmesg > "log_$(date +%F_%T).txt"
```

### Silence normal output, only see errors:

```bash
my_prog > /dev/null
```
### Monitor a growing file in real-time:

```bash
tail -f /var/log/syslog
```

### Split stdout between file and screen:

```bash
ping -c 4 google.com | tee ping.log
```
### Run something noisy but keep only errors:

```bash
make target 1> /dev/null 2> build_errors.log
```

---
# 🕶 Hacker’s mindset tip

- Think of **streams** like laser beams—you can redirect them through mirrors (pipes) or into glass jars (files).
- Redirection is **non-destructive automation**: you don’t copy files, you _reroute rivers of text_.
- Combine redirection + functions + loops, and you create **scripts that log, filter, and analyze** without ever touching a GUI.

---

# 🚀 Challenge

Create a command pipeline that:

1. Lists all files in `/etc`,
2. Filters out only those containing the substring `conf`,
3. Counts them,
4. Logs both the full filtered list and the total count into separate files (`conf_files.txt`, `conf_count.txt`).

---

