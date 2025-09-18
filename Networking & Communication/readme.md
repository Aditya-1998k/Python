# Networking & Communication in Python  

This section covers core networking and communication concepts using Python. It includes traditional socket programming, modern communication frameworks, message brokers, and RPC systems.  

## Topics Covered  
- **Sockets (TCP & UDP):** Low-level communication between processes and systems.  
- **WebSockets:** Real-time, bidirectional communication over a single TCP connection.  
- **HTTP Clients:** Libraries like `requests` and `httpx` for making API calls.  
- **gRPC:** High-performance RPC framework using Protocol Buffers.  
- **XML-RPC:** Remote procedure calls over XML and HTTP.  
- **PyZMQ (ZeroMQ):** High-performance asynchronous messaging library.  
- **RabbitMQ:** A message broker for reliable message delivery and pub-sub patterns.  

---

## 📌 Questions  

### Sockets (TCP & UDP)  
1. What’s the difference between TCP and UDP, and when would you use each in Python?  
2. How would you implement a simple TCP client and server in Python?  
3. How does `socket.sendall()` differ from `socket.send()`?  
4. What are common issues you might face with blocking sockets, and how do you avoid them?  

### WebSockets  
5. How are WebSockets different from traditional HTTP requests?  
6. Can you explain how to implement a WebSocket server in Python using `websockets` library?  
7. What are some real-world use cases where WebSockets are preferred?  

### HTTP Clients  
8. Compare `requests` vs. `httpx`. Which would you use for async APIs and why?  
9. How would you handle retries, timeouts, and connection pooling in `requests`?  

### gRPC  
10. What advantages does gRPC have over REST APIs?  
11. How does gRPC use Protocol Buffers, and why are they efficient?  
12. Can you explain how to generate Python gRPC client and server code from `.proto` files?  

### Messaging (RabbitMQ & PyZMQ)  
13. What is the difference between RabbitMQ and ZeroMQ (PyZMQ) in terms of use cases?  
14. How would you implement a publisher-subscriber model using RabbitMQ in Python?  

---
