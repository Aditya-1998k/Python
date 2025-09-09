# ZeroMQ / PyZMQ – High-Performance Messaging Library

#### ZeroMQ
1. The core messaging library.
2. Written in C++.
3. Provides the fast, brokerless messaging engine.

#### PyZMQ
1. The Python bindings (wrapper) for ZeroMQ.
2. Lets you use ZeroMQ in Python.
3. Provides Pythonic APIs for sockets, contexts, and messaging patterns.

Installation: `pip install pyzmq`

Conclusion:
- `ZeroMQ` = the actual technology / engine.
- `PyZMQ` = Python library that lets you use ZeroMQ.



PyZMQ is a light weight library (No broker required). Faster than RabbitMQ for peer-to-peer or distributed system.
But not as reliable as Rabbitmq (So, Not replacement of rabbitmq).

We can use PyZMQ, where:
1. Low-latency systems (HPC, trading, simulations).
2. Distributed systems where you don’t want a broker.
3. Flexible messaging patterns beyond TCP/UDP.

Provide advance socket type:
1. `REQ/REP` = Request/Reply
2. `PUB/SUB` = Publish/Subscribe
3. `PUSH/PULL` = Pipeline
4. `DEALER/ROUTER` = Advance Routing

---

## 1. REQUEST/REPLY Socket
**server.py**
```python
import zmq

context = zmq.Context()
socket = context.socket(zmq.REP)   # Reply socket
socket.bind("tcp://*:5555")

while True:
    msg = socket.recv().decode()
    print("Recieved:", msg)
    socket.send_string(f"Echo: {msg}")
```

**client.py**
```python
import zmq

context = zmq.Context()
socket = context.socket(zmq.REQ)
socket.connect("tcp://localhost:5555")

socket.send_string("Hello ZeroMQ")
print("Server says: ", socket.recv().decode())
```

Output (server.py):
```
Recieved: Hello ZeroMQ
```
Output (client.py)
```
Server says:  Echo: Hello ZeroMQ
```
---

## 2. Pub/Sub
Suppose You have server which is publishing news every 2 second to the topic called "news"
The subscriber which subsribed to the topic "news" will get the messages.

**server.py**
```python
import time
import zmq

context = zmq.Context()
socket = context.socket(zmq.PUB)
socket.bind("tcp://*:5556")

while True:
    topic = "news"
    message = f"Breaking news at time {time.time}"
    socket.send_string(f"{topic} {message}")
    print("Published:", message)
    time.sleep(2)
```
**client.py**
```python
import zmq

context = zmq.Context()
socket = context.socket(zmq.SUB)
socket.connect("tcp://localhost:5556")

# Subscribe to Topic "news"
socket.setsockopt_string(zmq.SUBSCRIBE, "news")

while True:
    message = socket.recv_string()
    print("Recieved: ", message)
```
Output (server.py):
```
Published: Breaking news at time 1757385573.4737158
Published: Breaking news at time 1757385575.4761477
Published: Breaking news at time 1757385577.4784865
Published: Breaking news at time 1757385579.4808614
Published: Breaking news at time 1757385581.4827735
```

Output (client.py):
```
Recieved:  news Breaking news at time 1757385573.4737158
Recieved:  news Breaking news at time 1757385575.4761477
Recieved:  news Breaking news at time 1757385577.4784865
Recieved:  news Breaking news at time 1757385579.4808614
Recieved:  news Breaking news at time 1757385581.4827735
```

## 3. PUSH/PULL (Pipeline Pattern)
This is work like a queue:
1. Push socket: send tasks
2. Pull socket: recieve task and load balance automatically

**producer.py**
```python
import zmq
import time

context = zmq.Context()
socket = context.socket(zmq.PUSH)
socket.bind("tcp://127.0.0.1:5555")

for i in range(5):
    msg = f"Task {i}"
    print("Sending:", msg)
    socket.send_string(msg)
    time.sleep(1)
```

**worker.py**
```python
import zmq

context = zmq.Context()
socket = context.socket(zmq.PULL)
socket.connect("tcp://127.0.0.1:5555")

while True:
    msg = socket.recv_string()
    print("Recieved:", msg)
```

Run python worker.py in multiple terminals, then run python producer.py.
Tasks will be **distributed across workers**.

Output(producer)
```
Sending: Task 0
Sending: Task 1
Sending: Task 2
Sending: Task 3
Sending: Task 4
```

Output (worker1)
```
Recieved: Task 0
Recieved: Task 1
Recieved: Task 3
```

Output(worker2)
```
Recieved: Task 2
Recieved: Task 4
```
