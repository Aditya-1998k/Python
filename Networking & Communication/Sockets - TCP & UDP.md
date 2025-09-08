# Sockets (TCP/UDP) – Low-level Network Communication
What is socket?
A socket is a virtual endpoints for sending and recieving data
between two programs over a network.
Think of it as a door between your application and the network.
In python, provided by socket module.

Note: Worked at low level as compared to HTTP library  like requests.

**Why TCP or UDP called low level Network communication?**

TCP and UDP are called low-level networking protocols because:

Networking has layers (OSI model):
```
7. Application   → HTTP, FTP, DNS
6. Presentation  → encryption, compression
5. Session       → managing sessions
4. Transport     → TCP, UDP   ✅
3. Network       → IP
2. Data Link     → Ethernet, WiFi
1. Physical      → cables, radio waves
```
TCP & UDP live at the Transport Layer (Layer 4). They are just above IP and deal with raw data 
delivery between two endpoints. They don’t understand high-level things like web pages or REST APIs.

**Why “Low-Level”?**
They work with raw bytes (no JSON, no HTML, no HTTP headers).
Programmer must handle:
1. Message encoding/decoding.
2. Splitting/reassembling messages (especially with TCP streams).
3. Reliability (for UDP).
By contrast, high-level protocols like HTTP, gRPC, MQTT build on top of TCP/UDP and provide more abstraction.

---

## TCP 
Transmission control protocol is a communication protocol that:
1. Ensures reliable, ordered and error-checked delievery of data
2. Connection oriented - Both client and server first establish the connection before exchanging the data
3. Used in Web browsing (HTTP/HTTPS), Email (SMTP/IMAP), FTP (File transfer protocol)

Basic Example (Server which will listen for client):

**server.py**
```python
import socket

server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server_socket.bind("localhost", 12345)
server_socket.listen()

print("Server is listening on port: 12345)

conn, addr = server_socket.accept()
print("Connected by ", addr)

data = conn.recv(1024).decode()
print("Recieved: ", data)

conn.sendall(f"Echo: {data}.".encode())
conn.close()
```

**client.py**
```python
import socket

client_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
client_socket.connect(("localhost", 12345))  # <-- connect (not bind)

client_socket.sendall(b"Hello TCP Server")
response = client_socket.recv(1024).decode()
print("Server says:", response)

client_socket.close()
```
1. First run the server.py file
```
python server.py
Server is listening on port 12345...
```
2. Run client.py on other terminal
```
python client.py
Server says: Echo: Hello TCP Server
```

**ASCII FLOW**
```
Client                         Server
------                         ------
socket()                       socket()
connect()   ----------------->  bind() + listen()
send("Hello") --------------->  accept() + recv()
recv("Echo: Hello") <--------  send("Echo: Hello")
close()                        close()
```
Output on server.py terminal
```
Connected by ('127.0.0.1', 47794)
Received: Hello TCP Server
```

Expalaination:

```
server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

socket.AF_INET → means we’re using IPv4 addresses (localhost, 127.0.0.1, etc.).
socket.SOCK_STREAM → means we’re using TCP protocol (reliable, connection-oriented).
```

```
server_socket.bind(("localhost", 12345))

Attach the socket to IP + port:
"localhost" = only accept connections from your own computer.
12345 = the port number (must be free).
If another process is already using this port, you’ll see:
OSError: [Errno 98] Address already in use.
```

```
server_socket.listen()

Put the socket into listening mode, meaning it will wait for clients to connect.
By default, it allows a backlog (queue) of 5 pending connections.
```

```
conn, addr = server_socket.accept()

Blocking call → The program pauses here until a client connects.
conn = a new socket object for communicating with that client.
addr = client’s address (('127.0.0.1', some_random_port)).
So now your server can talk only with that client.
```

```
data = conn.recv(1024).decode()

Receive up to 1024 bytes of data from client.
.recv() returns bytes → .decode() converts it to string.
```

```
conn.sendall(f"Echo: {data}".encode())

Send data back to the client.
.encode() converts string → bytes.
.sendall() ensures all data is sent (TCP guarantees reliability).
This makes your server behave like an echo server.
```


```
+------------------+                    +------------------+
|   Client App     |                    |   Server App     |
|  (Python code)   |                    |  (Python code)   |
+------------------+                    +------------------+
         |                                       |
         |  socket()                             |  socket()
         |-------------------------------------->|
         |           connect()                   |
         |-------------------------------------->|
         |                                       |  bind(), listen()
         |                                       |  accept()
         |<--------------------------------------|
         |         TCP Connection Established     |
         |                                       |
         |  send("Hello")                        |
         |-------------------------------------->|
         |                                       |  recv("Hello")
         |                                       |
         |                Data Flow              |
         |                                       |
         |  close()                              |  close()
         |-------------------------------------->|
```

---

## UDP (User Datagram Protocol)
UDP is called connectionless protocol.
Here:
1. No Handshake
2. No gaurantee of delievery
3. No order
4. But **very fast**

Useful for:
1. Streaming audio/video
2. Online Gaming
3. DNS queries

Note: A DNS query is a request from a user's device to a Domain Name System (DNS) server to 
find the numerical IP address associated with a specific domain name.

**Comparison with TCP**
```
-------------------------------------------------------------
     TCP:              |           UDP:
-------------------------------------------------------------
 Client ↔ Server       |      Client → Server
(Handshake, reliable)  |    (No connection, fire & forget)
```

Basic Example:

**server.py**
```python
import socket

server_socket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
server_socket.bind(("localhost", 12345))

print("UDP server is listening on port : 12345")

while True:
         data, addr = server_socket.recvfrom(1024)     # Recieve packet + client address
         print(f"Recieved from {addr}: {data.decode()}")
         server_socket.sendto(f"Recieved you Message: {data.decode()}".encode(), addr)
```

**client.py**

```python
import socket

client_socket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

client_socket.sendto(b"Hello UDP server", ("localhost", 12345))
data, addr = client_socket.recvfrom(1024)

print("server says: ", data.decode())

client_socket.close()
```

1. Start the server first
2. Start the client

Output (server.py)
```
UDP server is listening on port : 12345
Recieved from ('127.0.0.1', 52604): Hello UDP server
```

Output(client.py)
```
server says:  Recieved you Message: Hello UDP server
```

Explaination:
```python
server_socket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

AF_INET → IPv4
SOCK_DGRAM → means UDP socket (datagram-based, connectionless).
If this were SOCK_STREAM, it would be TCP.
```

```python
server_socket.bind(("localhost", 12345))

Attach the socket to IP & port.
"localhost" = local machine only.
12345 = port number.
Now the OS knows: “Anything sent to 127.0.0.1:12345 via UDP goes to this socket.”
```

```python
print("UDP Server is listening on port 12345...")

Note: Just a message → But ⚠️ technically, UDP doesn’t “listen” (no connections).
```

**Why no listen() in UDP?**

Because UDP is connectionless:
In TCP:
1. You must listen() → wait for a client → accept() → establish a connection.
2. Communication happens on that connection (reliable stream).

In UDP:
1. No connection exists.
2. Clients just fire off datagrams to the server’s IP:port.
3. Server just receives them → no handshake, no accept(), no listen().

That’s why your server goes straight to recvfrom() without listen().

- TCP = Phone call 📞 (connection established, then talk).
- UDP = Postcard 📮 (send message directly, no setup, may be lost).
