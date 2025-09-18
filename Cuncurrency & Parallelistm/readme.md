# Concurrency & Parallelism in Python

This section covers different approaches and tools in Python to achieve concurrency (managing multiple tasks at once) 
and parallelism (executing tasks simultaneously).

---

## 📑 Topics Covered

### 1. Threading
- Lightweight, share the same memory space.
- Good for I/O-bound tasks (e.g., network calls, file I/O).
- Limited by Python’s Global Interpreter Lock (GIL), so not effective for CPU-bound tasks.

### 2. ThreadPoolExecutor
- High-level API for managing a pool of worker threads.
- Simplifies task submission (`submit`, `map`) and result handling via `futures`.

### 3. ProcessPoolExecutor
- Manages a pool of processes (separate memory space).
- Effective for CPU-bound tasks (e.g., data crunching, heavy computation).
- No GIL limitations.

### 4. Multiprocessing
- Directly spawns multiple processes.
- Provides `Process`, `Queue`, `Pipe`, `Manager` for communication.
- Suitable for CPU-bound parallelism.

### 5. Asyncio
- Single-threaded, single-process cooperative multitasking.
- Tasks voluntarily yield control (non-blocking I/O).
- Best for high-concurrency I/O apps like chat servers, web scrapers.

### 6. Third Party Tools
- **gevent** → Coroutine-based, uses greenlets + libev/libuv.
- **eventlet** → Simplified networking library with green threads.
- **Trio / Curio** → Alternative async frameworks focusing on structured concurrency.

---

## 🤔 When to Use What?

- **I/O-bound tasks** (waiting on network/disk): Use `threading`, `asyncio`, or `gevent`.  
- **CPU-bound tasks** (heavy computation): Use `multiprocessing` or `ProcessPoolExecutor`.  
- **Mixed workloads**: Combine approaches, e.g., `asyncio` for I/O + `ProcessPoolExecutor` for CPU.  

---

## ❓ Questions (12)

### Conceptual
1. What is the difference between concurrency and parallelism?  
2. Why is Python threading limited by the Global Interpreter Lock (GIL)?  
3. In what cases would you prefer multiprocessing over threading?  
4. Explain the difference between ThreadPoolExecutor and ProcessPoolExecutor.  
5. How does asyncio differ from threading?  

### Practical
6. Write a simple Python program using `ThreadPoolExecutor` to download multiple URLs concurrently.  
7. Show an example where `ProcessPoolExecutor` outperforms `ThreadPoolExecutor`.  
8. How would you share data between processes in the `multiprocessing` module?  
9. Write a simple asyncio example that runs two coroutines concurrently.  

### Advanced
10. How do third-party tools like gevent achieve concurrency differently from asyncio?  
11. What is structured concurrency and why is it important in frameworks like Trio?  
12. If you had to handle 10,000 socket connections, which concurrency model would you choose and why?  

---

## 🚀 Quick Takeaway

- Threads = I/O concurrency.  
- Processes = CPU parallelism.  
- Asyncio = scalable async I/O in one thread.  
- Executors = easy abstraction for parallelism.  
- Third-party libs = specialized, sometimes more ergonomic.  
