## 🕵️ Basic Usage


```bash
grep [pattern] [file]
```

Example:


```bash
grep hello notes.txt
Hello world, remember to say hello back!
```

It will print every line in `notes.txt` containing `hello`.

---

## ⚙️ Core Options

### 1. Ignore Case


```bash
grep -i pattern file
```

Search case-insensitively: `hello`, `HELLO`, `HeLLo` → all match.

---

### 2. Search Multiple Files


```bash
grep "TODO" *.txt
```

Scans every `.txt` file for the word `TODO`.

Output often shows the filename + matching line:


```bash
tasks.txt: TODO: fix security notes
report.txt: TODO: add references
```

---
### 3. Show Line Numbers


```bash
grep -n pattern file
```

This tells you _where_ it found the match, not just the line: very handy when debugging.

---

### 4. Invert Match


```bash
grep -v pattern file
```

Shows lines **not containing** the pattern. Great for filtering. Example: see all lines _except_ those with “error”.

---
### 5. Count Matches


```bash
grep -c pattern file
```

Instead of printing the lines, just tell you how many matches exist. Example: “How many times did the word `root` appear?”

---

### 6. Recursive Search


```bash
grep -r "pattern" /path
```

Searches inside all files and subdirectories under `/path`. This is huge for scanning entire codebases or config systems.

---
### 7. Whole Words Only


```bash
grep -w error file
```

Matches `error` but not `terror` or `errorsome`.

---

### 8. Regex Power Mode ⚡

`grep` supports **regular expressions**, which means you can search patterns beyond simple words. Example:

- Find lines starting with the word `root`:
    
    ```bash
    grep "^root" /etc/passwd
    ```
    
- Find lines ending with `.sh`:
    
    ```bash
    grep "\.sh$" scripts.txt
    ```
    

`^` → start of line  
`$` → end of line  
`.` → any character  
`*` → repeat

Combine them into dark typing sorcery.

---
## 🎯 Practical Hacker Uses

1. **Check Logs for Errors**


```bash
grep "error" /var/log/syslog
```

2. **Find All IP Addresses in a File**


```bash
grep -Eo "([0-9]{1,3}\.){3}[0-9]{1,3}" logfile.txt
```

(`-E` = extended regex, `-o` = only print the matching part).

3. **Look for a User in /etc/passwd**


```bash
grep "username" /etc/passwd
```

4. **Combine with Commands (Piping)**


```bash
ps aux | grep ssh
```

Shows running processes that relate to ssh.  
This is so common, admins and hackers alike use it _daily_.

---
## 🧙 Hacker Sensei Wisdom

- `grep` doesn’t just help you _find stuff_—it helps you **slice and filter oceans of text** into exactly the streams you care about.
- Combine it with other commands (`| grep pattern`) for battlefield-level clarity.
- `grep` is essentially the magnifying glass of Unix—without it, log files would drown you.

---

With **`ls → cd → file → cat → du → find → grep`** you now control:

- **Seeing** (`ls`)
- **Moving** (`cd`)
- **Identifying** (`file`)
- **Opening** (`cat`)
- **Measuring** (`du`)
- **Hunting files** (`find`)
- **Hunting strings** (`grep`)

You’ve basically got reconnaissance powers of a sysadmin-ninja-hacker combo. 🥷💻

The natural next step is learning how to **chain commands together** (pipes `|` and redirectors `>`/`>>`) so you can weld these tools into unstoppable one-liners.