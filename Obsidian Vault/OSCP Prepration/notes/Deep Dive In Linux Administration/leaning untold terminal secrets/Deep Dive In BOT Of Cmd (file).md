## 🕵️ Basic Use


```bash
$ file filename
```

Example:

```bash
$ file notes.txt
notes.txt: ASCII text
```

Another:

```bash
$ file picture.png
picture.png: PNG image data, 800 x 600, 8-bit colormap, non-interlaced
```

Notice how it doesn’t just say “this is picture.png”—it analyzes the file contents and reports _type, encoding, and other juicy details_.

---

## ⚙️ How It Works

Most systems rely on **magic numbers**, which are unique signatures at the start of a file.

- `.png` begins with specific hex bytes
- `.elf` binaries (executables in Linux) start with `ELF`
- Text files often tell by being plain ASCII/UTF-8 instead of gibberish bits

➡️ So `file` inspects the _content_, not the _name_. That’s why it can call out fakes.

---

## 🔑 Common Results You’ll See

- `ASCII text`, `UTF-8 Unicode text` → plain text (what you’d open in any editor).
- `Bourne-Again shell script, ASCII text executable` → a Bash script.
- `ELF 64-bit LSB executable` → a compiled binary program (Linux’s version of `.exe`).
- `JPEG image data`, `PNG image data` → picture files.
- `gzip compressed data` or `Zip archive` → compressed archives.
- `PDF document, version 1.7` → self-explanatory.

---
## 🔮 Advanced Moves

1. **Check Multiple Files at Once**


```bash
file *
```

Scans every file in the current directory, super handy when you find a pile of mysterious loot.

2. **Identify Hidden Executables**


```bash
file suspicious_file
```

Even if someone renamed `backdoor.sh` to `cat_picture.jpg`, `file` will out it immediately.

3. **Applications in Forensics/Hacking**

- Uploads named `.php.jpg`? `file` reveals the truth.
- You stumble on a weird binary with no extension? `file` identifies if it’s a program or malware.
- You need to know if a file will open in a text editor or crash it → `file` first.

---

## **NOTE***
## 🧙 Hacker Sensei Wisdom

Think of `ls` and `file` like a tag team:

- **`ls -lh`** tells _what it’s called, size, permissions_.
- **`file`** tells _what it really is under the hood_.

This saves you time, prevents accidents, and uncovers sneaky tricks (pretending files).

---

So now—with **`ls`, `cd`, `cat`, and `file`**—you’ve got reconnaissance, movement, opening, and scanning mastered. That’s basically the four elements of hacker bending. 🌍🔥🌊💨