# ⚙️ Concurrency & Parallelism in Python

Concurrency and parallelism are key for building **efficient, responsive, and scalable applications**.  
This section covers Python’s built-in and third-party tools for handling **multiple tasks at once**, whether they are **I/O-bound** or **CPU-bound**.

---

## 📂 Topics Covered

1. **Threading** – Lightweight concurrency, best for I/O-bound tasks.
2. **Multiprocessing** – Run tasks in separate processes, bypassing the GIL for CPU-bound workloads.
3. **ThreadPoolExecutor** – High-level thread pool API for concurrent I/O tasks.
4. **ProcessPoolExecutor** – High-level process pool API for parallel CPU-bound tasks.
5. **Asyncio** – Asynchronous programming with event loops, tasks, and coroutines.
6. **Third-Party Tools** – Libraries like `gevent`, `trio`, and `concurrent.futures` extensions.

---

## 🧑‍💻 Why Learn Concurrency & Parallelism?

- Write **non-blocking applications** (servers, scrapers, data pipelines).
- Scale across **multiple cores** for CPU-heavy workloads.
- Improve **latency and responsiveness** in real-time systems.
- Use the right model for **I/O vs CPU tasks**.
- Master high-level abstractions like executors for simpler concurrency.

---

## ❓ Common Practice Questions

### 🔹 Threading
1. What is the Global Interpreter Lock (GIL), and how does it affect threading?
2. Difference between threads and processes in Python?
3. When is threading preferred over multiprocessing?

### 🔹 Multiprocessing
4. How does multiprocessing bypass the GIL?
5. How do you share data between processes?
6. What are the drawbacks of multiprocessing (e.g., overhead, memory)?

### 🔹 Executors
7. Difference between `ThreadPoolExecutor` and `ProcessPoolExecutor`?
8. How do futures (`concurrent.futures.Future`) work?
9. Example: Submitting tasks to a pool and collecting results.

### 🔹 Asyncio
10. Difference between concurrency and parallelism?
11. How does the asyncio event loop work?
12. Compare asyncio coroutines with threading.

### 🔹 Third-Party Tools
13. What is `gevent`, and how does it use greenlets?
14. How does `trio` differ from asyncio?
15. When would you choose a third-party async library over the standard library?

---

## 📖 Next Steps

- Implement a **multi-threaded web scraper**.
- Write a **CPU-bound parallel computation** using multiprocessing.
- Build an **async web server** with `asyncio`.
- Compare performance between **ThreadPoolExecutor vs ProcessPoolExecutor**.
- Explore **gevent** or **trio** for advanced concurrency.

---

✨ Concurrency & parallelism let you unlock **responsiveness, scalability, and efficiency** in Python applications.
