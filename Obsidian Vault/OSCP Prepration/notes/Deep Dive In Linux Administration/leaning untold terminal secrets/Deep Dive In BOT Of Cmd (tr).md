## 🔍 What `tr` (translate) Does

- **`tr` = translate or delete characters** from standard input, sending the cleaned/transformed stream to standard output.
- Think of it as: _take this stream of characters, swap or drop some according to my rule._

---

## ⚔️ Syntax

```bash
tr [options] SET1 [SET2]
```

- **SET1** = what you’re looking for
- **SET2** = what you’re replacing it with

If only SET1 is given (and certain options), it deletes or squeezes those.

---
## 🛠️ Common Usages

### 1. **Change Case**

- Uppercase → lowercase:
    
    ```bash
    echo "HACK THE PLANET" | tr 'A-Z' 'a-z'
    ```
    
    Output:
    
    ```bash
    hack the planet
    ```
    
- Lowercase → uppercase:
    
    ```bash
    echo "root access" | tr 'a-z' 'A-Z'
    ```
    
    Output:
    
    ```bash
    ROOT ACCESS
    ```
    

---
### 2. **Replace Single Characters**


```bash
echo "1337 h4ck3r" | tr '347' 'aek'
```

Output:


```bash
lake hacker
```

Translation: it mapped each `3 → a`, `4 → e`, `7 → k`.  
(Yes, you can “de-leetspeak” text with `tr` 🕶).

---
### 3. **Delete Characters**

Use the `-d` flag.


```bash
echo "No!#$ Secrets!@#" | tr -d '!@#$'
```

Output:

```bash
No Secrets
```

Poof—noise gone. Cleaner than a hacker’s alibi.

---
### 4. **Squeeze Repeated Characters**

With `-s` (squeeze):

```bash
echo "heeelllooooooo" | tr -s 'eo'
```

Output:


```bash
helo
```

Useful for normalizing messy input (logs, scrapes, etc.).

---
### 5. **Character Class Shenanigans**

POSIX character classes let you affect whole categories at once.

- `[:digit:]` → 0–9
- `[:alpha:]` → letters
- `[:space:]` → spaces, tabs, etc.

Examples:


```bash
echo "phone: 123-456-7890" | tr -d '[:digit:]'
```

Output:


```bash
phone: --
```

Another:


```bash
echo "mixing SPACES and TABS" | tr '[:space:]' '\n'
```

Splits the input by whitespace, one word per line.

---
## 🕶 Hacker Flavor

Hackers often combine `tr` with other commands in pipelines:

1. **Normalize logs** to only letters/numbers:
    
    
    ```bash
    cat logfile | tr -cd '[:alnum:]\n'
    ```
    
    (removes punctuation—just clean alphanumerics left).
    
2. **Base64 decoded data filtering**:  
    Sometimes decoded payloads have binary junk—strip it to readable ranges:
    
    
    ```bash
    cat payload.b64 |
    ```
	That’s how you salvage hidden human-readable strings from suspicious blobs.
    
3. **Password list cleanup**:  
    Convert everything lower for brute-force consistency:
    
    ```bash
    tr 'A-Z' 'a-z' < wordlist.txt > cleanlist.txt
    ```
    

---

## ⚡ The Hacker’s Lens

- Quick character substitution? `tr`.
- Need to nuke unwanted symbols? `tr`.
- Need to control text like it’s Play-Doh in your hands? `tr`!

It’s tiny, fast, and sneaky—a humble scalpel hiding in plain sight. In fact, many one-liners that look magical are just `cat something | tr ...` doing the heavy lifting.

---
