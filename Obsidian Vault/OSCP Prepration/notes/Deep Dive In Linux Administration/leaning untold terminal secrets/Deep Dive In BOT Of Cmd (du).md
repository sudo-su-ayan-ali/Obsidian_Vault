## 🕵️ Basic Usage


```bash
$ du [options] [path]
```

- Without options:


```bash
$ du
4       ./Documents
8       ./Downloads
12      .
```

This shows space consumed in **kilobytes**, by default, for each directory and subdirectory, ending with the total (`.`).

---
## ⚙️ Important Options

### 1. **`du -h`** (_human-readable_)

Shows size in KB, MB, GB, instead of raw kilobytes. Much nicer for actual humans:


```bash
$ du -h
4.0K    Documents
8.0K    Downloads
12K     .
```

### 2. **`du -sh`** (_summarize + human-readable_)

If you just want the total size of a folder and don’t want the breakdown:


```bash
$ du -sh Documents
2.3M    Documents
```

💡 This is the hacker’s go-to for quickly checking “How fat is this directory?”

### 3. **`du -a`** (_all files + directories_)

By default, `du` only reports directories. Adding `-a` also shows individual files:


```bash
$ du -ah
4.0K Documents/notes.txt
8.0K Downloads/song.mp3
...
```

### 4. **`du -c`** (_grand total_)

Adds a final line with the overall size:


```bash
$ du -ch
4.0K    Documents
8.0K    Downloads
12K     .
12K     total
```

### 5. **`du --max-depth=N`**

Controls how far into subdirectories `du` goes. Helpful when a directory has layers upon layers.

```bash
$ du -h --max-depth=1
2.3M Documents
600M Downloads
602M .
```

That shows _only one level deep_. Add deeper numbers for more detail.

---
## 🔮 Real-World Hacker Tricks

- **Find what’s bloated:**


```bash
du -sh *
```

Shows each item in current directory with their sizes → quick way to spot the biggest offender.

- **Hunt space-hogs system-wide:**

```bash
du -sh /var/* | sort -h
```

Lists sizes of everything in `/var/`, sorted smallest to largest. Perfect for discovering “ah, logs ate my disk again.”

- **Check logs specifically:**

```bash
du -ah /var/log | sort -h | tail -n 10
```

See the 10 biggest log files.

---
## 🧙 Hacker Sensei Wisdom

- `du` = **what space is being _used_**.
- `df` = **what space is _available_** (disk _free_).  
    Together, they’re the twin guardians of disk management. One tells you _who’s hogging your room_, the other tells you _how much room you’ve got left_.
---

So far, you’re mastering reconnaissance (`ls`, `file`), movement (`cd`), file spelunking (`cat`), and disk awareness (`du`). That’s basically hacker detective 101.