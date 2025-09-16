## Know Thy Packets: Demystifying Network Fundamentals with Scapy

Modern tools (requests, browsers, frameworks) hide network details.
But debugging, performance tuning, security, and distributed systems need packet-level understanding.
Scapy lets us see and manipulate packets directly.

OSI Model:
```python
+--------------------------------------------------+
| 7. Application   (HTTP, DNS, TLS, SMTP, etc.)    |
+--------------------------------------------------+
| 6. Presentation  (TLS encryption, compression)   |
+--------------------------------------------------+
| 5. Session       (Managing conversations)        |
+--------------------------------------------------+
| 4. Transport     (TCP, UDP)                      |
+--------------------------------------------------+
| 3. Network       (IP addressing, routing)        |
+--------------------------------------------------+
| 2. Data Link     (Ethernet, Wi-Fi frames)        |
+--------------------------------------------------+
| 1. Physical      (Cables, radio waves, fiber)    |
+--------------------------------------------------+
```

When we run `https://www.example.com` in browser:
1. DNS Lookup → Convert www.example.com → 93.184.216.34
2. TCP Handshake → Establish connection with server (SYN, SYN-ACK, ACK)
3. TLS Handshake → Negotiate encryption keys
4. HTTP GET Request → Ask for /index.html
5. HTTP Response → Server sends back data
6. Connection Teardown → TCP FIN → ACK → FIN → ACK

### 1. DNS Lookup
Goal: Convert a human-readable domain name to an IP address that computers understand.
Browser sees www.example.com and asks the DNS (Domain Name System) server:
1. “Hey, what is the IP address for www.example.com?”
2. The DNS server responds with the IP: e.g., 93.184.216.34.
3. Without this, your computer doesn’t know where to send packets.
Key Protocols: DNS, UDP

### 2. TCP Handshake

Goal: Establish a reliable connection between your computer and the server.
TCP is connection-oriented, so before sending data, both sides agree on communication parameters.

Steps (3-way handshake):
```
Client -> Server : SYN       (I want to start a connection)
Server -> Client : SYN-ACK   (Acknowledges your request, ready to connect)
Client -> Server : ACK       (Acknowledges server, connection established)
```
Key Protocol: TCP (Transport Layer)

Conceptually: Now both sides are ready to send/receive data reliably.

### 3. TLS Handshake
Goal: Secure the connection with encryption.
Since the URL is `https://`, browser uses TLS (Transport Layer Security):
1. ClientHello: Browser proposes encryption algorithms.
2. ServerHello + Certificate: Server picks algorithm, sends its certificate.
3. Key Exchange: Both sides compute session keys.
4. Finished: Browser and server confirm encryption setup.
5. Result: All further communication is encrypted.
Key Protocol: TLS (over TCP)

### 4. HTTP GET Request
Goal: Ask the server to send the resource (e.g., /index.html).
Browser sends a request like:
```
GET /index.html HTTP/1.1
Host: www.example.com
User-Agent: Mozilla/5.0
```
The request goes over TCP (encrypted if HTTPS).

Key Protocol: HTTP (Application Layer)

### 5. HTTP Response
Goal: Server sends back the requested resource.

Example response headers + body:
```
HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 1256

<html> ... </html>
```
Browser receives data, renders the webpage.
Key Protocols: HTTP (encrypted if HTTPS)

### 6. Connection Teardown
Goal: Gracefully close the TCP connection.
TCP uses a 4-step handshake:
```
Client -> Server : FIN       (I’m done sending)
Server -> Client : ACK       (Acknowledges)
Server -> Client : FIN       (Server done)
Client -> Server : ACK       (Acknowledges)
```
Ensures all remaining packets are delivered before closing the connection.

Key Protocol: TCP

## requests.get() Under the Hood
```python
import requests
response = requests.get("https://www.example.com")
print(response.text[:200])
```
Python and your computer do a lot of things automatically. Here’s the step-by-step flow:

##### 1. URL Parsing

requests parses the URL: "https://www.example.com"
```
Scheme → https → use TLS
Host → www.example.com
Port → default 443 (because HTTPS)
Path → /
```

##### 2. DNS Lookup
Converts domain name → IP address (just like Scapy example).
Usually done via OS resolver, not manually in Python.
```
Result: 93.184.216.34 (example IP).
www.example.com -> 93.184.216.34
```

##### 3. TCP Handshake
requests creates a TCP socket and connects to the server.
The OS performs a 3-way handshake automatically:
```
Client → Server : SYN
Server → Client : SYN-ACK
Client → Server : ACK
```
After this, the TCP connection is established.

##### 4. TLS Handshake (if HTTPS)
Since it’s HTTPS, Python wraps the socket with ssl module.
Steps:
```
ClientHello → propose TLS version & cipher suites
ServerHello + Certificate → pick cipher, verify certificate
Key Exchange → establish session keys
Finished → encryption active
From here on, all data is encrypted.
```
#### 5. HTTP GET Request
requests builds an HTTP request:
```
GET / HTTP/1.1
Host: www.example.com
User-Agent: python-requests/2.32
Accept-Encoding: gzip, deflate
Connection: keep-alive
```
Sends it over the TCP connection (encrypted if HTTPS).
OS ensures packets are fragmented, delivered, retransmitted if necessary.

##### 6. HTTP Response
Server responds:
```
HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 1256
...
<html> ... </html>
```
requests library automatically parses headers and body, decompresses if gzip, and returns a Response object.

##### 7. TCP Connection Teardown
After reading the response, requests may:
Keep connection alive (default) for reuse
Or close it `(FIN → ACK → FIN → ACK)`

##### 8. Python Layer
Everything above is abstracted into requests and urllib3:
```
Socket creation → TCP connection
SSL wrap → TLS handshake
HTTP message encoding/decoding
Chunked transfer handling, redirects, errors
```
So from Python code it looks simple:
```
response = requests.get("https://www.example.com")
```
But under the hood, it’s doing DNS → TCP → TLS → HTTP → Parse → Return.

**ASCII Diagram**
```bash
Browser/Python requests
     │
     │ DNS Lookup (www.example.com -> 93.184.216.34)
     ▼
  TCP Handshake
 SYN → SYN-ACK → ACK
     │
     │ TLS Handshake (HTTPS)
     │  ClientHello <-> ServerHello/Cert <-> Key Exchange
     ▼
  HTTP GET Request
 GET / HTTP/1.1
     │
     ▼
  HTTP Response
  200 OK + HTML
     │
     ▼
  TCP Connection Teardown (FIN/ACK)
```

### Using Scapy in Python
```
pip install scapy
```
Note: Optional for network sniffing on Windows/Linux: you might need admin/root privileges.

```python
from scapy.all import *
```
all imports all the core modules: IP, TCP, UDP, ICMP, sr1, send, sniff, etc

