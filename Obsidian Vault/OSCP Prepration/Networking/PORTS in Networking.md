# 🚪 **Ports in Networking**

**Definition:**  
A **port** is a logical number (ranging from **0 to 65535**) that identifies a specific process or service running on a networked device.

They are used **with IP addresses** to direct traffic to the right application.

- **IP address = the house number**.
- **Port = the specific door into the house.**

Without ports, your computer would get mail dumped in one giant pile at its front porch. Ports keep messages organized by service.

---

## ⚙️ Role of Ports

- Allow **multiple network services** to run on a single IP address simultaneously.  
    Example:
    
    - Port 80 → Web traffic (HTTP)
    - Port 25 → Email (SMTP)
    - Port 22 → Secure remote login (SSH)
- Prevent collisions: your email and your web traffic won’t trip over each other.
    

---

## 🔢 Port Number Ranges

1. **Well-Known Ports (0–1023):**
    
    - Assigned to core services, standardized.
    - Examples:
        - 21 = FTP
        - 22 = SSH
        - 53 = DNS
        - 80 = HTTP
        - 443 = HTTPS
2. **Registered Ports (1024–49151):**
    
    - Used by user applications and vendor software.
    - Example: 3306 = MySQL, 8080 = alternative HTTP.
3. **Dynamic / Private Ports (49152–65535):**
    
    - Picked temporarily by client computers for outbound connections.
    - Example: When your browser connects to a webserver on port 443, your own computer may use port 53721 as its _outgoing_ source port.

---

## 🧩 How Ports Actually Work

Imagine you open your browser and go to `https://example.com`:

1. Your browser asks your operating system: “I need an open port to send my request from.”
2. OS opens a random port (say 49522).
3. Your browser sends data → **from 49522** TO **server port 443** (HTTPS).
4. The server replies from **port 443 → port 49522** back on your machine.
5. When you close the browser, that temporary port is released.

So traffic is always **source port → destination port**.

---

## 🕶️ Hacker’s Perspective

- **Port Scanning:** First step in reconnaissance. Tools like Nmap probe which doors are open. Open ports = potential attack surfaces.
- **Well-Known Weaknesses:** If an outdated service is running on port 21 (FTP), a hacker thinks, “Can I brute-force the login or exploit an old vulnerability?”
- **Service Enumeration:** Identifying what _exactly_ is behind that open door (e.g., is port 22 running OpenSSH 7.4?).

---