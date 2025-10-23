# ⚙️ Python Concurrency & Parallelism Notes

---

## 📖 Table of Contents
1. [Overview](#overview)
2. [Threading](#threading)
3. [ThreadPoolExecutor](#threadpoolexecutor)
4. [ProcessPoolExecutor](#processpoolexecutor)
5. [Multiprocessing](#multiprocessing)
6. [Asyncio](#asyncio)
7. [Third Party Tools](#third-party-tools)

---

## Overview
Concurrency and parallelism are key for building **efficient, responsive, and scalable applications**.  
This section covers Python’s built-in and third-party tools for handling **multiple tasks at once**, 
whether they are **I/O-bound** or **CPU-bound**.

1. **Threading** – Lightweight concurrency, best for I/O-bound tasks.
2. **Multiprocessing** – Run tasks in separate processes, bypassing the GIL for CPU-bound workloads.
3. **ThreadPoolExecutor** – High-level thread pool API for concurrent I/O tasks.
4. **ProcessPoolExecutor** – High-level process pool API for parallel CPU-bound tasks.
5. **Asyncio** – Asynchronous programming with event loops, tasks, and coroutines.
6. **Third-Party Tools** – Libraries like `gevent`, `trio`, and `concurrent.futures` extensions.


 Why Learn Concurrency & Parallelism?    

- Write **non-blocking applications** (servers, scrapers, data pipelines).
- Scale across **multiple cores** for CPU-heavy workloads.
- Improve **latency and responsiveness** in real-time systems.
- Use the right model for **I/O vs CPU tasks**.
- Master high-level abstractions like executors for simpler concurrency.


## Threading
What is Threading?    
A thread is a light weight unit of execution inside a **process**. Thread shares
same memory space (unlike processes).

**Best for I/O Bound Task:**
- Network Calls
- File I/O
- API Requests    
**Inefficient for CPU Bound Task** Because of Python GIL (Global interpreter Lock)

Basic Example:
```python
import threading
import time

def worker(name):
    print(f"Thread {name} starting")
    time.sleep(2)
    print(f"Thread {name} Ending")

# Creating Thread
t = threading.Thread(target=worker, args=("A",))
t.start()

# Main Thread continue
print("Main thread running...")

# Wait for thread to finish (Optional if you want main thread to wait)
t.join()

print("Main thraed finish")
```

Note: `t.join()` tells the main program: "Wait until this thread finishes before continuing."
```
Execution flow:
Thread A starting
Main thread is running...
(Main waits until Thread A finishes)
Thread A finished
Main thread finished
```
without `t.join()` the main thread does not wait for the worker thread.    
but it also doesn’t exit until `non-daemon` threads are done.

```bash
Execution flow:
Thread A starting
Main thread running...
Main thraed finish
Thread A Ending
```

### Daemon vs Non-Daemon Threads
1. **Non-Daemon Thread** :
Default thread in python. The program wait for them to finish before exiting.
Even if main thread ends, python won't shutdown untill all non-daemon are done.

```python
import threading, time

def worker():
    time.sleep(3)
    print("Worker finished")

# Non-daemon (default)
t = threading.Thread(target=worker)
t.start()

print("Main finished")
```
Output:
```
Main finished
Worker finished
```
The program `didn’t exit` immediately — it waited `3 seconds` for the worker thread.

2. **Daemon Thread**: 
`Background thread` that `kills automatically` when main program exits. Program does not weight for    
daemon thread to finish. Useful for background task like logging, monitoring or cleanup that does not    
need to block program exits.

```python
import threading, time

def log_worker():
    time.sleep(3)
    print("Worker finished")

# Daemon thread
t = threading.Thread(target=log_worker, daemon=True)
t.start()

print("Main finished")
```
Output:
```
Main finished
```
The worker never prints Worker finished — because the program ended before it could finish.        
Note: Think of non-daemon as **important work must finish** and     
daemon as **background helper, stop if main ends.**

---

Running multiple Thread    
```python
import threading
import time

def worker(name):
    print(f"Thread {name} starting")
    time.sleep(2)
    print(f"Thread {name} Ending")

threads = []

for i in range(5):
    t = threading.Thread(target=worker, args=(i,))
    threads.append(t)
    t.start()

# Wait for all threads to complete
for t in threads:
    t.join()

print("All threads finished")
```
---

#### Thread Synchronization (Locks)
When multiple threads shares the same data, use **lock** to prevent race conditions.

Let's create a force race condition
- We can release the GIL by using time.sleep() or C extensions
- Or use Multiprocessing (True Parallelism)

```python
import threading
import time

counter = 0

def increment():
    global counter
    for _ in range(1000):  # fewer iterations so you can see the issue
        value = counter
        time.sleep(0.0001)   # simulate context switch (GIL Release)
        counter = value + 1

threads = []

for i in range(5):
    t = threading.Thread(target=increment)
    threads.append(t)
    t.start()

for t in threads:
    t.join()

print("Final counter:", counter)  # Often < 5000 because of lost updates
```
Output:
```
Eache thread inceasing the counter by 1000, but actually our output will
be less than 5000
output: 1002
```

Let's fix this problem using lock

Why Lock?    
A Lock ensures that **only one thread at a time can access a critical section** (like modifying counter).
So even if multiple threads run, the counter += 1 operation becomes **atomic**.

```python
import threading
import time

counter = 0
lock = threading.Lock()

def increment():
    global counter
    for _ in range(1000):  # fewer iterations so you can see the issue
        with lock:
           value = counter
           time.sleep(0.0001)   # simulate context switch (GIL Release)
           counter = value + 1
threads = []

# start 5 threads
for i in range(5):
    t = threading.Thread(target=increment)
    threads.append(t)
    t.start()

for t in threads:
    t.join()

print("Final counter with lock:", counter)  # Always 5000
```
Output
```
Final counter with lock: 5000
```
---

## ThreadPoolExecutor
**What is ThreadpoolExecutor?**

ThreadpoolExecutor is a class within python's **concurrent.futures** module that provides 
a high level interface (API) for managing a pool of worker threads.

1. A high level API for running function in multiple threads.
2. You don't need to manage `Thread` manually.
3. The executor manages a pool of worker threads for you.

**Creating a Thread Pool**
```python
from concurrent.futures import ThreadPoolExecutor

# create a pool with 3 worker threads
executor = ThreadPoolExecutor(max_workers=3)
```
- max_workers → number of threads in the pool.
- If omitted, defaults to min(32, os.cpu_count() + 4).

#### Submitting Tasks with submit()
```python
from concurrent.futures import ThreadPoolExecutor
import time

def task(n):
    time.sleep(n)
    return f"Task {n} done"

with ThreadPoolExecutor(max_workers=3) as executor:
    futures = [executor.submit(task, i) for i in [3,2,1]]

    for f in futures:
        print(f.result()) # Waits unitll result is not ready
```

`executor.submit(func, *args)` schedules a function to run in a thread.
Returns a Future object.
It will wait till result is not ready. And produce result in order.
```
Task 3 done
Task 2 done
Task 1 done
```

#### Using as_completed()
`as_completed()` lets you process results as soon as tasks finish (instead of waiting in order).

```python
from concurrent.futures import ThreadPoolExecutor, as_completed
import time

def task(n):
    time.sleep(n)
    return f"Finished in {n} sec"

with ThreadPoolExecutor(max_workers=3) as executor:
    futures = [executor.submit(task, i) for i in [3,1,2]]

    for f in as_completed(futures):
        print(f.result())
```
Output:
```
Finished in 1 sec
Finished in 2 sec
Finished in 3 sec
```

#### Using map()
Map will execute in parallel but always produce result in order. As it maintains input orders.
```python
from concurrent.futures import ThreadPoolExecutor
import time

def square(n):
    time.sleep(1)
    return n * n

with ThreadPoolExecutor(max_workers=3) as executor:
    results = executor.map(square, [5,2,3,4,1])
    print(list(results))
```
Output:
```
[25, 4, 9, 16, 1]
```

#### Handling Exception
If task raises exception, it is stored inside the `Future` objects.
```python
from concurrent.futures import ThreadPoolExecutor

def divide(x, y):
    return x / y

with ThreadPoolExecutor() as executor:
    futures = [
        executor.submit(divide, 10, 2),
        executor.submit(divide, 5, 0)  # division by zero
    ]

    for f in futures:
        try:
            print(f.result())
        except Exception as e:
            print("Error:", e)
```
Output:
```
ERROR!
5.0
Error: division by zero
```

#### Cancelling Tasks
`future.cancel()` works only if the task has not started yet.
If already running, cancellation fails.

```python
from concurrent.futures import ThreadPoolExecutor
import time

def work():
    time.sleep(3)
    return "done"

with ThreadPoolExecutor(max_workers=2) as executor:
    for _ in range(4):
        future = executor.submit(work)
        cancelled = future.cancel()
        print("Cancelled:", cancelled)
```
Output:
```
Cancelled: False
Cancelled: False
Cancelled: True
Cancelled: True
```
Here Since we have 2 workers which will take first two task and start processing immediately.
Hence those task will not cancel, but other two task, which submitted not started yet since
workers are busy processing other task, so those will be cancelled.

#### Shutting down the Pool
- The with statement automatically calls `executor.shutdown(wait=True)`.
- `shutdown(wait=False)` → allows main thread to exit without waiting for tasks.

```python
executor.shutdown(wait=True)   # default, waits for tasks
executor.shutdown(wait=False)  # don’t wait
```

Summary:
1. `submit()` → run a function, returns Future.
2. `map()` → parallel map, ordered results.
3. `as_completed()` → process results as soon as ready.
4. Exceptions are raised when you call `future.result()`.
5. `Locks` are still needed if threads modify shared data.

---

## 🧮 ProcessPoolExecutor
*(Content from `ProcessPoolExecutor.md`)*

---

## 🧬 Multiprocessing
*(Content from `Multiprocessing.md`)*

---

## ⏱ Asyncio
*(Content from `Asyncio.md`)*

---

## 🧰 Third Party Tools
*(Content from `Third Party Tools.md`)*

---
