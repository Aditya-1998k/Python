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


