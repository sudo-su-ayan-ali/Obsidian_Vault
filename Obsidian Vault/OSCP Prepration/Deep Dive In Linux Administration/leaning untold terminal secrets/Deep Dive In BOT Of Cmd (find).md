## 🔍 Basic Syntax


```bash
find [path] [conditions] [actions]
```

- **`path`** → Where to start searching (often `/`, `.` for current dir, or `/home/user`).
- **`conditions`** → What are you looking for (name, type, size, etc).
- **`actions`** → What to do with the match (print it, delete it, move it, etc).

By default, if you don’t give it conditions:


```bash
find .
```

will dump _everything_ inside the current directory and all subdirectories.

---

## 🗝️ Common Conditions

### 1. Search by Name

```bash
find . -name "notes.txt"
```

Searches for a file _exactly_ called `notes.txt` in the current directory tree.

- Use wildcards:
    
    ```bash
    find . -name "*.txt"
    ```
    
    Finds all files ending in `.txt`.

> 🔮 Tip: If you want case-insensitive:

```bash
find . -iname "*.txt"
```

That way `NOTES.TXT` and `notes.txt` both match.

---
### 2. Search by Type

text

```
find . -type d -name "Documents"
```

- `-type f` → files
- `-type d` → directories
- `-type l` → symbolic links

---

### 3. Search by Size

text

```
find . -size +100M
```

- `+100M` → files **greater than 100MB**.
- `-10k` → files smaller than 10 kilobytes.
-  `100c` → exactly 100 bytes.

---

### 4. Search by Time

- `-mtime` → modified time (in days)
- `-atime` → last accessed time (in days)
- `-ctime` → last changed permissions/ownership

Example:


```bash
find . -mtime -1
```

→ files modified in the last 1 day.

You can also do minutes with `-mmin`.


```bash
find . -mmin -30
```

→ modified in the last 30 minutes. Hacker fresh files.

---

### 5. Search by Permissions


```bash
find . -perm 644
```

Finds files with specific permission codes.

- `-perm 777` → world-readable/writable/executable (dangerous!).
 ---

## 🎯 Actions

By default, `find` just **prints paths**. But the real fun is what you _do_ once you find something.

1. **Delete on the spot (⚠ Danger!):**


```bash
find . -name "*.tmp" -delete
```

2. **Run a Command on Each Match (with `-exec`)**


```bash
find . -name "*.log" -exec ls -lh {} \;
```

- The `{}` is replaced by each found file.
- `\;` closes the `exec` command.
Another:

```bash
find /var/log -type f -name "*.log" -exec du -sh {} \;
```

→ Shows size of each `.log` file.

3. **Safer Test First**  
    Run `ls` or `echo` with `-exec` before doing destructive things like `rm`.

---
## ⚡ Killer Combos

- **Find suspicious world-writable files:**


```bash
find / -type f -perm 777
```

- **Find recently changed configs:**


```bash
find /etc -type f -mtime -2
```

- **Find big boys hogging your system:**

```bash
find /home -type f -size +500M
```

- **Find and remove “.DS_Store” Mac trash in repo:**


```bash
find . -name ".DS_Store" -delete
```

---
## ***NOTE***
	So now you’ve mastered:
	
	- **`ls`** = see nearby
	- **`cd`** = move
	- **`cat`** = open files
	- **`file`** = check file identity
	- **`du`** = measure disk hogs
	- **`find`** = sniff out anything
	
	You’re practically a filesystem ninja.

---
