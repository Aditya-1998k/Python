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

### Pika
Pika is a official client library for Rabbitmq. It implements 0.9.1 protocol, allowing  python apps to publish, consume and manage
queues/exchanges.

RMQ Details running on local:
```
host: localhost
port: 15672
user_name: guest
pwd: guest
```
Note: No need to mentioned rabbitmq port if you are using default pika port 5672

**publisher.py**
```python
import pika

# create blocking connection and channel
connection = pika.BlockingConnection(pika.ConnectionParameters("localhost"))
channel = connection.channel()

# Declare queues
channel.queue_declare(queue="test")

# Publish a message
channel.basic_publish(exchange="",
                      routing_key = "test",
                      body="Hello Rabbitmq"
                     )
print("Sent Hello Rabbitmq")
connection.close()
```

**consumer.py**
```python
import pika

def on_response(ch, method, properties, body):
    """
    When message recieved then the method
    will trigger with mentioned method parameter
    by rabbitmq
    """
    print(f"Recieved: {body.decode()}")

connection = pika.BlockingConnection(pika.ConnectionParameters("localhost"))
channel = connection.channel()

channel.queue_declare(queue="test")

# Subscribe to queues
channel.basic_consume(queue="test", on_message_callback=on_response, auto_ack=True)

print("Waiting for messages")
channel.start_consuming()
```

**on_response** method have below Parameters. Rabbitmq call this method with those arguments when recieves messages on
subscribed queues.
```bash
(Pdb) ch
<BlockingChannel impl=<Channel number=1 OPEN conn=<SelectConnection OPEN transport=<pika.adapters.utils.io_services_utils._AsyncPlaintextTransport object at 0x7c1688369210> params=<ConnectionParameters host=localhost port=5672 virtual_host=/ ssl=False>>>>
(Pdb) method
<Basic.Deliver(['consumer_tag=ctag1.2705851096264b82862b2da52ffc2e0e', 'delivery_tag=1', 'exchange=', 'redelivered=False', 'routing_key=test'])>
(Pdb) properties
<BasicProperties>
(Pdb) properties.__dict__
{'content_type': None, 'content_encoding': None, 'headers': None, 'delivery_mode': None, 'priority': None, 'correlation_id': None, 'reply_to': None, 'expiration': None, 'message_id': None, 'timestamp': None, 'type': None, 'user_id': None, 'app_id': None, 'cluster_id': None}
(Pdb) body
b'Hello Rabbitmq'
```

### RPC with Rabbitmq

<img width="768" height="107" alt="image" src="https://github.com/user-attachments/assets/0e6486ea-25c5-438c-8e2b-212a7bcf9e3c" />

How RPC works:
1. When the Client starts up, it creates an exclusive callback queue.
2. For an RPC request, the Client sends a message with two properties: reply_to, 
which is set to the callback queue and correlation_id, which is set to a unique value for every request.
3. The request is sent to an rpc_queue queue.
4. The RPC worker (aka: server) is waiting for requests on that queue. When a request appears, 
it does the job and sends a message with the result back to the Client, using the queue from the reply_to field.
5. The client waits for data on the callback queue. When a message appears, it checks the correlation_id property. If it matches the value from the request it returns the response to the application.

**rpc_client.py**
```python
import pika
import uuid

def publish_to_rabbitmq_rpc():
    # Creating connection and channel
    connection = pika.BlockingConnection(pika.ConnectionParameters("localhost"))
    channel = connection.channel()

    # Define callback queue
    result = channel.queue_declare(queue="", exclusive=True)
    callback_queue = result.method.queue

    response = None
    corr_id = str(uuid.uuid4())

    def on_response(ch, method, props, body):
         # use response defined in outer function
         nonlocal response
         if props.correlation_id == corr_id:
             response = body.decode()

    channel.basic_consume(queue=callback_queue, on_message_callback=on_response, auto_ack=True)

    # Send messages
    message = "World"
    channel.basic_publish(
        exchange='',
        routing_key='rpc_queue',
        properties=pika.BasicProperties(
            reply_to=callback_queue,
            correlation_id=corr_id,
        ),
        body=message
    )

    print("Message Sent to worker, waiting for response")

    while response is None:
        connection.process_data_events()

    print(f"Got Response : {response}")

publish_to_rabbitmq_rpc()
```

**rpc_server.py (worker)**
```python
import pika

def on_response(ch, method, props, body):
    message = body.decode()
    print(f" [.] Received request: {message}")

    # Echo back the message
    response = f"Hello {message} from worker"
    if props.reply_to:
        ch.basic_publish(
            exchange='',
            routing_key=props.reply_to,
            properties=pika.BasicProperties(
                correlation_id=props.correlation_id
            ),
            body=response
        )
    ch.basic_ack(delivery_tag=method.delivery_tag)

# Creating connection and channel
connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connection.channel()

# Declaring rpc_queue
channel.queue_declare(queue='rpc_queue')

#prefetch count = 1 (for event distribution)
channel.basic_qos(prefetch_count=1)
channel.basic_consume(queue='rpc_queue', on_message_callback=on_response)

print("Awaiting RPC requests")
channel.start_consuming()
```

Output (cleint.py)
```
Message Sent to worker, waiting for response
Got Response : Hello World from worker
```

Output (server.py)
```
Awaiting RPC requests
 [.] Received request: World
 [.] Received request: World
```

**connection.process_data_events()**

Normally, when you call channel.start_consuming(), pika enters its own internal event loop and continuously processes events.
But in your RPC client code, you don’t want to block forever in start_consuming(). Instead, you just want to:
1. Send a request
2. Wait until the response comes back

That’s where connection.process_data_events() comes in:
It tells pika:

“Process any pending network events (incoming messages, connection updates, heartbeats) for me right now.”
So each time you call it, pika checks if RabbitMQ has sent something — if the server (worker) replied, the callback (on_response) gets triggered.

Note: process_data_events() also accepts a timeout (in seconds)
```python
connection.process_data_events(time_limit=1)
```

Some keyword you can visit:
1. heartbeat
2. redelievered messages
3. consumer_timeout
