# ZeroMQ / PyZMQ – High-Performance Messaging Library

#### ZeroMQ
1. The core messaging library.
2. Written in C++.
3. Provides the fast, brokerless messaging engine.

#### PyZMQ
1. The Python bindings (wrapper) for ZeroMQ.
2. Lets you use ZeroMQ in Python.
3. Provides Pythonic APIs for sockets, contexts, and messaging patterns.

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

### REQUEST/REPLY Socket





