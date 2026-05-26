# 1. What Encryption _Really_ Is (Pentester Mindset)

**Encryption = controlled secrecy**  
It transforms readable data (**plaintext**) into unreadable data (**ciphertext**) using:

- an **algorithm** (math)
    
- a **key** (secret)
    

What OSCP cares about:

- **Where encryption is used**
    
- **How keys are generated, stored, and reused**
    
- **How encryption fails in real systems**
    

> 🔥 You almost never “crack” strong encryption — you **bypass, steal keys, downgrade, or exploit bad implementations**.

---

# 2. Symmetric Encryption (AES etc.)
## 2.1 What Symmetric Encryption _Actually_ Means

**Symmetric encryption** uses **one single secret key** to:

- Encrypt plaintext → ciphertext
    
- Decrypt ciphertext → plaintext
    

If you have the key, you have **everything**.

> 🧠 Pentester mindset:  
> **You don’t break the algorithm — you steal, guess, reuse, or misuse the key.**

---

## 2.2 Core Building Blocks (Critical for OSCP+)

Symmetric encryption is **not just “AES”**. It’s a combo of parts:

### A. Algorithm

Examples:

- AES (modern standard)
    
- DES (broken)
    
- 3DES (deprecated)
    

AES is standardized by **NIST** and is **cryptographically strong**.

👉 **OSCP reality**: If AES is implemented correctly, you won’t crack it.

---

### B. Secret Key

- Random bytes
    
- Length matters (AES-128, AES-256)
    

Bad keys = easy attacks:

- Short keys
    
- Predictable keys
    
- Hardcoded keys
    
- Password-derived keys
    

---

### C. Mode of Operation (VERY IMPORTANT)

AES alone encrypts **only 128-bit blocks**.  
Modes define _how blocks are chained_.

|Mode|Secure?|Why OSCP Cares|
|---|---|---|
|ECB|❌|Patterns leak|
|CBC|⚠️|IV reuse, padding oracle|
|CTR|⚠️|Nonce reuse breaks security|
|GCM|✅|Authenticated encryption|

> 🔥 Many real exploits come from **bad modes**, not bad algorithms.

---

### D. IV / Nonce (Initialization Vector)

- Random value added to encryption
    
- Prevents identical plaintext → identical ciphertext
    

**Rules**:

- Must be random
    
- Must never repeat (in some modes)
    

---

## 2.3 How AES Encryption Works (Conceptual)

Let’s say:

```
Plaintext:  "admin=true"
Key:        K
IV:         R
Mode:       CBC
```

Process:

1. Plaintext split into blocks
    
2. First block XORed with IV
    
3. Encrypted with key
    
4. Output feeds into next block
    

➡️ Change IV or key → ciphertext completely changes

---

## 2.4 Practical Encryption Example (Linux)

### Encrypt

```bash
openssl enc -aes-256-cbc -salt -pbkdf2 \
-in secrets.txt -out secrets.enc
```

### Decrypt

```bash
openssl enc -aes-256-cbc -d -pbkdf2 \
-in secrets.enc -out secrets.txt
```

**What matters to OSCP**:

- `-pbkdf2` → key derivation (slow brute-force)
    
- Without it → faster cracking
    

---

## 2.5 Where Symmetric Encryption Is Used (Real Targets)

You WILL see it in:

- Config files
    
- Database credentials
    
- API tokens
    
- JWT payload encryption
    
- Backup archives
    
- Malware configs
    
- HTTPS session data (after handshake)
    

---

## 2.6 OSCP+ Attack Scenarios (This Is the Gold)

### 1️⃣ Hardcoded AES Keys (Classic)

```python
KEY = b"mysecretkey12345"
cipher = AES.new(KEY, AES.MODE_CBC, iv)
```

**Attack**:

- Read source code
    
- Extract key
    
- Decrypt everything
    
- Forge new encrypted data
    

> ✔ No crypto attack needed — **logic failure**

---

### 2️⃣ Password-Based Encryption (Very Common)

```bash
openssl enc -aes-256-cbc -in data -out data.enc -pass pass:admin123
```

**Why this is weak**:

- Password ≠ cryptographic key
    
- Often no salt
    
- Often no key stretching
    

**Attack**:

- Offline brute-force
    
- `john` / `hashcat`
    

---

### 3️⃣ ECB Mode (Pattern Leakage)

ECB encrypts **each block independently**.

Same plaintext block → same ciphertext block

That’s why images encrypted with ECB still show shapes.

> 🧨 OSCP trick: detect ECB by repeated blocks

```bash
xxd encrypted.bin | sort | uniq -d
```

Repeated blocks = ECB = bad crypto.

---

### 4️⃣ IV Reuse (Silent Killer)

If the same IV is reused:

- Patterns leak
    
- XOR attacks become possible
    

Common bug:

```python
iv = b"\x00" * 16
```

💀 Static IV = broken encryption.

---

### 5️⃣ Padding Oracle Attacks (CBC Mode)

CBC requires padding.  
If the app:

- Returns different errors for bad padding
    
- Leaks timing differences
    

➡️ You can **decrypt data byte-by-byte without the key**.

OSCP LOVES this conceptually (even if not always exploited directly).

---

### 6️⃣ “Encrypted” Auth Tokens

```json
{
  "user": "admin",
  "role": "user"
}
```

Encrypted with AES.

**Attack path**:

1. Find key in app
    
2. Decrypt token
    
3. Change role → admin
    
4. Re-encrypt
    
5. Privilege escalation
    

---

## 2.7 Encryption vs Authentication (Critical Distinction)

Encryption alone does NOT prevent tampering.

Bad design:

- AES-CBC without MAC
    

Good design:

- AES-GCM (encrypt + authenticate)
    

If no integrity check:

- Attacker can modify ciphertext
    
- App decrypts manipulated data
    

---

## 2.8 Base64 Is NOT Encryption (OSCP Meme)

```bash
echo YWRtaW46cGFzcw== | base64 -d
```

That’s encoding.

> 🚩 If a dev says “encrypted” and you see base64 → instant vuln.

---

## 2.9 OSCP+ Mental Checklist for Symmetric Crypto

When you see AES, ask:

✔ Where is the key stored?  
✔ Is it hardcoded?  
✔ Is it reused?  
✔ Is it password-derived?  
✔ Which mode is used?  
✔ Is the IV random and unique?  
✔ Is integrity verified?

If any answer is bad → **you have an attack path**.

---

## 2.10 One-Line OSCP+ Truth

> **Strong symmetric encryption fails almost exclusively because humans implemented it badly.**


---

# 3. Asymmetric Encryption (RSA, ECC)
# Asymmetric Encryption (Public-Key Crypto) — OSCP+ Deep Dive

## 3.1 What Asymmetric Encryption _Actually_ Is (Beyond the Definition)

At its core:

- **Two keys**
    
    - **Public key** → shared with the world
        
    - **Private key** → kept secret
        
- What one key encrypts, **only the other can decrypt**
    

This solves a huge real-world problem:

> _How do two strangers securely communicate over an untrusted network?_

From an OSCP+ perspective, this is everywhere:

- SSH access
    
- HTTPS
    
- Kerberos
    
- VPNs
    
- Secure file transfers
    
- Code signing
    
- Passwordless authentication
    

---

## 3.2 Why Asymmetric Encryption Exists (Threat Model Thinking)

Before asymmetric crypto, symmetric encryption had a fatal flaw:

> **Key distribution problem**

If I want to send you an AES key:

- How do I send it securely **before** encryption exists?
    

Asymmetric crypto fixes this:

- Anyone can encrypt using the **public key**
    
- Only the owner can decrypt using the **private key**
    

💡 **OSCP mindset**:  
Asymmetric crypto doesn’t remove risk — it **moves the attack surface**:

- Private key theft
    
- Weak permissions
    
- Bad implementations
    
- Poor trust validation
    
- Misconfigured services
    

---

## 3.3 The Math (Only What You Actually Need)

You **do not** need to derive formulas for OSCP, but you must understand the _assumptions_ attackers exploit.

### Example: RSA (Most Common)

RSA security relies on:

- Factoring a **very large number** is computationally infeasible
    

Key idea:

```
public_key = (n, e)
private_key = (n, d)
```

Where:

- `n = p × q` (two huge primes)
    
- If attacker factors `n`, game over
    

💥 **Attack relevance**

- Small key sizes (1024-bit RSA)
    
- Weak primes
    
- Leaked private keys
    
- Reused keys
    

---

## 3.4 Where You See Asymmetric Encryption in Real Attacks

### 🔐 SSH Authentication

SSH uses asymmetric crypto **for authentication**, not bulk encryption.

#### How it works:

1. Client proves it owns the **private key**
    
2. Server checks against `authorized_keys`
    
3. No password needed
    

#### OSCP Goldmine:

- Stolen `id_rsa`
    
- Weak permissions on `.ssh/`
    
- Private keys embedded in backups, Git repos, containers
    

---

### 🌐 HTTPS / TLS Handshake

TLS uses **both** asymmetric and symmetric crypto.

#### Simplified TLS Flow:

1. Server sends **public key**
    
2. Client verifies certificate
    
3. Client creates symmetric session key
    
4. Session key encrypted with server’s public key
    
5. All further traffic uses fast symmetric crypto (AES)
    

💡 **Key point**:  
Asymmetric crypto is **slow** → only used for **key exchange**

---

## 3.5 OSCP+ Practical: SSH Key Abuse

### 🔎 Finding SSH Keys

Common places:

```bash
/home/*/.ssh/
/root/.ssh/
~/.ssh/
/var/backups/
/opt/
/tmp/
/mnt/
```

Search:

```bash
find / -name "id_rsa" 2>/dev/null
```

---

### 🔓 Private Key Permissions Matter

If private key permissions are too open:

```bash
-rw-r--r--  id_rsa
```

You can use it:

```bash
chmod 600 id_rsa
ssh -i id_rsa user@target
```

🎯 **OSCP lesson**:  
Crypto can be perfect — **file permissions ruin everything**

---

## 3.6 OSCP+ Practical: TLS Certificate Abuse

### 🕵️ Inspect Certificates

```bash
openssl s_client -connect target:443
```

Look for:

- Weak algorithms
    
- Expired certs
    
- Self-signed certs
    
- Internal hostnames
    

---

### 🔐 Private Key Leakage

If you ever find:

```bash
server.key
```

That’s catastrophic:

- Decrypt traffic
    
- Impersonate server
    
- MITM attacks
    

---

## 3.7 Asymmetric Encryption vs Digital Signatures (OSCP Critical)

Same math — different direction.

### Encryption:

```
Encrypt with public key → decrypt with private key
```

### Signature:

```
Sign with private key → verify with public key
```

#### OSCP Examples:

- SSH authentication
    
- JWT signatures
    
- Software update verification
    
- Malware signing to evade defenses
    

---

## 3.8 Common OSCP+ Misconfigurations

### 🚨 Reused Keys

- Same SSH key on multiple servers
    
- Same TLS cert across environments
    

### 🚨 Weak Key Sizes

- RSA 1024
    
- Old DSA keys
    

### 🚨 Trust Abuse

- User blindly accepts unknown SSH fingerprint
    
- Ignoring TLS warnings
    

### 🚨 Backup Leakage

- Private keys in `.tar.gz` or `.zip` backups
    

---

## 3.9 OSCP Attack Chain Example

> Realistic exam-style scenario:

1. Gain low-priv shell
    
2. Find `/home/dev/.ssh/id_rsa`
    
3. Key belongs to `root`
    
4. SSH into target as root
    
5. Full compromise
    

Crypto didn’t fail.  
**Key management failed.**

---

## 🔥 OSCP+ Memory Hook (Remember This)

> **Asymmetric crypto rarely breaks — implementations do**

If you remember only one thing:

- **Steal keys**
    
- **Abuse trust**
    
- **Exploit misconfigurations**
    
- **Crypto is only as strong as its storage**
    

---

# 4. Hashing vs Encryption (OSCP LOVES THIS)
## 4.1 Core Difference (Attacker Mindset)

| Feature      | Hashing                     | Encryption           |
| ------------ | --------------------------- | -------------------- |
| Reversible?  | ❌ No (one-way)              | ✅ Yes (two-way)      |
| Uses keys?   | ❌ No                        | ✅ Yes                |
| Main purpose | Password storage, integrity | Confidentiality      |
| If leaked…   | Crack or replay             | Decrypt if key found |

💡 **OSCP rule of thumb**

> If you’re _supposed_ to get the original data back → **encryption**  
> If you’re _not_ → **hashing**

---

## 4.2 Hashing Explained (What Defenders Intend)

### What hashing does

- Takes input of any length
    
- Produces fixed-length output
    
- Same input → same output
    
- Cannot be reversed mathematically
    

Example:

```text
password123 → ef92b778bafe771e89245b89ecbc08a44a4e166c06659911881f383d4473e94f
```

Used for:

- Password storage
    
- File integrity
    
- Digital signatures
    
- Malware detection
    

---

## 4.3 OSCP Reality: Hashing Is _Meant_ to Be Attacked

### Common password hashes you’ll see

| Hash     | Where              |
| -------- | ------------------ |
| MD5      | Old web apps       |
| SHA1     | Legacy systems     |
| SHA256   | APIs, config files |
| NTLM     | Windows            |
| bcrypt   | Linux, modern apps |
| yescrypt | Newer Linux        |

---

## 4.4 OSCP+ Practical: Hash Cracking

### 🔎 Finding hashes

Linux:

```bash
/etc/shadow
```

Windows:

```bash
SAM
SYSTEM
```

Databases:

```sql
users.password
```

Config files:

```bash
*.env
config.php
```

---

### 🔓 Identifying hash type

```bash
hashid hash.txt
```

or

```bash
hashcat --example-hashes
```

---

### 🔥 Cracking with Hashcat

Example: NTLM

```bash
hashcat -m 1000 hashes.txt rockyou.txt
```

Example: bcrypt

```bash
hashcat -m 3200 hashes.txt rockyou.txt
```

⚠️ **OSCP truth**  
Strong hashing algorithms **can’t be reversed** — only **guessed**.

Attack works because:

- Weak passwords
    
- Reused passwords
    
- No rate limits
    

---

## 4.5 Salts (Why Rainbow Tables Fail)

A **salt** is random data added to the password before hashing.

```text
hash = H(password + salt)
```

### Defender goal:

- Prevent precomputed attacks
    

### Attacker reality:

- Salts do NOT stop cracking
    
- They only slow it down
    

Hashcat handles salts automatically.

---

## 4.6 Encryption Explained (What Attackers Love)

### What encryption does

- Data is **reversible**
    
- Requires a **key**
    
- Same plaintext can encrypt differently
    

Used for:

- HTTPS
    
- SSH
    
- VPNs
    
- Database encryption
    
- API secrets
    

---

## 4.7 OSCP+ Practical: Encryption Abuse

### 🔑 Symmetric Encryption

Same key encrypts & decrypts.

Example (AES):

```bash
openssl enc -aes-256-cbc -d -in secrets.enc -out secrets.txt -k password
```

🎯 **OSCP target**

- Hardcoded keys
    
- Weak passwords
    
- Keys stored next to encrypted data
    

---

### 🔐 Asymmetric Encryption

Public/private key pairs.

Used in:

- SSH
    
- TLS
    
- Kerberos
    

#### OSCP goldmine:

- Private keys (`id_rsa`, `.pem`, `.key`)
    
- Bad permissions
    
- Backup leaks
    

---

## 4.8 OSCP Exam Scenario: Hashing vs Encryption Confusion

### Scenario A

You find:

```text
password = e99a18c428cb38d5f260853678922e03
```

✅ That’s a **hash**  
➡️ Crack it

---

### Scenario B

You find:

```text
password = U2FsdGVkX1+P9Q8...
```

✅ That’s **encrypted**  
➡️ Find the key

---

### Scenario C

You find:

```bash
ENCRYPTION_KEY=SuperSecretKey123
password=U2FsdGVkX1...
```

🎯 **Instant win**

- Decrypt
    
- Login
    
- Pivot
    

---

## 4.9 Integrity vs Confidentiality (Exam Favorite)

| Goal             | Tool        |
| ---------------- | ----------- |
| Hide data        | Encryption  |
| Detect tampering | Hashing     |
| Authenticate     | Hash + salt |
| Prove identity   | Signature   |

Example:

- Malware uses hashing to verify payload integrity
    
- HTTPS uses encryption to hide traffic
    

---

## 🔥 OSCP Attack Chain Example

1. SQLi dumps user table
    
2. Passwords are hashed (bcrypt)
    
3. Crack one weak password
    
4. Password reuse → SSH login
    
5. Find encrypted config file
    
6. Key stored in `.env`
    
7. Decrypt API token
    
8. Privilege escalation
    

---

## 🧠 Memory Hooks (Burn These In)

> 🔐 **Encryption hides data — find the key**  
> 🔑 **Hashing hides passwords — guess the input**

> ❌ You don’t “decrypt” hashes  
> ✅ You “crack” hashes

> Crypto doesn’t fail — **humans do**

---

## 🚀 OSCP+ Checklist (Use This in the Exam)

### When you see a weird string:

- Can it be decrypted?
    
- Is there a key nearby?
    
- Is it base64 or encrypted?
    
- Is it a known hash?
    

### When you see passwords:

- Hash type?
    
- Weak algorithm?
    
- Password reuse?
    
- Online login possible?
    


---

# 5. TLS / HTTPS (Where Real Attacks Happen)

# 5.1. What HTTPS _really_ is (no marketing lies)

**HTTPS = HTTP + TLS**

- **HTTP** → application protocol (requests, responses)
    
- **TLS** → cryptographic tunnel underneath
    

TLS does **three critical things**:

1. **Confidentiality** – encrypts data
    
2. **Integrity** – prevents tampering
    
3. **Authentication** – proves the server is who it claims to be
    

If any of these fail, HTTPS becomes **HTTP with extra steps**.

---

# 5.2. TLS architecture (what’s actually happening on the wire)

Think of TLS as **two phases**:

## Phase 1: Handshake (asymmetric crypto)

Used to:

- Agree on encryption algorithms
    
- Authenticate the server
    
- Securely exchange keys
    

## Phase 2: Data transfer (symmetric crypto)

Used to:

- Encrypt actual HTTP traffic efficiently
    

💡 **Asymmetric crypto is slow → only used briefly**

---

# 5.3. TLS Handshake (packet-level mental model)

Here’s the classic TLS 1.2 handshake (still VERY relevant in OSCP labs):

```
Client → Server: ClientHello
Server → Client: ServerHello
Server → Client: Certificate
Server → Client: ServerHelloDone
Client → Server: ClientKeyExchange
Client → Server: ChangeCipherSpec
Server → Client: ChangeCipherSpec
```

Let’s break this down **like a pentester**, not a cryptographer.

---

## 5.3.1 ClientHello (recon phase)

The client says:

- TLS versions it supports
    
- Cipher suites it supports
    
- Compression methods
    
- Random nonce
    

📌 **Pentest relevance**

- Weak TLS versions offered (SSLv3, TLS 1.0)
    
- Weak cipher suites accepted (RC4, DES, EXPORT)
    
- This is what tools like `sslscan` and `nmap` fingerprint
    

---

## 5.3.2 ServerHello + Certificate (trust model)

Server replies with:

- Chosen TLS version
    
- Chosen cipher suite
    
- **X.509 certificate**
    

Certificate contains:

- Server public key
    
- Domain name(s)
    
- Issuer (CA)
    
- Validity period
    

### Certificate trust chain:

```
Server cert → Intermediate CA → Root CA (trusted by OS)
```

📌 **Pentest relevance**

- Expired certs
    
- Self-signed certs
    
- Wrong CN / SAN mismatch
    
- Weak key sizes (1024-bit RSA)
    
- Internal CA leaks infrastructure names
    

---

## 5.3.3 Key Exchange (where attacks live)

### RSA key exchange (legacy)

- Client encrypts pre-master secret with server public key
    
- Server decrypts using private key
    

💀 **If private key is compromised → past traffic decryptable**

### Diffie-Hellman (modern)

- Ephemeral key exchange (DHE / ECDHE)
    
- Provides **Perfect Forward Secrecy (PFS)**
    

📌 **Pentest relevance**

- Weak DH parameters (Logjam)
    
- Non-ephemeral DH → traffic decryptable later
    

---

## 5.3.4 ChangeCipherSpec (point of no return)

After this:

- Everything is encrypted
    
- HTTP becomes invisible without keys
    

---

# 5.4. TLS versions (what OSCP expects you to know)

|Version|Status|Why you care|
|---|---|---|
|SSLv2 / SSLv3|DEAD|POODLE, DROWN|
|TLS 1.0|Weak|PCI fail, BEAST|
|TLS 1.1|Weak|Deprecated|
|TLS 1.2|Acceptable|Most labs|
|TLS 1.3|Strong|Fewer attacks|

📌 **OSCP labs still use TLS 1.0/1.1 on purpose**

---

# 5.5. Cipher suites (attack surface goldmine)

Cipher suite format:

```
TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
```

Breakdown:

- **ECDHE** → key exchange
    
- **RSA** → authentication
    
- **AES_256_GCM** → encryption
    
- **SHA384** → integrity
    

### Weak cipher red flags:

- `RC4`
    
- `DES`
    
- `3DES`
    
- `EXPORT`
    
- `NULL`
    
- `MD5`
    

---

# 5.6. Practical TLS enumeration (OSCP muscle memory)

## 5.6.1 Nmap

```
nmap --script ssl-enum-ciphers -p 443 target
```

Look for:

- TLS 1.0 enabled
    
- Weak ciphers accepted
    
- EXPORT-grade support
    

---

## 5.6.2 sslscan

```
sslscan target:443
```

Quick visibility into:

- Supported protocols
    
- Cipher strength
    
- Certificate issues
    

---

## 5.6.3 OpenSSL (manual handshake testing)

```
openssl s_client -connect target:443
```

Force protocol:

```
openssl s_client -tls1 -connect target:443
```

📌 **OSCP exam tip**  
This is how you prove a weak protocol is enabled **without scanners**.

---

# 5.7. HTTPS ≠ secure application (critical mindset)

TLS protects:

- Data in transit
    

TLS does NOT protect:

- SQL injection
    
- Command injection
    
- XSS
    
- File upload flaws
    
- IDOR
    
- Broken auth
    

📌 **Exam mindset**  
“HTTPS login page” ≠ “secure login”

---

# 5.8. TLS-related attacks you _must_ recognize

## 5.8.1 SSL stripping (when HTTPS isn’t enforced)

Attack:

- Downgrade HTTPS → HTTP
    
- Steal credentials
    

Detection:

- Missing HSTS
    
- Mixed content
    

---

## 5.8.2 Weak certificates

- Same cert reused across hosts
    
- Internal domain names leaked
    
- Private key leaked in web root (happens in labs)
    

---

## 5.8.3 Client-side trust abuse

- App accepts **any certificate**
    
- Mobile / thick clients skip validation
    

📌 Leads to **MITM credential theft**

---

# 5.9. Practical OSCP lab scenario walkthrough

### Scenario:

```
https://intranet.corp.local
```

### Step 1 – Cert inspection

```
openssl s_client -connect intranet.corp.local:443
```

Find:

- CN: intranet.corp.local
    
- SANs: dc01.corp.local, fileserver.corp.local
    

🔥 **New targets discovered**

---

### Step 2 – Cipher analysis

```
sslscan intranet.corp.local
```

Find:

- TLS 1.0 enabled
    
- 3DES supported
    

📌 Vulnerable to downgrade attacks (exam write-up material)

---

### Step 3 – Application attack anyway

Even with perfect TLS:

- SQLi in login form
    
- File upload bypass
    
- JWT misconfig
    

TLS didn’t save them.

---

# 5.10. OSCP+ exam mindset summary

What examiners expect you to:

- **Recognize** weak TLS configs
    
- **Enumerate** them properly
    
- **Explain** why they’re bad
    
- **Exploit around** HTTPS, not through it
    

TLS is:

> a _transport protection layer_, not a vulnerability shield

---

# 6. Encryption Failures You _Will_ Exploit

### ❌ Encrypting passwords instead of hashing

### ❌ Storing keys in source code

### ❌ Using ECB mode (pattern leakage)

### ❌ Reusing IVs

### ❌ Using base64 and calling it “encryption” 🤡

```bash
echo "admin:password" | base64
```

That’s **encoding**, not encryption.

---

# 7. OSCP+ Style Mental Checklist

When you see encryption, ask:

✔ Where is the key stored?  
✔ Who can read the key?  
✔ Is it reused?  
✔ Is it password-derived?  
✔ Can I downgrade crypto?  
✔ Is something “encrypted” that should be hashed?

> If crypto is strong, **attack the implementation**.

---

# 8. Mini OSCP+ Practice Scenario

**You find:**

```php
$key = "P@ssw0rd!";
$data = openssl_decrypt($token, "AES-256-CBC", $key);
```

**Exploit path**:

1. Extract key
    
2. Decrypt tokens
    
3. Forge admin tokens
    
4. Privilege escalation
    

---
