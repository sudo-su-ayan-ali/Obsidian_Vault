## **What a socket is**

- A socket is one endpoint of a two-way network connection.
- Address family: AF_INET (IPv4), AF_INET6 (IPv6)
- Types:
    - SOCK_STREAM (TCP): connection-oriented, reliable
    - SOCK_DGRAM (UDP): connectionless, faster, may drop/reorder packets

Typical flow

- Server (TCP): socket() → bind() → listen() → accept() → recv()/send()
- Client (TCP): socket() → connect() → send()/recv()
- UDP: socket() → sendto()/recvfrom() (no connect/accept)

**TCP echo server (single client)**

- Saves: server.py
- Run: python server.py, then connect with the client below


```python
# server.py
import socket

HOST = "127.0.0.1"   # or "0.0.0.0" to accept from any interface
PORT = 65432         # 1024–65535 recommended for non-root

with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
    s.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)  # helpful during dev
    s.bind((HOST, PORT))
    s.listen()  # start listening for incoming connections
    print(f"Listening on {HOST}:{PORT}")
    conn, addr = s.accept()  # blocks until a client connects
    with conn:
        print(f"Connected by {addr}")
        while True:
            data = conn.recv(1024)  # blocks until data or client closes
            if not data:
                break
            conn.sendall(data)  # echo back
```

**TCP client**

- Saves: client.py
- Run while the server listens

```python
# client.py
import socket

HOST = "127.0.0.1"
PORT = 65432

with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
    s.connect((HOST, PORT))
    s.sendall("Hello, server!".encode("utf-8"))  # strings must be bytes
    data = s.recv(1024)
    print("Received:", data.decode("utf-8"))
```

Notes

- encode()/decode(): sockets send bytes, not Python strings.
- sendall() ensures all bytes are sent (send() may send only part).
- recv(1024) reads up to 1024 bytes; may return fewer; empty means the other side closed.
- To handle multiple clients: accept() in a loop and spawn a thread/process per client, or use selectors/asyncio.

UDP echo (very small demo)  
Server:

```python
# udp_server.py
import socket

HOST, PORT = "127.0.0.1", 65433
with socket.socket(socket.AF_INET, socket.SOCK_DGRAM) as s:
    s.bind((HOST, PORT))
    print(f"UDP listening on {HOST}:{PORT}")
    while True:
        data, addr = s.recvfrom(1024)
        print("From", addr, ":", data)
        s.sendto(data, addr)
```

Client:

```python
# udp_client.py
import socket

HOST, PORT = "127.0.0.1", 65433
with socket.socket(socket.AF_INET, socket.SOCK_DGRAM) as s:
    s.sendto(b"ping", (HOST, PORT))
    data, addr = s.recvfrom(1024)
    print("Received:", data, "from", addr)
```

**Common gotchas**

- Address already in use: the port is in TIME_WAIT or another process is bound; use SO_REUSEADDR and/or change port.
- Connection refused: server not running or wrong host/port.
- Blocking by default: operations wait; set timeouts with s.settimeout(seconds).
- Firewalls: local or OS firewall can block inbound connections.