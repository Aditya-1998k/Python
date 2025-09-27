## Distributed task queues
A task queue is a mechanism, where we can run the job simultneously in background
without blocking the main application.

Distributed: Multiple workers can pull the tasks and execute parallally.

Usecase: 
- Sending Mail
- Processing Images and videos
- Background data processing
- Scheduling periodic job (cron)

### Celery:
Celery is most popular distributed task queue in Python.
It uses different message broker (Rabbitmq, Redis etc) as a queue.
Provide workers that consumes and execute tasks.

Supports:
- Scheduling
- Retries
- Task Chaining
- Monitoring

Architecture Diagram:
```
               +--------------------+
               |   Application      |
               |  (Producer/Client) |
               +---------+----------+
                         |
                         |  Task request
                         v
                +-------------------+
                |   Message Broker  |
                | (Redis / RabbitMQ)|
                +---------+---------+
                          |
      +-------------------+-------------------+
      |                                       |
      v                                       v
+-------------+                        +-------------+
|   Worker 1  |                        |   Worker 2  |
| (Consumer)  |                        | (Consumer)  |
+------+------+                        +------+------+
       |                                      |
       | Executes task                        | Executes task
       v                                      v
   +----------+                          +----------+
   |  Result  |                          |  Result  |
   +----+-----+                          +----+-----+
        |                                     |
        +-------------------------------------+
                          |
                          v
                  +---------------+
                  |  Result Store |
                  | (Redis/DB/etc)|
                  +---------------+
```
1. Client which is called Producer pushes the message to Broker.
2. Broker like Rabbitmq, Redis etc holds the message in the queue.
3. Worker picks the task from the queue and executes
4. Then result stored in either redis/DB/S3 etc

For Testing keep Redis Running:
```shell
docker run -d --name redis -p 6379:6379 redis
```

`pip install celery`

consumer.py
```python
from tasks import add

# Send task asynchronously
result = add.delay(4, 6)

print(result)  # AsyncResult object
print(result.id)  # Task ID

# Wait and get result
print(result.get())  # Output: 10
```

tasks.py
```python
# tasks.py
from celery import Celery

app = Celery(
    "my_tasks", broker="redis://localhost:6379/0", backend="redis://localhost:6379/0"
)


@app.task
def add(x, y):
    return x + y
```

Start the celery worker:
```
celery -A tasks worker --loglevel=info
```
- `-A tasks` → tells Celery to load from tasks.py
- `worker` → start worker process
- `--loglevel=info` → print logs

Note: 
1. We can use either cli or flower dashboard to view
2. We can scheduler task with celery `beat`
3. We can configure number of worker also

Execution Flow:
```
Producer (your script)          Redis (Broker)           Worker
---------------------           -----------------        -----------------
add.delay(2,3)   --->   [Queue: "add", args: (2,3)] --->  executes add(2,3)
                                      |
                                      v
                         (Optional) store result
                                in Redis
```

### RQ (Redis Queue)
RQ is alternative to celery but with less features.
It uses only redis as backend and broker.

1. Easy to setup
2. Easy to maintain
3. Less feature than celery
4. Ideal for small features

```
pip install rq rq-dashboard
```

rq-dashboard = Optional web UI

**tasks.py**
```python
# tasks.py
import time

def background_task(x, y):
    print(f"Processing task: {x} + {y}")
    time.sleep(5)  # simulate a long job
    return x + y
```

**producer.py**
```python
# producer.py
from redis import Redis
from rq import Queue
from tasks import background_task

# Connect to Redis (local instance)
redis_conn = Redis(host="localhost", port=6379, db=0)

# Create a queue
q = Queue(connection=redis_conn)

# Enqueue task
job = q.enqueue(background_task, 4, 6)
print("Job enqueued:", job.id)
print("Result (initially None):", job.result)
```

1. Run the producer.py
2. Job will be submitted and you get result as none initially
```
Job enqueued: 4ff8c3bc-8e2b-4e12-b62a-2c13e17f9f17
Result (initially None): None
```
3. Start worker in another terminal
```

```

