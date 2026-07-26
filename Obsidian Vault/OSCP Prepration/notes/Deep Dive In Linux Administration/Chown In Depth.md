## 🏆 `chown` — **Change Owner**

The command is used to change the **owner** (user) and/or **group** of files and directories.

### Basic Structure

text

```
chown [new_owner] file
chown [new_owner]:[new_group] file
```

### Examples

1. **Change the owner only**:

text

```
chown alice report.txt
```

Now `alice` owns the file. She holds the keys.

2. **Change group ownership**:

text

```
chown :developers data.csv
```

Now the group `developers` owns it (but the user owner remains unchanged).

3. **Change both user and group**:

text

```
chown alice:developers project/
```

Ownership set to user `alice` and group `developers`. Together, they form a permissions duo.

---
## 📂 Important Variations

### Recursive Option (`-R`)

This applies ownership changes to entire directories (including everything inside).

text

```
chown -R bob:staff /var/www
```

Owner: `bob`, group: `staff` for `/var/www` _and all its files and subfolders_. Like declaring, “This entire castle belongs to Bob now.”

---
## 🔎 Ownership in `ls -l`

You’ve seen this format before:

text

```
-rwxr-xr--  1 alice developers 1234 Jun 12 10:15 secret.txt
```

Here:

- **alice** → user/owner of the file.
- **developers** → group that owns it.

So if you run:

text

```
chown bob:admins secret.txt
```

It becomes:

text

```
-rwxr-xr--  1 bob admins 1234 Jun 12 10:15 secret.txt
```

---
## 🚦 Permissions vs Ownership

Let’s put it into a hacker metaphor:

- `chown` = changes _who owns the house_. (The deed gets reassigned.)
- `chmod` = changes _what people can do_ inside the house. (Knock down the walls, lock some rooms, etc.)

If you’re root (superuser), you can reassign ownership however you like, even snatching away someone else’s files like an all-powerful wizard. If you’re a normal user, you generally **can’t `chown` files to someone else** (for security reasons).

---
## 🧩 Hacker Practice Mission

1. Make a directory: `mkdir hackers_den`
2. Create some files inside: `touch hackers_den/file1 hackers_den/file2`
3. As root (or with `sudo`):
    - `chown -R $USER:users hackers_den` → claim ownership for yourself.
    - `ls -l` to verify.
4. Try `chmod g+w hackers_den` → give write powers to the group.
5. Then `chown root:root file1` → make file1 sacred, only root owns it. Notice the difference in your ability to edit it afterward.

---
