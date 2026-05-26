# 📘 **TCP Filter Manual (Wireshark & tcpdump Guide)**

---

## 🟢 1. Basics — “Show me TCP”

- **Filter:**
    
    text
    
    ```
    tcp
    ```
    
- **Meaning:** Displays **all TCP packets**: handshakes, ACKs, retransmissions, data.
- **Use Case:** When you want to see TCP traffic only (no ICMP, ARP, UDP, etc.).

---

## 🟡 2. Filter By Port

- **Filter:**
    
    text
    
    ```
    tcp.port == 80
    ```
    
- **Meaning:** Shows any TCP packet involving port 80 (source or dest).
- **Examples:**
    - `tcp.port == 443` → HTTPS traffic.
    - `tcp.dstport == 22` → Only packets going **to** port 22 (SSH).
    - `tcp.srcport == 25` → Packets **from** SMTP server.

---

## 🟠 3. TCP Flags — Building Blocks

Every TCP header has **flag bits**; you can filter by them:

|Flag|Meaning|Filter|Example Use|
|---|---|---|---|
|SYN|Start connection|`tcp.flags.syn == 1`|Show connection startups.|
|ACK|Acknowledgment|`tcp.flags.ack == 1`|Show all ACK packets.|
|FIN|Finish/session close|`tcp.flags.fin == 1`|Show connections being closed.|
|RST|Reset/abort|`tcp.flags.reset == 1`|Spot abrupt connections.|
|PSH|Push data immediately|`tcp.flags.push == 1`|Useful in analyzing interactivity (Telnet, SSH).|
|URG|Urgent pointer valid|`tcp.flags.urg == 1`|Rarely used but shows prioritization.|

---

## 🔵 4. Handshake Filters (3‑Way Handshake)

### Step 1 — Initial SYN

text

```
tcp.flags.syn == 1 and tcp.flags.ack == 0
```

👉 Shows only **initial SYNs** (client asking to connect).

### Step 2 — SYN‑ACK

text

```
tcp.flags.syn == 1 and tcp.flags.ack == 1
```

👉 Shows server response during handshake.

### Step 3 — Final ACK

text

```
tcp.flags.ack == 1 and tcp.flags.syn == 0 and tcp.len == 0
```

👉 Shows the handshake completion.

🔗 **Combined (all 3):**

text

```
tcp.flags.syn == 1 or (tcp.flags.ack == 1 and tcp.len == 0)
```

👉 Shows all handshake packets together. (But note: this also shows ACKs beyond handshake if not careful).

---

## 🔴 5. Connection Teardown

- **Filter:**
    
    text
    
    ```
    tcp.flags.fin == 1
    ```
    

👉 Shows all packets where connections are being closed politely.

- **Filter (abrupt resets):**
    
    text
    
    ```
    tcp.flags.reset == 1
    ```
    

👉 Shows where connections failed/crashed or were reset.

---

## 🟤 6. Detecting Scans & Attacks

- **Half Open SYN Scan (Nmap style):**
    
    text
    
    ```
    tcp.flags.syn == 1 and tcp.flags.ack == 0
    ```
    

👉 Massive amounts of SYNs with no follow‑up ACKs = port scan or flood.

- **SYN Flood (DoS attack):**
    
    text
    
    ```
    tcp.flags.syn == 1 and tcp.flags.ack == 0
    ```
    

- Count 🤯 thousands per second.

- **Abnormal (Xmas, Null, FIN scans):**
    
    text
    
    ```
    tcp.flags == 0
    ```
    

👉 Null scan (no flags set).  
Or

text

```
tcp.flags.fin == 1 and tcp.flags.psh == 1 and tcp.flags.urg == 1
```

👉 Xmas scan packets.

---

## 🟣 7. Data Transfer vs. Control

- Empty ACKs (just acknowledging, no payload):
    
    text
    
    ```
    tcp.len == 0 and tcp.flags.ack == 1
    ```
    
- Real data transfer (payload present):
    
    text
    
    ```
    tcp.len > 0
    ```
    

---

## ⚫ 8. Filtering Retransmissions & Duplicates

- **TCP Retransmissions:**
    
    text
    
    ```
    tcp.analysis.retransmission
    ```
    
- **Out-of-order packets:**
    
    text
    
    ```
    tcp.analysis.out_of_order
    ```
    
- **Duplicate ACKs:**
    
    text
    
    ```
    tcp.analysis.duplicate_ack
    ```
    

This is how defenders **debug network slowness** (is it an application problem or bad network conditions?).

---

## 🟠 9. Combining Filters

You can combine with `and`, `or`, `not`:

- Capture only HTTP handshakes:
    
    text
    
    ```
    tcp.port == 80 and tcp.flags.syn == 1
    ```
    
- Ignore RSTs:
    
    text
    
    ```
    tcp and not tcp.flags.reset == 1
    ```
    
- Traffic between two hosts:
    
    text
    
    ```
    ip.addr == 192.168.1.10 and ip.addr == 192.168.1.20 and tcp
    ```
    

---

## 🛠️ Bonus — tcpdump Equivalents

Most Wireshark filters carry directly into tcpdump BPF syntax. Example:

- SYN only:
    
    Bash
    
    ```
    tcpdump 'tcp[13] & 2 != 0'
    ```
    
- SYN‑ACK:
    
    Bash
    
    ```
    tcpdump 'tcp[13] & 18 == 18'
    ```
    

(Where byte 13 is the flags field).

---

# 🎤 Analogy

Think of **TCP filters** like having X‑ray glasses:

- Just `tcp` → you see the whole skeleton.
- Add flags → you focus on specific **joints moving** (SYN, ACK, FIN).
- Add ports → you zoom into a specific **organ** (HTTP, SSH, SMTP).

---

# ✨ TL;DR (Cheat Sheet)

- **All TCP:** `tcp`
- **Port filtering:** `tcp.port == 443`
- **Initial SYNs:** `tcp.flags.syn == 1 and tcp.flags.ack == 0`
- **Handshake (all):** `tcp.flags.syn == 1 or (tcp.flags.ack == 1 and tcp.len == 0)`
- **Connection close:** `tcp.flags.fin == 1`
- **Force close:** `tcp.flags.reset == 1`
- **Null scan:** `tcp.flags == 0`
- **Data only:** `tcp.len > 0`

---