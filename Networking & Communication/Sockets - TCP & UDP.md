# Sockets (TCP/UDP) – Low-level Network Communication
What is socket?
A socket is a virtual endpoints for sending and recieving data
between two programs over a network.
Think of it as a door between your application and the network.
In python, provided by socket module.

Note: Worked at low level as compared to HTTP library  like requests.

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
---
