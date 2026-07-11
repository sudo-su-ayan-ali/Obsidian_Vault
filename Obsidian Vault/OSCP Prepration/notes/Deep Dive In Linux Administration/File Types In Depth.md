## 🌱 Core File Types

### 1. **Regular Files (`-`)**

- These are your everyday files: text documents, source code, images, compiled binaries, logs.
- Regular files don’t really care _what’s inside_; Linux just stores raw bytes. It’s the programs (like text editors, compilers) that interpret meaning.
- `ls -l` will show them starting with `-`.

Useful checks:

Bash

```
file file.txt   # tells you what type of content Linux *thinks* it has
stat file.txt   # detailed metadata: size, blocks, ownership, timestamps
```

---
### 2. **Directories (`d`)**

- Fancy word for “folders,” but in Linux, a directory is actually **a file that contains pointers** (lists of filenames and inode numbers).
- When you type `ls`, you’re really asking Linux to read the “table of contents” inside that directory file.
- Example:

Bash

```
ls -ldi /etc
```

The number shown is the **inode** where the directory metadata lives.

---
### 3. **Symbolic Links (`l`)**

- Think of them like **shortcuts** in Windows or **magical mirrors** that point to other files.
- If the target moves or is deleted, the symlink becomes “broken” (dangling).
- Symbolic links have their own inodes, but they store the pathname of the target instead of data.

Bash

```
ln -s /etc/passwd my_fake_passwd
ls -l my_fake_passwd
```

You’ll see it pointing with an arrow → `/etc/passwd`.

---
### 4. **Character Devices (`c`)**

- Represent devices that communicate by **streaming data one character at a time**.
- Examples: `/dev/tty` (terminal), `/dev/random` (random number generator).
- These don’t hold data, they’re gateways to drivers.

Bash

```
ls -l /dev/tty
```

Notice the `c` at the start.

Hacker note: `/dev/random` and `/dev/urandom` are used to generate cryptographic randomness—feeding entropy like a digital slot machine.

---
### 5. **Block Devices (`b`)**

- Represent hardware devices that move data in **blocks** (chunks).
- Examples: hard disks, SSDs `/dev/sda`, partitions `/dev/sda1`.
- Used by the kernel to actually read/write sectors of storage.

Bash

```
ls -l /dev/sda
```

You’ll see it starts with `b`.

---
### 6. **Socket Files (`s`)**

- Special files that enable communication between processes.
- Used for networking, or local inter‑process chat rooms.
- For example: the systemd journal uses sockets, and databases like MySQL often create a socket file, like `/var/run/mysqld.sock`.
- They allow processes to connect without needing actual network packets over TCP/UDP.

Inspect:

Bash

```
ls -l /var/run | grep sock
```

---
### 7. **Named Pipes (FIFOs) (`p`)**

- A “First In, First Out” communication channel.
- Think of them like **mail tubes in an office**: one process writes in, another reads out, in order.
- Unlike sockets, they don’t allow complex 2‑way communication across systems—just a simple queue inside the same machine.

Create one:

Bash

```
mkfifo mypipe
ls -l
```

You’ll see the `p` type.

---
## 🧠 Hacker worldview

- **Regular & Directories** → the “civilian population” of the filesystem.
- **Symlinks** → sneaky spies pointing elsewhere.
- **Character & Block devices** → wormholes into hardware.
- **Sockets & FIFOs** → dark alley meeting spots for processes to whisper secrets.

---

## 🕵️ Bonus: Find them all in action

Try:

Bash

```
ls -l /dev | head
ls -l /var/run | head
```

You’ll see block devices, char devices, sockets, FIFOs… the whole secret city under `/`.

---
