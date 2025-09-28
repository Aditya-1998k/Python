# Distributed Systems & Cloud – Python

This repository covers essential concepts in **Distributed Systems & Cloud** with Python.

---

## **Topics**

1. **Celery / RQ – Distributed Task Queues**
   - Celery: Asynchronous distributed task queue.
   - RQ: Simpler Redis-based queue.
   - Key Concepts: Producer, Broker, Worker, Result Backend.

2. **Ray / Dask – Distributed Computation**
   - Ray: Distributed Python tasks and actors.
   - Dask: Parallel computing for arrays, dataframes, and delayed execution.
   - Key Concepts: Task graph, lazy evaluation, scaling across cores/nodes.

3. **Kubernetes Clients – Automating Deployments**
   - Python client libraries to interact with Kubernetes clusters.
   - Use Cases: Create/update pods, services, configmaps programmatically.
   - Note: Useful if working with K8s clusters; can be skipped if not.

4. **Async Event-Driven Systems – asyncio + Message Brokers**
   - Asyncio: Non-blocking concurrent Python execution.
   - Combined with RabbitMQ/Kafka for event-driven architectures.
   - Key Concepts: Event loop, coroutines, async message consumption.

5. **Serverless Python – AWS Lambda, GCP Functions**
   - Run Python code without managing servers.
   - Event-driven, stateless, and scalable.
   - Key Concepts: Trigger → Cloud Function → Storage/Output → Monitoring.

---

## **Common Interview Questions**

### Celery / RQ
- What is a message broker and why is it needed?
- Difference between Celery and RQ.
- Explain Celery worker, beat, and result backend.
- How are retries handled in Celery?

### Ray / Dask
- Difference between Ray and Dask.
- How does lazy evaluation work in Dask?
- Explain Ray tasks vs actors.

### Async Event-Driven Systems
- Difference between synchronous and asynchronous code in Python.
- Why use aio-pika instead of pika?
- What is the role of the event loop and coroutines?

### Serverless Python
- What are cold starts?
- How do AWS Lambda and GCP Functions scale?
- How is state managed in serverless functions?
- What triggers can invoke serverless functions?

---
