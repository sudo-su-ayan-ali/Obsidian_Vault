## 🏆 The Basics of `chmod`

`chmod` = **change mode** (i.e., change permissions on a file/directory).

Every file/directory has **three categories of people** and **three types of permissions**:

### Categories

- **u** = user (the owner of the file)
- **g** = group (users in the file’s group)
- **o** = others (everyone else)
- **a** = all (a shortcut for u+g+o)

### Permissions

- **r** = read
- **w** = write
- **x** = execute

> Think of it as doors:

- **r** = see inside
- **w** = rearrange or change stuff
- **x** = actually enter and run

---
## 📊 Permission Structure

When you type `ls -l`, you’ll see something like:

text

```
-rwxr-x---
```

Breakdown:

- First character: `-` means file, `d` means directory.
- Then _triplets_:
    - `rwx` → permissions for **user** (owner).
    - `r-x` → permissions for **group**.
    - `---` → permissions for **others**.

So `-rwxr-x---` means:

- Owner can read/write/execute.
- Group can read & execute (but not write).
- Others = no permissions (access denied, peasant!).

---
## ⚙ Ways to Use `chmod`

### 1) Symbolic Method

You manipulate permissions using letters.

Examples:

- `chmod u+x script.sh` → give execute permission to the owner.
- `chmod g-w secret.txt` → remove write permission from the group.
- `chmod o=r public.txt` → set "others" to exactly read-only.
- `chmod a+x tool.sh` → everyone gains execute rights.

This feels like directly telling the guard what to do.

---
### 2) Numeric (Octal) Method

This is where things get hacker-cool. Each permission has a number:

- `r = 4`, `w = 2`, `x = 1`.

Add them together:

- `7 = rwx`
- `6 = rw-`
- `5 = r-x`
- `4 = r--`
- `0 = ---`

So `chmod 750 script.sh` means:

- Owner = 7 (rwx).
- Group = 5 (r-x).
- Others = 0 (---).

Legendary shorthand in the hacker world.

---
### 3) Special Permissions (Boss Level)

Now it gets spicy. Besides the usual r,w,x, there are three special "magical runes":

1. **Setuid (4xxx)** → If set on an executable, when anyone runs it, they run it with the permissions of the file’s owner.  
    Example: `chmod 4755 program` means that even if you’re a normal peasant, when you execute it, you borrow the file owner’s superpowers.
    
2. **Setgid (2xxx)** → On executables, process runs with group permissions. On directories, new files automatically inherit the directory’s group.  
    Example: shared project folder magic.
    
3. **Sticky bit (1xxx)** → On directories, means users can only delete/move their own files, not others.  
    Common in `/tmp` so chaos doesn’t erupt.
    

---
## 🧩 Hacker Practice Mission

1. Create a file: `touch hackdoor.sh`
2. Give yourself (user) full control, group only read+execute, others none. That’s `chmod 750 hackdoor.sh`.
3. Double-check with: `ls -l hackdoor.sh`.
4. Add execute permission for everyone symbolically (`chmod a+x hackdoor.sh`).
5. Remove other’s execute permission only (`chmod o-x hackdoor.sh`).

---
