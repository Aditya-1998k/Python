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


