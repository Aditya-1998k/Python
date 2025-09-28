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


