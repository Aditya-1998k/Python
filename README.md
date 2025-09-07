# 🚀 Advanced Python Concepts

## 🔹 Concurrency & Parallelism
- **Threading** – Lightweight execution, good for I/O-bound tasks.
- **ThreadPoolExecutor** – Manages a pool of worker threads.
- **Multiprocessing** – True parallelism using separate processes (bypasses GIL).
- **ProcessPoolExecutor** – Abstraction for multiprocessing.
- **Asyncio (async/await)** – Single-threaded cooperative multitasking.
- **Greenlets & Gevent** – Coroutine-based concurrency.
- **Joblib / Dask** – High-level parallelism libraries.

---

## 🔹 Networking & Communication
- **Sockets (TCP/UDP)** – Low-level network communication.
- **ZeroMQ / PyZMQ** – High-performance messaging.
- **gRPC** – Remote Procedure Calls.
- **HTTP Clients** – `requests`, `httpx`, `aiohttp`.
- **WebSockets** – Real-time communication.
- **RPC / Message Queues** – RabbitMQ, Kafka, Celery.

---

## 🔹 System & Performance
- **Memory Management** – `gc`, `weakref`, `sys.getsizeof`.
- **Cython / Numba** – Speeding up Python with C extensions.
- **Profiling** – `cProfile`, `line_profiler`, `memory_profiler`.
- **Multithreaded I/O** – `selectors`, epoll, kqueue.
- **Shared Memory** – `multiprocessing.shared_memory`.

---

## 🔹 Metaprogramming
- **Decorators** – Modify behavior of functions/classes.
- **Context Managers** – Custom `__enter__` and `__exit__`.
- **Descriptors** – Control attribute access.
- **Metaclasses** – Control class creation.
- **AST (Abstract Syntax Trees)** – Analyze/modify Python code dynamically.
- **Reflection / Introspection** – `inspect`, `getattr`, `setattr`.

---

## 🔹 Data & Computation
- **NumPy / Pandas Internals** – Vectorization, broadcasting.
- **Generators & Itertools** – Lazy evaluation.
- **Coroutines & Pipelines** – Stream data processing.
- **RAG / Vector Databases** – Advanced ML/NLP workloads.

---

## 🔹 Security & Cryptography
- **SSL/TLS with Sockets** – Encrypted communication.
- **Hashlib & HMAC** – Cryptographic hashing.
- **Cryptography library** – Symmetric/asymmetric encryption.
- **Key Management** – Fernet, RSA, ECC.

---

## 🔹 Distributed Systems & Cloud
- **Celery / RQ** – Distributed task queues.
- **Ray / Dask** – Distributed computation.
- **Kubernetes Clients** – Automating deployments.
- **Async Event-Driven Systems** – `asyncio` + message brokers.
- **Serverless Python** – AWS Lambda, GCP Functions.

---

## 🔹 CI/CD & Automation
- **Jenkins with Python** – Automating jobs via Python scripts.
- **GitLab CI/CD with Python** – Pipelines with `.gitlab-ci.yml`.
- **GitHub Actions (Python workflows)** – Python-based build/test/deploy automation.
- **Dagger (Python SDK)** – CI/CD pipelines as code in Python.
- **Ansible** – Infrastructure automation with Python.
- **Fabric / Invoke** – Automating deployments and tasks.
- **Prefect / Airflow** – Workflow orchestration with Python.

---

## 🔹 Other Interesting Areas
- **Design Patterns** – Singleton, Factory, Observer.
- **Functional Programming** – `functools`, `toolz`.
- **DSLs (Domain-Specific Languages)** – Mini-languages in Python.
- **Plugin Systems** – Dynamically load modules.
- **Testing & Mocking** – Advanced `pytest`, fixtures, mocking I/O.
