## Basic usage

```bash
locate filename
```

Example:

```bash
locate secrets.txt
```

This searches the **locate database**, not your disk in real time.

---

## Case-insensitive search (use this a lot)

```bash
locate -i filename
```

Example:

```bash
locate -i seclists
```

---

## Match part of a path

```bash
locate bin/python
```

Output example:

```
/usr/bin/python
/usr/bin/python3
```

---

## Limit results (sanity saver)

```bash
locate seclists | head
```

Or:

```bash
locate seclists | less
```

---

## Only show files that still exist

Useful if your database is slightly out of date:

```bash
locate -e filename
```

---

## Use wildcards (quotes matter)

```bash
locate "*.conf"
```

Without quotes, the shell expands it and ruins your day.

---

## Regex mode (power move)

```bash
locate -r 'seclists.*txt'
```

Example:

```bash
locate -r '/usr/share/.*wordlist'
```

---

## Find a directory specifically

```bash
locate -i seclists/
```

Trailing `/` helps narrow it to directories.

---

## Refresh the database manually

If you _know_ a file exists but `locate` doesn’t find it:

```bash
sudo updatedb
```

---

## When NOT to use locate

- You just created the file **seconds ago**
    
- You need permissions-aware results
    
- You’re searching removable media
    

In those cases, use:

```bash
find / -name filename 2>/dev/null
```

---

## Quick cheat sheet

```bash
locate file
locate -i file
locate -e file
locate "*.ext"
locate -r 'regex'
sudo updatedb
```

