
## 🌱 **Level 1: Fundamentals**

👉 Goal: Learn filters, navigate packets, understand layers.

### 🔹 Challenge 1: Identify TCP 3‑Way Handshake

- Open your browser → visit a plain HTTP site (`http://example.com`).
- Capture traffic, apply filter:
    
    text
    
    ```
    tcp.flags.syn == 1 or tcp.flags.ack == 1
    ```
    
- Identify: **SYN, SYN‑ACK, ACK**.
- Write down sequence/ack numbers.

### 🔹 Challenge 2: Find DNS Query/Response

- Filter:
    
    text
    
    ```
    udp.port == 53
    ```
    
- Clear DNS cache, then load a site.
- Identify transaction ID, query (`A example.com`), and the response (`A → IP`).
- Bonus: Notice UDP used by default unless response is huge (then it’s TCP).

### 🔹 Challenge 3: Spot HTTP Request and Response

- Filter:
    
    text
    
    ```
    http
    ```
    
- Load an **insecure HTTP site** (not HTTPS).
- Extract: Host header, User‑Agent, HTTP status code.
- Bonus: Right‑click → Follow → HTTP Stream.

---

## 🌿 **Level 2: Intermediate Analysis**

👉 Goal: Recognize protocol behavior across layers.

### 🔹 Challenge 4: Capture HTTPS Handshake (TLS)

- Filter:
    
    text
    
    ```
    tls
    ```
    
- Visit `https://` site.
- Identify:
    - **Client Hello**
    - **Server Hello**
    - Negotiated cipher suite.
- Bonus: Find the **certificate** (expand → Certificates).

### 🔹 Challenge 5: Follow a File Transfer

- Filter:
    
    text
    
    ```
    tcp.port == 21 or ftp
    ```
    
- Use an FTP client to download a file (to/from test machine).
- Follow TCP Stream → Export payload → You literally recover the file!
- Lesson: Why unencrypted FTP is dangerous.

### 🔹 Challenge 6: Detect Retransmissions

- Filter:
    
    text
    
    ```
    tcp.analysis.retransmission
    ```
    
- Transfer a big file.
- Identify at which points packets were resent.
- Lesson: This is how you spot network congestion or packet loss.

---

## 🌳 **Level 3: Security & Recon Mode**

👉 Goal: See how attackers / defenders use Wireshark.

### 🔹 Challenge 7: Spot a Port Scan

- Run Nmap against a target (in lab).
- Filter:
    
    text
    
    ```
    tcp.flags.syn == 1 and tcp.flags.ack == 0
    ```
    
- You’ll see SYN “fans” going to many ports.
- Lesson: Detect scanning activity.

### 🔹 Challenge 8: Find Credentials in Cleartext

- Run FTP or Telnet login.
- Filter:
    
    text
    
    ```
    ftp or telnet
    ```
    
- Follow stream → Spot **username** and **password** in clear text.
- Lesson: Why insecure protocols = hacker’s candy.

### 🔹 Challenge 9: DNS Tunnel Detection

- Use a DNS tunneling tool, or simulate with weirdly long domain lookups.
- Filter:
    
    text
    
    ```
    dns
    ```
    
- Notice many suspicious, long domain queries.
- Lesson: Malware exfiltrates data this way.

---

## 🗻 **Level 4: Advanced (Threat Hunter Mode)**

👉 Goal: Put it all together — rebuild scenarios.

### 🔹 Challenge 10: Rebuild a Web Browsing Session

- Capture your HTTP/HTTPS browsing (pcap).
- For HTTP → follow streams, rebuild page elements.
- For HTTPS → analyze TLS handshake, find **SNI (Server Name Indication)** (filter: `tls.handshake.extensions_server_name`).
- Lesson: Even if payload is encrypted, metadata leaks info (like domains visited).

### 🔹 Challenge 11: Detect C2 Traffic (Malware Simulation)

- Simulate a reverse shell using `nc` (netcat).
- Capture packets.
- Identify:
    - Outbound ephemeral port connecting to attacker’s host.
    - Continuous small encrypted packets back and forth.
- Lesson: Command‑and‑control traffic looks very different from normal browsing.

### 🔹 Challenge 12: Trace the Path (Like Traceroute in Packets)

- Run `traceroute` during capture.
- Filter:
    
    text
    
    ```
    icmp or udp
    ```
    
- Watch TTL expiry + ICMP Time Exceeded messages.
- Lesson: You see each hop’s IP/MAC revealing infrastructure.

---

# 🛠️ Tools & Tricks for Wireshark Learning

- **Display Filters** (`tcp`, `udp`, `http`, `dns`, `tls`, etc.) → zoom in.
- **Coloring Rules** → highlight anomalies (e.g., SYN in red).
- **Statistics Menu** → Protocol Hierarchy, Conversations, Endpoints.
- **Expert Info** (bottom pane) → see retransmissions, dup ACKs, malformed packets.

---
## 🌱 **Level 1: Fundamentals**

👉 Goal: Learn filters, navigate packets, understand layers.

### 🔹 Challenge 1: Identify TCP 3‑Way Handshake

- Open your browser → visit a plain HTTP site (`http://example.com`).
- Capture traffic, apply filter:
    
    text
    
    ```
    tcp.flags.syn == 1 or tcp.flags.ack == 1
    ```
    
- Identify: **SYN, SYN‑ACK, ACK**.
- Write down sequence/ack numbers.

### 🔹 Challenge 2: Find DNS Query/Response

- Filter:
    
    text
    
    ```
    udp.port == 53
    ```
    
- Clear DNS cache, then load a site.
- Identify transaction ID, query (`A example.com`), and the response (`A → IP`).
- Bonus: Notice UDP used by default unless response is huge (then it’s TCP).

### 🔹 Challenge 3: Spot HTTP Request and Response

- Filter:
    
    text
    
    ```
    http
    ```
    
- Load an **insecure HTTP site** (not HTTPS).
- Extract: Host header, User‑Agent, HTTP status code.
- Bonus: Right‑click → Follow → HTTP Stream.

---

## 🌿 **Level 2: Intermediate Analysis**

👉 Goal: Recognize protocol behavior across layers.

### 🔹 Challenge 4: Capture HTTPS Handshake (TLS)

- Filter:
    
    text
    
    ```
    tls
    ```
    
- Visit `https://` site.
- Identify:
    - **Client Hello**
    - **Server Hello**
    - Negotiated cipher suite.
- Bonus: Find the **certificate** (expand → Certificates).

### 🔹 Challenge 5: Follow a File Transfer

- Filter:
    
    text
    
    ```
    tcp.port == 21 or ftp
    ```
    
- Use an FTP client to download a file (to/from test machine).
- Follow TCP Stream → Export payload → You literally recover the file!
- Lesson: Why unencrypted FTP is dangerous.

### 🔹 Challenge 6: Detect Retransmissions

- Filter:
    
    text
    
    ```
    tcp.analysis.retransmission
    ```
    
- Transfer a big file.
- Identify at which points packets were resent.
- Lesson: This is how you spot network congestion or packet loss.

---

## 🌳 **Level 3: Security & Recon Mode**

👉 Goal: See how attackers / defenders use Wireshark.

### 🔹 Challenge 7: Spot a Port Scan

- Run Nmap against a target (in lab).
- Filter:
    
    text
    
    ```
    tcp.flags.syn == 1 and tcp.flags.ack == 0
    ```
    
- You’ll see SYN “fans” going to many ports.
- Lesson: Detect scanning activity.

### 🔹 Challenge 8: Find Credentials in Cleartext

- Run FTP or Telnet login.
- Filter:
    
    text
    
    ```
    ftp or telnet
    ```
    
- Follow stream → Spot **username** and **password** in clear text.
- Lesson: Why insecure protocols = hacker’s candy.

### 🔹 Challenge 9: DNS Tunnel Detection

- Use a DNS tunneling tool, or simulate with weirdly long domain lookups.
- Filter:
    
    text
    
    ```
    dns
    ```
    
- Notice many suspicious, long domain queries.
- Lesson: Malware exfiltrates data this way.

---

## 🗻 **Level 4: Advanced (Threat Hunter Mode)**

👉 Goal: Put it all together — rebuild scenarios.

### 🔹 Challenge 10: Rebuild a Web Browsing Session

- Capture your HTTP/HTTPS browsing (pcap).
- For HTTP → follow streams, rebuild page elements.
- For HTTPS → analyze TLS handshake, find **SNI (Server Name Indication)** (filter: `tls.handshake.extensions_server_name`).
- Lesson: Even if payload is encrypted, metadata leaks info (like domains visited).

### 🔹 Challenge 11: Detect C2 Traffic (Malware Simulation)

- Simulate a reverse shell using `nc` (netcat).
- Capture packets.
- Identify:
    - Outbound ephemeral port connecting to attacker’s host.
    - Continuous small encrypted packets back and forth.
- Lesson: Command‑and‑control traffic looks very different from normal browsing.

### 🔹 Challenge 12: Trace the Path (Like Traceroute in Packets)

- Run `traceroute` during capture.
- Filter:
    
    text
    
    ```
    icmp or udp
    ```
    
- Watch TTL expiry + ICMP Time Exceeded messages.
- Lesson: You see each hop’s IP/MAC revealing infrastructure.

---

# 🛠️ Tools & Tricks for Wireshark Learning

- **Display Filters** (`tcp`, `udp`, `http`, `dns`, `tls`, etc.) → zoom in.
- **Coloring Rules** → highlight anomalies (e.g., SYN in red).
- **Statistics Menu** → Protocol Hierarchy, Conversations, Endpoints.
- **Expert Info** (bottom pane) → see retransmissions, dup ACKs, malformed packets.

---
# 🕶️ **What a Black Hat Sees in Wireshark Challenges**

---

## 🌱 **Level 1: Reconnaissance**

1. **TCP 3‑Way Handshake**
    
    - Defender sees: “Connection established normally between two hosts.”
    - **Black Hat sees:**
        - Which service replied (“Port 80 is alive!”).
        - The target’s **sequence number behavior** (helps with recon or session hijacking on poorly protected systems).
        - Identifies _active hosts_ → “This IP is worth targeting next.”
2. **DNS Query/Response**
    
    - Defender sees: Name resolution working.
    - **Black Hat sees:**
        - Every **domain victim queries**, e.g., `mail.company.com`, `vpn.company.com`.
        - Maps out the company’s **infrastructure & subdomains** = potential attack surface.
        - Knows which external services are used (Office365, Google, or local mail) → helps phishing or targeted exploits.
3. **HTTP Request & Response (Unencrypted)**
    
    - Defender sees: The page loads fine.
    - **Black Hat sees:**
        - Full **URLs, cookies, headers, login forms**, even search queries.
        - Leaks **User-Agent** (OS/browser version → used for targeted exploits).
        - Steals **session cookies** to hijack logins without a password.

---

## 🌿 **Level 2: Harvesting Sensitive Data**

4. **TLS (HTTPS) Handshake**
    
    - Defender sees: Nice, encrypted!
    - **Black Hat sees:**
        - Can’t read content, but still sees:
            - **SNI (Server Name Indication)** → tells which domain is being visited.
            - Metadata: timing, frequency, destination → “I know what site you hit, just not the content.”
        - May try downgrade attacks if old TLS versions are seen.
5. **FTP File Transfer**
    
    - Defender sees: File is transferred.
    - **Black Hat sees:**
        - **Username + password in plaintext.**
        - Contents of the file itself.
        - Directory structure of the FTP server (mapping paths for later exploitation).
6. **TCP Retransmissions**
    
    - Defender sees: “Oh, network congestion.”
    - **Black Hat sees:**
        - Signs of **DoS attacks** (if retransmissions spike abnormally).
        - Potential firewall bottlenecks (hints where security infrastructure lies).

---

## 🌳 **Level 3: Attack Validation**

7. **Spot a Port Scan**
    
    - Defender sees: “Someone scanning us!”
    - **Black Hat sees (their own scan):**
        - Which ports responded (services running).
        - Which are filtered (firewall present).
        - Builds a **map of target services** → “Only SSH and DNS are alive, focus attacks there.”
8. **FTP/Telnet Credentials Sniffing**
    
    - Defender sees: plaintext logins, red flag.
    - **Black Hat sees:**
        - **Instant account access.**
        - Doesn’t need to brute‑force → credentials just handed over.
        - Can reuse stolen credentials on other systems → **pivoting** further inside.
9. **DNS Tunnel Detection**
    
    - Defender sees: long weird DNS queries.
    - **Black Hat sees:**
        - A great covert channel for exfiltration.
        - “I can smuggle my stolen data out through DNS since admins rarely block it.”

---

## 🗻 **Level 4: Command & Control**

10. **Full Web Browsing Session**
    
    - Defender sees: HTTP pages, TLS sessions.
    - **Black Hat sees:**
        - On HTTP: username/passwords, search history, private data.
        - On HTTPS: domains contacted, potential phishing targets.
        - Collects **fingerprints of browsing behavior**.
11. **Reverse Shell Traffic**
    
    - Defender sees: A strange outbound connection to unknown IP:4444.
    - **Black Hat sees:**
        - “Yes, my backdoor worked. I now have an interactive shell.”
        - Can confirm packets are flowing out → access established.
        - Watches commands and responses moving through traffic.
12. **Traceroute Packets**
    
    - Defender sees: Normal TTL expiration.
    - **Black Hat sees:**
        - The **exact path** packets take.
        - Identifies specific firewall/routers along the path → possible choke points.
        - Confirms whether target is hidden behind NAT, VPN, or proxy.

---

# 🎯 **Black Hat Perception Summary**

- **Every packet is recon info**: IPs, ports, services, versions.
- **Every unencrypted protocol is loot**: usernames, passwords, cookies, data.
- **Every handshake tells a story**: what’s alive, what’s filtered, what’s exploitable.
- **Every DNS request leaks secrets**: infrastructure map & covert exfiltration channel.
- **Every TLS certificate handshake is metadata gold**: domain visited, software flaws.

Basically: **Wireshark to a black hat = treasure chest of intel**.

---
