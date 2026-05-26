# 📘 **UDP Filter Manual (Wireshark & tcpdump Guide)**

---

## 🟢 1. Basics — Show All UDP

- **Filter:**
    
    text
    
    ```
    udp
    ```
    
- **Meaning:** Show every packet using UDP.
- **Use Case:** Quickly isolate UDP traffic from everything else (ignore TCP, ICMP, ARP, etc.).

---

## 🟡 2. Filter By Port

Like TCP, you can filter by ports because UDP also uses them:

- **Destination or source port:**
    
    text
    
    ```
    udp.port == 53
    ```
    
    → DNS traffic (both queries & replies).
    
- **By specific direction:**
    
    - To server (dst):
        
        text
        
        ```
        udp.dstport == 67
        ```
        
        (DHCP server port)
    - From server (src):
        
        text
        
        ```
        udp.srcport == 68
        ```
        
        (DHCP client port)

### Common UDP services ⬇️

- **53** = DNS
- **67/68** = DHCP (server ↔ client)
- **123** = NTP (time sync)
- **161/162** = SNMP (management)
- **69** = TFTP (trivial file transfer)
- **500** = IPSec (IKE)
- **137/138** = NetBIOS

👉 Hackers often sniff these ports because plaintext creds and configs may ride over them (e.g., TFTP, SNMPv1).

---

## 🟠 3. Data Payload vs Empty

Unlike TCP, UDP has no flags. But you can still filter based on whether a packet **carries data**:

- Non‑empty payload:
    
    text
    
    ```
    udp.length > 8
    ```
    
    (8 bytes = UDP header, so >8 = contains data).
    
- Exactly header only (rare, suspicious):
    
    text
    
    ```
    udp.length == 8
    ```
    

---

## 🔴 4. Protocol Identification (Above UDP)

Most of the time, UDP is just a **carrier** for other application protocols. You can use higher‑level filters:

- DNS analysis:
    
    text
    
    ```
    dns
    ```
    
- DHCP:
    
    text
    
    ```
    bootp
    ```
    
- NTP:
    
    text
    
    ```
    ntp
    ```
    
- SNMP:
    
    text
    
    ```
    snmp
    ```
    

👉 Wireshark decodes these automatically if UDP port is well‑known.

---

## 🟣 5. Attack / Recon Detection with UDP Filters

Unlike TCP scans, UDP scanning is noisier because there’s no handshake. But filters still help:

- Any UDP traffic to “unexpected” port range (possible UDP scan):
    
    text
    
    ```
    udp and !(udp.port == 53 or udp.port == 67 or udp.port == 123)
    ```
    
- Large number of **ICMP Port Unreachable** paired with UDP → classic sign of a UDP scan.

👉 In Wireshark:

text

```
icmp.type == 3 and icmp.code == 3
```

= Destination Unreachable (Port Unreachable), triggered by a bad UDP probe.

---

## ⚫ 6. tcpdump Equivalents

- Capture all UDP traffic:
    
    Bash
    
    ```
    tcpdump udp
    ```
    
- Capture DNS queries:
    
    Bash
    
    ```
    tcpdump udp port 53
    ```
    
- Capture NTP:
    
    Bash
    
    ```
    tcpdump udp port 123
    ```
    
- Capture DHCP:
    
    Bash
    
    ```
    tcpdump 'udp portrange 67-68'
    ```
    

---

# 🎤 Analogy

Think of UDP as **sending postcards**:

- Each one has a source address and a destination (ports).
- No one signs for them (no ACKs, no SYN/FIN).
- You just flood them out and hope they arrive.

So, when sniffing UDP with filters, you’re basically saying:

- _“Show me all postcards (udp).”_
- _“Only postcards to the post office box #53 (DNS).”_
- _“Only postcards that actually have a message longer than the stamp (udp.length > 8).”_

---

# ✨ TL;DR UDP Filter Cheatsheet

- **All UDP:** `udp`
- **Port filtering:** `udp.port == 53` (DNS), `udp.dstport == 68`, etc.
- **Payload present:** `udp.length > 8`
- **Application level:** `dns`, `ntp`, `snmp`, `bootp`
- **UDP scans:** `udp and icmp.type == 3 and icmp.code == 3`

---

⚔️ Here’s a challenge for you (parallel to the TCP handshake one):  
👉 Set Wireshark filter:

text

```
udp.port == 53
```

Then visit any webpage.

- You’ll see your system **sending DNS queries** (UDP packets out).
- Minutes later, responses arrive (UDP packets in).  
    Try to identify **query vs. response**, and check the transaction ID numbers.

---
