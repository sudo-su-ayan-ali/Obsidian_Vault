## 1. What hashing really is (clear + exam-safe)

**Hashing = converting input data into a fixed-length output using a one-way function.**

Key properties:

- ❌ **Not reversible**
    
- ✅ Same input → same hash
    
- ❌ Different input → _ideally_ different hash
    
- ❌ No key involved
    

### OSCP exam sentence you should memorize

> Hashing is used for **verification**, not secrecy.

If it’s reversible → **not hashing**.

---

## 2. Hashing vs Encoding vs Encryption (OSCP trap alert)

|Type|Reversible|Used for|
|---|---|---|
|Encoding|✅ Yes|Transport|
|Encryption|✅ Yes (key)|Confidentiality|
|**Hashing**|❌ No|Password storage|

If you see:

- Base64 → decode
    
- AES → decrypt (if key)
    
- **MD5/SHA → crack**
    

---

## 3. Why hashing matters in OSCP+

You WILL see hashes in:

- `/etc/shadow`
    
- Database dumps
    
- Web app backups
    
- Config files
    
- Password reset tokens
    
- Windows SAM dumps
    

OSCP expects you to:

1. **Identify hash type**
    
2. **Crack it**
    
3. **Reuse creds**
    
4. **Escalate privileges**
    

---

## 4. Common hash types you must recognize

### Linux / Web

|Hash|Example|Notes|
|---|---|---|
|MD5|`5f4dcc3b5aa765d61d8327deb882cf99`|Very weak|
|SHA1|`cbfdac6008f9cab4083784cbd1874f76618d2a97`|Weak|
|SHA256|long hex|Slower|
|bcrypt|`$2y$10$...`|Strong|
|yescrypt|`$y$...`|Modern Linux|

### Windows

|Hash|Example|
|---|---|
|NTLM|`aad3b435b51404eeaad3b435b51404ee`|

OSCP tip:

> If it’s **unsalted MD5/SHA1**, you’re basically done.

---

## 5. How hashing works (enough depth for OSCP)

### Example

```text
password123
```

MD5:

```text
482c811da5d5b4bc6d497ffa98491e38
```

Change ONE character:

```text
password124
```

MD5:

```text
c6c13d6c66a6a0e90a46a70b1b6d8b7c
```

This is the **avalanche effect**.

---

## 6. Salting (VERY IMPORTANT)

### Without salt

```text
password → hash
```

Same password = same hash (bad)

### With salt

```text
salt + password → hash
```

Example:

```text
$1$salt$hash
```

OSCP reality:

- Web apps often **don’t salt properly**
    
- Linux shadow **does**
    
- You still crack via **wordlists + rules**
    

---

## 7. Practical #1: Cracking Linux hashes (/etc/shadow)

### Shadow entry

```text
john:$6$abc123$K0YyYl1F...:19000:0:99999:7:::
```

`$6$` → SHA512

### Crack with John

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt shadow.txt
```

Show results:

```bash
john --show shadow.txt
```

🎯 OSCP goal:

> Get plaintext → SSH → sudo → root

---

## 8. Practical #2: Cracking MD5 / SHA hashes

### Given hash

```text
5f4dcc3b5aa765d61d8327deb882cf99
```

### Crack with Hashcat

```bash
hashcat -m 0 -a 0 hash.txt rockyou.txt
```

Result:

```text
password
```

Modes you should know:

- `-m 0` → MD5
    
- `-m 100` → SHA1
    
- `-m 1400` → SHA256
    
- `-m 1800` → SHA512
    
- `-m 1000` → NTLM
    

---

## 9. Practical #3: Web app database dump

### users table

```text
admin | e99a18c428cb38d5f260853678922e03
```

Looks like MD5.

Crack:

```bash
hashcat -m 0 hash.txt rockyou.txt
```

Output:

```text
abc123
```

Next steps (OSCP flow):

- Login as admin
    
- Find file upload / RCE
    
- Shell
    
- PrivEsc
    

---

## 10. Practical #4: Windows NTLM hashes

### Dumped hash

```text
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
```

### Crack NTLM

```bash
hashcat -m 1000 ntlm.txt rockyou.txt
```

OR pass-the-hash:

```bash
evil-winrm -i 10.10.10.5 -u Administrator -H <NTLM_HASH>
```

🔥 No cracking needed.

---

## 11. Practical #5: Identifying hash type

### Use hashid

```bash
hashid 5f4dcc3b5aa765d61d8327deb882cf99
```

Or:

```bash
hashcat --help | grep MD5
```

OSCP exam advice:

> Correct hash identification saves HUGE time.

---

## 12. Why strong hashes still fall (OSCP reality)

Even bcrypt/sha512 fall when:

- Weak passwords
    
- Reused credentials
    
- Small wordlists
    
- Predictable patterns
    

That’s why **rockyou + rules** works.

---

## 13. Hashing attack chain (real OSCP flow)

1. LFI → config file
    
2. DB creds → dump users
    
3. Hashes → crack
    
4. Reused password → SSH
    
5. sudo misconfig → root
    

Hashing is **not isolated** — it’s a pivot tool.

---

## 14. Common OSCP mistakes (avoid these)

❌ Treating hashes like encryption  
❌ Not trying pass-the-hash  
❌ Ignoring weak hashes  
❌ Forgetting hash modes  
❌ Not reusing cracked creds

---

## 15. One-page OSCP+ hashing checklist

✔ Identify hash type  
✔ Crack with rockyou  
✔ Try creds everywhere  
✔ Attempt pass-the-hash  
✔ Use hashes to pivot

---

## 16. Quick practice (do this now)

Crack:

```text
e10adc3949ba59abbe56e057f20f883e
```

Answer:

```text
123456
```

---

