### 🔑 File Types in Linux

Every file in Linux has a “type.” You can think of them as different species in the wild:

- **Regular file** (`-`) → text files, binaries, scripts, logs.
- **Directory** (`d`) → folders that hold other files.
- **Symbolic link** (`l`) → shortcuts, like teleport portals to other files.
- **Character device** (`c`) → hardware devices that send data one character at a time (e.g. keyboard).
- **Block device** (`b`) → hardware devices that send data in chunks (e.g. hard drives).
- **Socket** (`s`) → for inter‑process communication, network sockets.
- **Named pipe (FIFO)** (`p`) → a way for processes to talk, like a message tube.

👉 Run `ls -l` and look at the very first character of each line—that’s the secret symbol of the file type.

---

### 🔒 File Ownership

Every file belongs to two key identities:

1. **User (owner)** → usually the person who created the file.
2. **Group** → a collection of users, all with shared access rights.

So, a file has a **user** and a **group** assigned to it, like a bouncer who checks two different guest passes before letting you in.

Check it yourself:
Bash

```
ls -l
```

You’ll see something like:

text

```
-rw-r--r--  1 alice devs  1234 Jan 31 20:15 secrets.txt
```

Breaking it down:

- `alice` → file owner (user)
- `devs` → group that also has permissions

---

### 🛡 Permissions: The `chmod` Command

The Linux world is guarded by **three sets of permissions**:

- **User (u)** → the owner’s rights
- **Group (g)** → the group’s rights
- **Others (o)** → everyone else

And the permissions themselves are:

- **r** → read (look inside)
- **w** → write (change it)
- **x** → execute (run it like a program)

Example:

text

```
-rwxr-x---
```

Means:

- Owner: can **read, write, execute**
- Group: can **read, execute**
- Others: **no rights at all** (locked out)

#### `chmod` Styles
1. **Symbolic (letters)**:
    - `chmod u+x script.sh` → give the owner execute power.
    - `chmod g-w file.txt` → remove group’s write power.
2. **Numeric (octal)**:
    - Read=4, Write=2, Execute=1. Add them like secret codes.
    - `chmod 755 script.sh`
        - Owner: 7 (4+2+1 = rwx)
        - Group: 5 (4+0+1 = r-x)
        - Others: 5 (r-x)

---

### 🏷 File Ownership: `chown`

Sometimes you need to hand over the crown. That’s where `chown` comes in:

- `chown bob file.txt` → make Bob the new owner.
- `chown bob:devs file.txt` → make Bob the owner _and_ assign the group `devs`.

There’s also `chgrp` if you only want to change the group.

---

#### ⚡ Hacker analogy:

- File **types** tell you _what you’re looking at_ (a secret diary? a locked chest?).
- **Ownership** is _who holds the keys_.
- **chmod** is _deciding what powers each faction has_.
- **chown** is _transferring the keys to someone else_.