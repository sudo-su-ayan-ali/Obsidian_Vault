# Cryptography for OSCP / OSCP+ – Study Guide

> Focus: Attacker mindset, real-world exploitation, not crypto math.

---

## 0. Crypto Fundamentals

### 0.1 What Cryptography Tries to Achieve
- Confidentiality
- Integrity
- Authentication
- Non-repudiation

### 0.2 Attacker Mindset for Crypto
- Identify misuse, not strength
- Look for implementation flaws
- Assume developers made mistakes

---

## 1. Encoding vs Encryption vs Hashing (VERY IMPORTANT)

### 1.1 Encoding
- Purpose: Data representation
- Reversible
- Not security
- Examples:
  - Base64
  - Hex
  - URL Encoding
- OSCP Skills:
  - Identify encoded data
  - Decode manually or with tools

### 1.2 Encryption
- Purpose: Confidentiality
- Reversible with key
- Examples:
  - AES
  - DES
  - RSA
- OSCP Skills:
  - Identify weak encryption
  - Find hardcoded keys
  - Abuse bad crypto usage

### 1.3 Hashing
- Purpose: Integrity / Password storage
- One-way
- Examples:
  - MD5
  - SHA1
  - SHA256
  - bcrypt
- OSCP Skills:
  - Identify hash types
  - Crack weak hashes

---

## 2. Symmetric Encryption

### 2.1 Basics
- Same key for encryption and decryption
- Fast and commonly used

### 2.2 Common Algorithms
- AES (secure if used correctly)
- DES (broken)
- 3DES (weak)

### 2.3 Modes of Operation
- ECB (INSECURE)
- CBC
- CFB
- OFB
- GCM

### 2.4 Important Concepts
- Key
- IV (Initialization Vector)
- Padding

### 2.5 OSCP Attack Focus
- ECB mode pattern leaks
- Hardcoded encryption keys
- Reused IVs
- Secrets in config files

---

## 3. Asymmetric Encryption

### 3.1 Basics
- Public key + Private key
- Used for key exchange and authentication

### 3.2 Common Algorithms
- RSA
- DSA
- ECC

### 3.3 Where You See It
- SSH
- TLS/SSL
- Certificates

### 3.4 OSCP Attack Focus
- Exposed private keys
- Weak RSA key sizes
- SSH key misuse
- Certificate leaks

---

## 4. Hashing & Password Attacks

### 4.1 Common Hash Types
- MD5 (broken)
- SHA1 (broken)
- SHA256
- NTLM
- bcrypt (slow, strong)

### 4.2 Important Concepts
- Salt
- Rainbow tables
- Fast vs slow hashes

### 4.3 OSCP Skills
- Identify hash formats
- Decide if cracking is feasible
- Crack hashes using tools

---

## 5. Password Storage Mistakes

### 5.1 Common Developer Errors
- Unsalted hashes
- Weak hash algorithms
- Plaintext passwords
- Reused passwords

### 5.2 OSCP Attack Focus
- Lateral movement using cracked creds
- Credential reuse across services
- Privilege escalation via password reuse

---

## 6. TLS / SSL Basics

### 6.1 What TLS Does
- Encrypts data in transit
- Uses certificates and keys

### 6.2 Key Concepts
- Certificates
- Public key vs private key
- Chain of trust

### 6.3 OSCP Attack Focus
- Self-signed certificates
- Weak cipher suites
- Old TLS versions
- Extracting certificate info

---

## 7. JWT (JSON Web Tokens)

### 7.1 JWT Structure
- Header
- Payload
- Signature

### 7.2 Important Properties
- Base64 encoded (not encrypted)
- Client-side readable

### 7.3 Common Vulnerabilities
- alg: none
- Weak signing secrets
- Algorithm confusion (HS256 vs RS256)

### 7.4 OSCP Skills
- Decode JWTs
- Modify payloads
- Bypass authentication

---

## 8. Cryptography in Web Applications

### 8.1 Common Locations
- Cookies
- Authorization headers
- Hidden form fields
- API keys
- Config files

### 8.2 Red Flags
- Client-side encryption
- Base64 “encrypted” passwords
- Predictable tokens
- Hardcoded secrets

---

## 9. File & Disk Encryption (Basic)

### 9.1 Common Scenarios
- Encrypted ZIP files
- Password-protected backups
- Encrypted configuration files

### 9.2 OSCP Attack Focus
- Weak passwords
- Password reuse
- Cracking encrypted files

---

## 10. Crypto-Related Tools (Must Know)

### 10.1 Encoding & Decoding
- base64
- xxd
- CyberChef
- Burp Decoder

### 10.2 Encryption & Certificates
- openssl

### 10.3 Hash Cracking
- hashcat
- john

---

## 11. Common OSCP Crypto Scenarios

- Decode API tokens
- Crack dumped password hashes
- Exploit weak JWT implementation
- Decrypt configuration secrets
- Identify fake encryption
- Abuse reused crypto keys

---

## 12. Study Checklist

- [ ] Understand encoding vs encryption vs hashing
- [ ] Crack basic password hashes
- [ ] Identify insecure encryption usage
- [ ] Exploit JWT vulnerabilities
- [ ] Recognize TLS misconfigurations
- [ ] Use crypto tools comfortably

---

## Final OSCP Rule for Crypto

> Do not ask “Is this cryptography strong?”
> Ask “How did the developer mess this up?”

Because they always do.
