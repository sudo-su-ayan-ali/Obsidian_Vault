# 1. Scan Types (How Nmap Probes Targets)

These define **how packets are sent** to determine port states.

---

## `-sS` — TCP SYN Scan (Stealth Scan)

**Most popular scan**

**How it works**

* Sends a SYN packet
* If SYN-ACK → port is **open**
* If RST → **closed**
* Never completes the TCP handshake

**Why it’s good**

* Fast
* Stealthy (often not logged as a full connection)
* Accurate

**Requirements**

* Needs root/admin (raw packets)

**When to use**

* Default choice for serious scanning

---

## `-sT` — TCP Connect Scan

**Full TCP handshake**

**How it works**

* Uses OS `connect()` call
* Completes full TCP connection

**Pros**

* No root required
* Reliable

**Cons**

* Loud (logged by services)
* Slower than SYN

**Use when**

* You don’t have admin privileges

---

## `-sU` — UDP Scan

**Scans UDP services**

**How it works**

* Sends UDP packets
* ICMP unreachable → closed
* No response → open|filtered

**Pain points**

* VERY slow
* Many false positives

**Use when**

* Looking for DNS, SNMP, NTP, DHCP, etc.

---

## `-sA` — ACK Scan

**Firewall detection**

**What it does**

* Determines if ports are **filtered**
* Does NOT tell you open vs closed

**Use case**

* Mapping firewall rules
* Seeing which ports are statefully filtered

---

## `-sF`, `-sX`, `-sN` — FIN / Xmas / Null

**TCP flag manipulation scans**

| Scan | Flags set     |
| ---- | ------------- |
| FIN  | FIN           |
| NULL | none          |
| XMAS | FIN, PSH, URG |

**Purpose**

* Evade some firewalls
* Identify closed ports via RFC behavior

**Modern reality**

* Mostly blocked
* Still useful in niche environments

---

## `-sM` — Maimon Scan

Rare, obscure TCP edge-case scan
Mostly academic now.

---

# 2. Target Specification (Who You Scan)

---

## Single target

```bash
nmap 192.168.1.10
```

## Multiple targets

```bash
nmap 192.168.1.1 192.168.1.2
```

## CIDR

```bash
nmap 192.168.1.0/24
```

## Ranges

```bash
nmap 192.168.1.1-100
```

## Input file

```bash
nmap -iL targets.txt
```

---

## Exclusions

```bash
--exclude 192.168.1.5
--exclude-file exclude.txt
```

---

# 3. Port Selection (What You Scan)

---

## `-p`

Specify ports

```bash
-p 80
-p 22,80,443
-p 1-65535
```

---

## `--top-ports <N>`

Scans most common ports

```bash
--top-ports 100
```

---

## `-F`

Fast scan (top ~100 ports)

---

## `--port-ratio <ratio>`

Only ports seen on X% of hosts

Advanced recon optimization.

---

# 4. Service & Version Detection

---

## `-sV` — Service Version Detection

**Identifies software & versions**

```bash
nmap -sV target
```

**How**

* Sends protocol-specific probes
* Matches responses against Nmap DB

**Options**

* `--version-intensity 0-9`
* Higher = slower but more accurate

---

## `--version-light`

Faster, fewer probes

## `--version-all`

Maximum aggression

---

# 5. OS Detection

---

## `-O`

Detect operating system

**How**

* TCP/IP stack fingerprinting
* TTL, window size, flags, etc.

**Needs**

* At least one open & one closed port
* Root privileges

---

## `--osscan-guess`

More aggressive guessing

## `--osscan-limit`

Only scan promising targets

---

# 6. NSE Scripts (Nmap’s Power Tool)

---

## `-sC`

Run default scripts

```bash
nmap -sC target
```

Includes:

* Auth checks
* SSL info
* SMB discovery
* HTTP titles

---

## `--script <name>`

```bash
--script http-enum
--script smb*
```

---

## Script categories

```bash
--script auth
--script vuln
--script safe
--script intrusive
```

⚠️ **`vuln` and `intrusive` can crash services**

---

## Script arguments

```bash
--script-args user=admin,pass=1234
```

---

# 7. Timing & Performance

---

## `-T0` → `-T5`

Timing templates

| Level | Name       |
| ----- | ---------- |
| T0    | Paranoid   |
| T1    | Sneaky     |
| T2    | Polite     |
| T3    | Normal     |
| T4    | Aggressive |
| T5    | Insane     |

**Reality**

* `-T4` is the sweet spot
* `-T5` causes packet loss

---

## Fine-tuning

```bash
--min-rate 1000
--max-rate 5000
--max-retries 2
--host-timeout 5m
```

Used in **high-speed recon or evasion**

---

# 8. Firewall & IDS Evasion

---

## Fragment packets

```bash
-f
```

## Custom MTU

```bash
--mtu 24
```

---

## Decoys

```bash
-D RND:10
```

Spoofs fake scanner IPs

---

## Source port spoofing

```bash
--source-port 53
```

Tricks weak firewalls

---

## Randomize targets

```bash
--randomize-hosts
```

---

# 9. Output Formats

---

## Normal

```bash
-oN scan.txt
```

## Grepable

```bash
-oG scan.grep
```

## XML

```bash
-oX scan.xml
```

## All formats

```bash
-oA scan
```

---

# 10. Verbosity & Debugging

---

## `-v`, `-vv`

More detail

## `-d`, `-dd`

Debug mode

## `--packet-trace`

See raw packets

## `--reason`

Why Nmap thinks a port is open/closed

---

# 11. Misc Useful Flags

---

## `-n`

No DNS resolution (faster)

## `-R`

Force DNS resolution

## `--open`

Show only open ports

## `--resume`

Resume interrupted scan

---

# TL;DR Cheat Combo (Most Used)

```bash
nmap -sS -sV -O -sC -T4 -p- --open target
```

---


Heck yes 🧪😈 — NSE is where Nmap stops being “a port scanner” and starts being a **mini exploitation framework**.

Below is a **category-by-category breakdown**, with:

- what each category is _for_
    
- what kinds of scripts live there
    
- real examples
    
- risk level (very important)
    

I’ll also point out which ones are safe for routine recon vs “don’t run this on prod unless you like phone calls”.

---

# NSE SCRIPT CATEGORIES (IN DEPTH)

You can list them anytime with:

```bash
nmap --script-help all
```

---

## 1. `default` 🟢

**Automatically run with `-sC`**

### Purpose

Basic discovery and light enumeration with **minimal risk**.

### Typical behavior

- Banner grabbing
    
- Service metadata
    
- Light auth checks
    
- Non-intrusive queries
    

### Examples

- `http-title` – page title
    
- `ssh-hostkey` – SSH fingerprints
    
- `ssl-cert` – TLS certificate details
    
- `ftp-anon` – anonymous FTP check
    

### Use it when

- Doing first-pass recon
    
- Scanning production systems
    
- You want “maximum info, minimum drama”
    

✅ **Safe**  
🧠 **This should be muscle memory**

---

## 2. `safe` 🟢

**Explicitly non-intrusive**

### Purpose

Gather info **without changing state or stressing services**.

### What “safe” means

- No brute force
    
- No fuzzing
    
- No exploitation
    
- No DoS risk (in theory)
    

### Examples

- `http-headers`
    
- `http-methods`
    
- `dns-recursion`
    
- `smb-os-discovery`
    
- `snmp-info`
    

### Command

```bash
nmap --script safe target
```

### Use it when

- Corporate environments
    
- Compliance scans
    
- You need written permission constraints
    

⚠️ Still not “guaranteed harmless” (bugs exist), but _very low risk_

---

## 3. `discovery` 🔵

**Find things, don’t touch them**

### Purpose

Discover:

- Hosts
    
- Services
    
- Users
    
- Shares
    
- Network topology
    

### Examples

- `smb-enum-shares`
    
- `ldap-rootdse`
    
- `snmp-brute` (yes, discovery, but note risk)
    
- `dns-service-discovery`
    
- `mysql-users`
    

### Interesting note

Some discovery scripts **blur into enumeration or brute force**, depending on config.

### Use it when

- Mapping internal networks
    
- Finding attack surface
    
- Pre-exploitation recon
    

⚠️ **Moderate risk depending on service**

---

## 4. `version` 🔵

**Enhances `-sV`**

### Purpose

Get **better service identification** and metadata.

### How it works

- Runs _after_ version detection
    
- Service-specific probes
    

### Examples

- `http-server-header`
    
- `mysql-info`
    
- `postgresql-info`
    
- `ssh2-enum-algos`
    

### Command

```bash
nmap -sV --script version target
```

### Use it when

- Fingerprinting tech stacks
    
- Matching vulnerabilities to versions
    

🟢 Usually safe

---

## 5. `auth` 🟡

**Authentication-related checks**

### Purpose

- Weak credentials
    
- Anonymous access
    
- Default accounts
    

### Examples

- `ftp-anon`
    
- `http-auth`
    
- `smb-auth`
    
- `mongodb-auth`
    

### Command

```bash
nmap --script auth target
```

### Risks

- Can trigger lockouts
    
- Can raise alerts
    
- Some scripts _attempt_ logins
    

⚠️ **Medium risk**

---

## 6. `brute` 🔴

**Credential brute forcing**

### Purpose

Actively brute-force credentials using wordlists.

### Examples

- `ssh-brute`
    
- `ftp-brute`
    
- `http-brute`
    
- `snmp-brute`
    

### Command

```bash
nmap --script brute target
```

### Script args (common)

```bash
--script-args userdb=users.txt,passdb=passes.txt
```

### Risks

- Account lockouts
    
- IDS/IPS triggers
    
- Legal trouble if unauthorized
    

🔥 **High risk**  
🚨 **Never run blindly**

---

## 7. `vuln` 🔴

**Vulnerability detection (not exploitation)**

### Purpose

Check for **known vulnerabilities** using fingerprints or probes.

### Examples

- `smb-vuln-ms17-010` (EternalBlue)
    
- `http-vuln-cve2017-5638` (Struts)
    
- `ssl-heartbleed`
    
- `dns-zone-transfer`
    

### Command

```bash
nmap --script vuln target
```

### Important distinction

- Most scripts **do not exploit**
    
- Some **crash fragile services**
    

⚠️ **High risk**  
📞 This is where SOC teams notice

---

## 8. `intrusive` 🔴

**May disrupt or modify services**

### Purpose

- Deep probing
    
- Stress testing
    
- Behavior analysis
    

### Examples

- `http-form-fuzzer`
    
- `dns-fuzz`
    
- `smb-fuzz-share`
    

### Characteristics

- Sends malformed input
    
- Repeated requests
    
- Can cause service instability
    

🔥 **Very high risk**  
❌ Avoid on production

---

## 9. `exploit` 🚨

**Active exploitation**

### Purpose

Exploit vulnerabilities to:

- Execute code
    
- Bypass auth
    
- Dump data
    

### Examples

- `http-shellshock`
    
- `smb-vuln-ms08-067`
    
- `ftp-vsftpd-backdoor`
    

### Reality check

- Many are outdated
    
- Success rate varies
    
- Still very dangerous
    

🚨 **DO NOT run unless explicitly authorized**

---

## 10. `dos` 💣

**Denial of Service**

### Purpose

Test resilience by attempting to crash services.

### Examples

- `http-slowloris`
    
- `ssl-dos`
    
- `smb-flood`
    

### Impact

- Can hang or crash targets
    
- Can affect entire networks
    

💣 **Maximum risk**  
❌ Almost never appropriate

---

## 11. `malware` 🧟

**Detect malware / backdoors**

### Purpose

Identify:

- Known backdoors
    
- Botnet C2 behavior
    
- Compromised services
    

### Examples

- `smtp-strangeport`
    
- `http-malware-host`
    
- `irc-botnet-channels`
    

### Use it when

- Incident response
    
- Threat hunting
    

🟡 Medium risk, high value

---

## 12. `fuzzer` 🧨

**Protocol fuzzing**

### Purpose

Send malformed input to:

- Trigger crashes
    
- Reveal parsing bugs
    

### Examples

- `dns-fuzz`
    
- `rtsp-fuzz`
    

### Risks

- Crashes
    
- Undefined behavior
    

🧨 **Extremely risky**

---

## 13. `external` 🌐

**Uses third-party services**

### Purpose

Query **external APIs** for intel.

### Examples

- `whois-ip`
    
- `ip-geolocation-*`
    
- `asn-query`
    

### Risks

- Data leakage
    
- Depends on internet access
    

🟢 Generally safe

---

## 14. `broadcast` 📡

**No target required**

### Purpose

Discover hosts via **broadcast traffic**.

### Examples

- `broadcast-dhcp-discover`
    
- `broadcast-avahi-dos` (yep, DoS lives here sometimes)
    
- `broadcast-netbios-master-browser`
    

### Command

```bash
nmap --script broadcast
```

⚠️ Can be noisy on LANs

---

# HOW PROS ACTUALLY USE NSE

### Safe recon pass

```bash
nmap -sC --script safe,discovery target
```

### Vulnerability assessment

```bash
nmap -sV --script vuln target
```

### Controlled auth testing

```bash
nmap --script auth --script-args=user=admin target
```

---

# NSE PRO TIP 🧠

You can **combine categories + exclusions**:

```bash
--script "default,safe,discovery and not brute"
```

---
