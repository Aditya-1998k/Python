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

---
