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

