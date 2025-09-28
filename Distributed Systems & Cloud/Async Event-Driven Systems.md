## Async Event-Driven Systems

### asyncio
`asyncio` is a python's built-in library for asynchronous programming.
Instead of waiting for IO (network, DB, File Read) to complete, it allows python to switch
tasks efficiently.

```python
import asyncio

async def say_hello():
    print("Hello...")
    await asyncio.sleep(2)   # Non-blocking wait
    print("...World!")

async def main():
    await asyncio.gather(say_hello(), say_hello())

asyncio.run(main())
```
What happens:

1. Both say_hello() coroutines run concurrently.
2. Total time = ~2s (not 4s), because they don’t block each other.

### Async + Message Broker
In distributed systems, you always want few techniques:
1. Produce messages - (API recieves requests, pushes to queue etc)
2. Consume Message - (worker processes messages from the queue)

with `asyncio`, we can integrate with brokers like:
- Rabbitmq (via aio-pika)
- Kafka (via aiokafka)
- Redis (via aioredis)

Note: aio-pika version is built on top of pika (asynio version)

1. Asyncio-based → non-blocking
2. Designed for event-driven microservices where you want concurrency without threads.
3. Uses `await / async` with.

Run rabbitmq:
```shell
docker run -it --rm -p 5672:5672 -p 15672:15672 rabbitmq:3-management
```

**sends.py**
```python
import asyncio
import aio_pika

async def main():
    connection = await aio_pika.connect_robust("amqp://guest:guest@localhost/")
    channel = await connection.channel()

    queue = await channel.declare_queue("my_queue", durable=True)

    # Publish messages
    await channel.default_exchange.publish(
        aio_pika.Message(body=b"Hello Async World!"),
        routing_key=queue.name
    )
    print("Message Sent")

    await connection.close()

asyncio.run(main())
```

**consumer.py**
```python
import asyncio
import aio_pika

async def main():
    connection = await aio_pika.connect_robust("amqp://guest:guest@localhost/")
    channel = await connection.channel()

    queue = await channel.declare_queue("my_queue", durable=True)

    async with queue.iterator() as queue_iter:
        async for message in queue_iter:
            async with message.process():
                print(f"Recieved: {message.body.decode()}")

asyncio.run(main())
```

Output:
```
🕒 13:18 ➜  python consume.py 
Recieved: Hello Async World!
```

### Why to use Asyncio + aio-pika
In traditional Distributed system. Each worker usually seperate process.
Each worker blocks while waiting for messages, but does not block the main
application because:
- Each worker is independent
- The process model allows concurrency without `asyncio`

Benefit of asyncio is minimal here, because each worker is already `parallel` due to multiple 
processes.
So here using blocking pika is fine, `aio-pika` is not necessary.

But when beneficial:

If your service is both producing and consuming messages AND also doing other tasks in the same process (e.g., an API server), async is valuable.
Example:
1. FastAPI service receives `HTTP requests ---> publishes to RabbitMQ ----> also needs to serve hundreds of other requests concurrently`.
2. Using blocking pika would freeze the event loop, slowing down other requests.
3. `aio-pika` lets the service publish/consume messages asynchronously without blocking HTTP request handling.

In classic distributed task queues, using pika vs aio-pika doesn’t matter much, because each worker is isolated.
aio-pika shines when you combine message handling with other async workloads in the same process.
