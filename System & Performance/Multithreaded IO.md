# Multithreaded I/O
Traditional blocking I/O - if you `recv()` on your socket, your program wait untill data arrives.
- For server handling many connections (chat apps, proxies, http), this waste resources
- Instead of this we can use `I/O multiplexing` - Here server connected with multiple sockets
- But it will not wait for data to arrive on one socket, instead of that it will
- Recieve where data is ready

**Blocking I/O Server**
```
Time →  ---------------------------------------------------->

Client A:   ---- send "Hi" ---- wait ---- send "Bye" ----
Client B:                                            ---- send "Ping" ----
Client C:                                                             ---- send "Yo" ----

Server:     accept A -----> recv from A -----> [BLOCKED waiting for A]
                    (ignores B, C until A finishes!)
```
Server is stuck on Client A. Clients B and C wait, even if they already have data ready.

**Multiplexed Server**
```
Time →  ---------------------------------------------------->

Client A:   ---- send "Hi" ---- wait ---- send "Bye" ----
Client B:                                ---- send "Ping" ----
Client C:                                                   ---- send "Yo" ----

Server:     [select()] → sees A ready → recv "Hi" from A
            [select()] → sees B ready → recv "Ping" from B
            [select()] → sees A ready → recv "Bye" from A
            [select()] → sees C ready → recv "Yo" from C
```
Server quickly jumps between ready clients. Nobody blocks others.

 ---
 
## Tools in Python
Python `selectors` module in `stdlib` is a high-level abstraction over different `OS-Specific` mechanism.
- `select()` → oldest, works everywhere, but inefficient with many sockets.
- `poll()` → Linux, better scalability.
- `epoll()` → Linux, very scalable, edge/level triggered.
- `kqueue()` → BSD/MacOS, efficient event notification.

**Best Way**: You write portable code with selectors, and Python internally picks the best available one.

Example: Traditional Blocking I/O vs Non-blocking I/O with selectors

**Traditional Blocking I/O**
```python
import socket

def blocking_server():
    sock = socket.socket()
    sock.bind(("localhost", 9000))
    sock.listen()
    print("Server (blocking) on port 9000...")

    while True:
        conn, addr = sock.accept()   # wait for client
        print("Client connected:", addr)

        while True:
            data = conn.recv(1024)   # 🚨 blocks until client sends something
            if not data:
                print("Client disconnected")
                break
            conn.sendall(data)

blocking_server()
```
Problem: 
- While this server is waiting for `recv()`, it cannot serve any other clients.
- To handle multiple clients, you’d need threads or processes, which is `heavy`.

**Non-blocking I/O with selectors**
```python
import selectors
import socket

# It will select the best available selector (epoll/kqueue/select, depending on OS)
sel = selectors.DefaultSelector()

def accept(sock):
    conn, addr = sock.accept()
    print("Client connected:", addr)
    conn.setblocking(False)
    # Register this client socket for READ events
    sel.register(conn, selectors.EVENT_READ, read)

def read(conn):
    data = conn.recv(1024)
    if data:
        # Echo back the received data
        conn.sendall(data)
    else:
        print("Client disconnected")
        # Unregister and close if client closes connection
        sel.unregister(conn)
        conn.close()

def multiplex_server():
    sock = socket.socket()
    sock.bind(("localhost", 9001))
    sock.listen()
    sock.setblocking(False)  # Non-blocking socket (important for multiplexing)

    # Register server socket to watch for "incoming connections"
    sel.register(sock, selectors.EVENT_READ, accept)
    print("Server (multiplex) running on port 9001...")

    # Event loop: blocks until *any* registered socket is ready
    while True:
        events = sel.select()   # Wait for sockets ready to read/write
        for key, _ in events:
            callback = key.data   # Either `accept` or `read`
            callback(key.fileobj) # Call the right handler

multiplex_server()
```

1. **Selector choice**  
   `selectors.DefaultSelector()` automatically picks the most efficient I/O multiplexing method available (`epoll` on Linux, `kqueue` on BSD/macOS, or `select` as fallback).

2. **Server socket setup**  
   A TCP socket is created, bound to `("localhost", 9001)`, and put into `listen()` mode to accept incoming client connections.

3. **Non-blocking mode**  
   The server socket is set to `setblocking(False)`, so `accept()` or `recv()` never block; instead, readiness is detected by the selector.

4. **Register server socket**  
   The server socket is registered with the selector for `EVENT_READ`, meaning the selector will notify us when a new client tries to connect.

5. **Accept callback**  
   When a connection is ready, the `accept()` function is called. It accepts the client socket, sets it non-blocking, and registers it with the selector for `EVENT_READ`.

6. **Register client socket**  
   Each new client connection is tracked by the selector with a callback to the `read()` function.

7. **Read callback**  
   When a client socket is ready to read, the `read()` function is called. It uses `recv()` to read data without blocking.

8. **Echo response**  
   If data is received, the server immediately sends it back using `sendall()` (simple echo server behavior).

9. **Client disconnect handling**  
   If `recv()` returns empty data, it means the client closed the connection. The socket is unregistered from the selector and closed.

10. **Main event loop**  
    The `while True:` loop calls `sel.select()`, which blocks until at least one socket (server or client) is ready. Then it dispatches the appropriate callback (`accept` for new connections, `read` for client data).
