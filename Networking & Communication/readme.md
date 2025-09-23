# 🌐 Networking & Communication in Python

Networking and communication are at the core of **distributed systems**, **microservices**, and **real-time applications**.  
This section explores Python’s libraries and frameworks for **sending, receiving, and managing data across networks**.

---

## 📂 Topics Covered

1. **Sockets (TCP/UDP)** – Low-level networking with `socket` library.
2. **HTTP Clients** – Communicating with REST APIs using `requests` and `httpx`.
3. **WebSockets** – Real-time bidirectional communication (`websockets`, `aiohttp`).
4. **gRPC** – High-performance RPC framework with Protocol Buffers.
5. **RabbitMQ (AMQP)** – Message broker for async communication and decoupled services.
6. **XML-RPC** – Remote Procedure Calls using XML over HTTP.
7. **PyZMQ (ZeroMQ)** – Fast messaging library for distributed systems.

---

## 🧑‍💻 Why Learn Networking & Communication?

- Build **client-server applications** (e.g., chat apps, multiplayer games).
- Work with **REST APIs** and **microservices**.
- Enable **real-time communication** (trading apps, dashboards, notifications).
- Use **message brokers** for **scalable, event-driven architectures**.
- Implement **cross-language communication** with gRPC/ZeroMQ.

---

## ❓ Common Questions

### 🔹 Sockets
1. Difference between TCP and UDP sockets?
2. How does the `socket` module create a simple client-server?
3. What is the three-way handshake in TCP?

### 🔹 HTTP Clients
4. Difference between `requests` and `httpx` libraries?
5. How do you handle retries, timeouts, and sessions in HTTP?
6. Explain idempotency in HTTP methods.

### 🔹 WebSockets
7. What problem do WebSockets solve compared to HTTP polling?
8. How do you implement a basic WebSocket server in Python?
9. Difference between WebSocket and gRPC streaming?

### 🔹 gRPC
10. What are Protocol Buffers, and why are they used in gRPC?
11. How is gRPC different from REST APIs?
12. Explain unary, server-streaming, client-streaming, and bidirectional streaming in gRPC.

### 🔹 RabbitMQ (AMQP)
13. What is a message broker, and why use RabbitMQ?
14. Difference between queue, exchange, and binding in AMQP.
15. How do you ensure message durability and acknowledgments?

### 🔹 XML-RPC
16. What is XML-RPC, and how does it differ from REST and SOAP?
17. How do you call a remote function using Python’s `xmlrpc.client`?
18. Why is XML-RPC less common today?

### 🔹 PyZMQ (ZeroMQ)
19. What is ZeroMQ, and how is it different from RabbitMQ?
20. Explain ZeroMQ messaging patterns: PUB/SUB, REQ/REP, PUSH/PULL.
21. Use cases where ZeroMQ is preferred over HTTP or gRPC.

---

## 📖 Next Steps

- Build a **chat application** using sockets or WebSockets.
- Create a **microservice** communicating with **gRPC**.
- Implement a **task queue** with RabbitMQ.
- Experiment with **ZeroMQ PUB/SUB** for real-time updates.

---

✨ Mastering networking & communication enables you to design **scalable, distributed, and fault-tolerant systems**.
