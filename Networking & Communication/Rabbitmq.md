# Rabbitmq
1. Rabbitmq is a open source message broker that implements the AMQP (Advance message queuing protocol).
2. It allows applications to communicate by sending messages via queues, decoupling producers (senders) and consumers(recievers)


**AMQP** is a open standard protocol for messaging between two applications. It defines how messages
are formatted, stored, routed, delievered and acknowledged.

Why AMQP?
- Interoperability : Any AMQP-complaint producer can talk to any AMQP broker like Rabbitmq.
- Reliability : Messages are acknowledged, can be persisted to disk, and re-delievered if consumer fails.
- Flexibility : Support different messaging patterns (work queues, pub/sub, routing, rpc etc)


 **AMQP Model Component**
 1. `Producer` : Sends messages
 2. `Exchange` : Route messages based on the rules
    - `Direct` : To specific queues
    - `Fanout` : To all queues
    - `Topic` : Based on pattern (eg: logs.*)
    - `Headers`: Based on message headers
  3. `Queue` : Holds messages untill consumed
  4. `Consumer` : Recieves messages
  5. `Binding` :  Rule connecting exchanges - queue

**Messaging Patterns**
1. `Point-to-Point` : One producer ---> One Queue ----> One consumer
2. `Publish/Subscribe` : One producer ---> Exchange ----> Multiple queues -----> Multiple consumers
3. `Routing` : messages sent to queues based on rules
4. `RPC` : Request/Response over queues

Rabbitmq Remote Procedure Call:

<img width="768" height="107" alt="image" src="https://github.com/user-attachments/assets/0e6486ea-25c5-438c-8e2b-212a7bcf9e3c" />


