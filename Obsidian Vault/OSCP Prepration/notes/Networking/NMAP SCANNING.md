# 🚀 **Nmap Scanning Deep Dive**

---

## 1️⃣ **The Basics: What Is Nmap Really Doing?**

- At its core, Nmap sends carefully crafted packets to a host or network, then analyzes **responses (or lack of them).**
- From those responses, Nmap figures out:
    - Which hosts are alive (host discovery / “ping scan”).
    - Which ports are open, closed, or filtered.
    - Which services and versions are running.
    - Potential OS and device signatures.
    - And if you turn on scripting — possible known vulnerabilities.

So: Nmap = detective, network = crime scene, packets = questions, responses = clues.

---

## 2️⃣ **Host Discovery (Finding Live Targets)**

Before scanning ports, Nmap first checks if a host is up. Methods:

- ICMP echo requests (ping).
- TCP SYN to port 443 or 80.
- ARP requests on local LAN.

🕶 Hackers use “ping sweeps” here to map the field. Defenders filter ICMP to make hosts less chatty.

---

## 3️⃣ **Port States**

When Nmap probes a port, it can classify it as:

- **Open:** Service responds → “I’m listening.”
- **Closed:** Host responds, but “no one’s home.”
- **Filtered:** No response (firewall ate the packet).
- **Unfiltered:** Got a response, but can’t tell.
- **Open | Filtered:** Not enough info to know.

For attackers: _open = exploitable, closed = harmless but confirms machine is there, filtered = interesting (firewall detected)._

---

## 4️⃣ **Scan Techniques (Nmap’s “Magic Spells”)**

These are packet‑level scans Nmap can perform:

### 🟢 a) **TCP Connect Scan** (`-sT`)

- Uses full TCP handshake (SYN → SYN/ACK → ACK).
- Reliable, but noisy (target logs the connection).

### 🟢 b) **TCP SYN Scan** (`-sS`) **(Default, the Hacker’s Sweetheart)**

- Sends SYN packets, waits for SYN/ACK (open) or RST (closed).
- Never completes handshake — stealthier.
- Also called “half‑open scan.”

### 🟢 c) **UDP Scan** (`-sU`)

- Sends UDP packets; no reply = maybe open, ICMP “port unreachable” = closed.
- Tricky, slow, lots of “false positives.”

### 🟢 d) **Other Special Scans**

- **FIN scan (`-sF`):** Sends only FIN flag.
- **Xmas scan (`-sX`):** Sets FIN, PSH, URG (making packet “twinkle” like a tree).
- **Null scan (`-sN`):** No flags set.  
    ➡ Exploits quirks where some OSes respond incorrectly. Useful for **firewall evasion**.

### 🟢 e) **Ping Sweep** (`-sn`)

- Simply pings hosts to see who’s up (no port scan).

---

## 5️⃣ **Service & Version Detection**

- `-sV` tells Nmap: not just “port 22 open,” but **which daemon/version**:
    - e.g., _OpenSSH 7.2 (Ubuntu)_ or _Apache httpd 2.4.46_.

This is critical for attackers (find vulnerable versions) _and_ defenders (inventory exact software).

---

## 6️⃣ **OS Fingerprinting**

- `-O` option: Nmap compares responses to its OS signature database.
- Example output: _Linux 4.x kernel, 85% confidence._
- Crucial for narrowing down payload exploits.

---

## 7️⃣ **Nmap Scripting Engine (NSE) – “Recon Bomb”**

- Lets you run handy scripts directly from Nmap.
- Categories:
    - **Discovery** (grab info about SSL certificates, HTTP titles, SMB shares).
    - **Vulnerability** (check for Heartbleed, SQL injection, etc.).
    - **Authentication** (bruteforce logins).
    - **Malware** (check known backdoors).

Example:

- `nmap --script=http-title,ssl-cert -p 443 target.com`

For blue teams: great way to pen‑test yourself. For red teams: devastating when abused.

---

## 8️⃣ **Stealth & Evasion Options**

Hackers don’t want alarms going off — Nmap supports:

- **Decoys (`-D`)** → makes scan look like it's coming from many IPs.
- **Fragmentation (`-f`)** → splits probes into small packets to bypass packet filters.
- **Timing Options (`-T0`–`-T5`)** → control stealth vs. speed tradeoff.
- **Idle Scan (`-sI`)** → bounces scan via a “zombie host” so the target never sees the real source.

---

## 9️⃣ **Interpreting Output**

Basic scan:

text

```
PORT     STATE  SERVICE  VERSION
22/tcp   open   ssh      OpenSSH 7.4 (protocol 2.0)
80/tcp   open   http     Apache httpd 2.4.6
3306/tcp open   mysql    MySQL 5.7
```

One glance: You see OS fingerprints, exposed databases, versions ready to be matched against CVE lists.

---

## 🔦 Hacker vs. Defender Mentality

- **Hacker:** Nmap = “map the attack surface.” → Find open services, detect versions, pick exploits.
- **Defender:** Nmap = “map exposure _before_ the hacker.” → Close extra ports, log scans, harden services.

---

# 🎤 Analogy

Imagine **a burglar (or auditor)** walking down a skyscraper’s hallway:

- Knocks on each door (port scanning)
- Listens if someone shouts back “Yes?” (open).
- Peeks at the doorplate: “Office of Apache Inc. v2.4.46” (version detection).
- Tries different ways to jiggle handles without being noticed (stealth scans).
- Calls in tools to check for rattling windows (NSE vulnerability scripts).

---

# ✨ Summary

- **Nmap isn’t just “is port open?” — it’s reconnaissance, fingerprinting, and even pre‑exploitation.**
- Core scanning methods: Connect, SYN, UDP, Xmas/FIN/Null.
- Rich info: Service detection, OS detection, scripting engine.
- Dual use: White hats audit & defend; black hats enumerate & exploit.

---
