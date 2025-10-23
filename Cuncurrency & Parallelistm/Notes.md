# Python Concurrency & Parallelism Notes

---

## Table of Contents
1. [Overview](#overview)
2. [Threading](#threading)
3. [ThreadPoolExecutor](#threadpoolexecutor)
4. [ProcessPoolExecutor](#processpoolexecutor)
5. [Multiprocessing](#multiprocessing)
6. [Asyncio](#asyncio)
7. [Third Party Tools](#third-party-tools)
8. [Practice Questions](#practice-questions)

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

## ProcessPoolExecutor
**Abstraction for Multiprocessing** 
This is also part of `concurrent.futures` module.
Provide a high level API for running tasks in parallel using multiple processes.
Abstract away the complexity of `multiprocessing.Process` and `multiprocessing.Pool`.

Useful for CPU bound task: True Parallelism (Bypass GIL)

**Key Features:**
1. Easy to submit and manage CPU-intensive tasks.
2. Automatically creates a pool of worker processes.
3. Methods
   - `submit(fn, *args, **kwargs)` → schedules a single task (returns Future).
   - `map(fn, iterable)` → runs a function on an iterable (parallel equivalent of built-in map).
   - `as_completed()` → iterate over futures as they complete.
   - `shutdown()` → gracefully stop workers (usually handled via context manager with).
  
#### **Basic Examples**
```python
import time
import os
from concurrent.futures import ProcessPoolExecutor

def task(n):
    print(f"Task {n} starting in process {os.getpid()}\n")
    time.sleep(2)
    return f"Task {n} done."

if __name__ == "__main__":
    with ProcessPoolExecutor(max_workers=3) as executor:
        futures = [executor.submit(task, i) for i in range(5)]

        for f in futures:
            print(f.result()) # Wait for the result
```
Output:
```
Task 0 starting in process 51605
Task 1 starting in process 51606
Task 2 starting in process 51607
Task 3 starting in process 51605
Task 4 starting in process 51606

Task 0 done.
Task 1 done.
Task 2 done.
Task 3 done.
Task 4 done.
```

#### Using map() (parallel version of map)
```python
from concurrent.futures import ProcessPoolExecutor
import time

def squere(n):
    time.sleep(n)
    return n*n

if __name__ == "__main__":
    with ProcessPoolExecutor(max_workers=3) as executor:
        result = executor.map(squere, [5,2,3,4,1])
        print(list(result)) # Collect the result
```
Output:
```
[25, 4, 9, 16, 1]
```

#### Using as_completed()
Iterate over futures as they complete.

```python
from concurrent.futures import ProcessPoolExecutor, as_completed
import time

def cube(n):
    time.sleep(n)  # simulate different runtimes
    return n**3

if __name__ == "__main__":
    with ProcessPoolExecutor(max_workers=3) as executor:
        futures = [executor.submit(cube, i) for i in [6,3,2,4,5]]

        for f in as_completed(futures):
            print("Result:", f.result())
```
Output (order depends on task completion time):
```
Result: 8
Result: 27
Result: 216
Result: 64
Result: 125
```

---

## Multiprocessing
Python's GIL prevents `multiple threads` from executing bytecode simultnously.
For `CPU bound` tasks (Heavey computation), threading does not help.
Eg: Heavy computation, Image Processing, ML Training etc

Multiprocessing achieves true parallelism by using seperate processes.
Each process has its own `interpreter and memory` space.

**Key concepts:**
1. `Process` → independent execution unit with its own memory.
2. `Pool` → manages a fixed number of worker processes.
3. `IPC (Inter-Process Communication)` → since processes don’t share memory, use Queue, Pipe, or Manager.

#### Creating Process
```python
from multiprocessing import Process
import os
import time

def task(n):
    print(f"Process {n} starting (PID={os.getpid()})")
    time.sleep(2)
    print(f"Process {n} done")

if __name__ == "__main__":   # required on Windows
    processes = [Process(target=task, args=(i,)) for i in range(5)]

    for p in processes:
        p.start()

    for p in processes:
        p.join()

    print("All processes finished")

```
Output:
```
Process 0 starting (PID=1208)
Process 1 starting (PID=1211)
Process 2 starting (PID=1214)
Process 4 starting (PID=1222)Process 3 starting (PID=1217)

Process 0 done
Process 1 done
Process 2 done
Process 4 doneProcess 3 done

All processes finished
```
Note: Each task runs in a separate process (different PID).

#### Process Pool
Simplifies managing multiple processes.

```python
from multiprocessing import Pool
import time

def squere(n):
    time.sleep(n)
    return n*n

if __name__ == "__main__":
    with Pool(processes=3) as pool:
        results = pool.map(squere, [3,1,2])
        print(results)
```
Output:
```
[9, 1, 4]
```
map() → distributes tasks, preserves input order.

#### Asynchronous Execution with `apply_async`
apply_async() → submit tasks asynchronously.

```python
from multiprocessing import Pool

def cube(n):
    return n*n*n

if __name__ == "__main__":
    with Pool(3) as pool:
        result = pool.apply_async(cube, (4,))  # Async call
        print("Result: ", result.get())        # Wait and fetch result
```

### Inter-Process Communication (IPC)
Since each process have different memory, so sharing data between each other is difficult.
We have several ways to do the inter process communication.
- Using queue
- Using Pipe
- Using manager

#### 1. Using `Queue`
```python
from multiprocessing import Process, Queue

def worker(q, x):
    # putting result in queue
    q.put(x*x)

if __name__ == "__main__":
    # A multiprocessing-safe queue is created.
    # This is used for inter-process communication.
    q = Queue()
    processes = [Process(target=worker, args=(q,i)) for i in range(5)]

    for p in processes: p.start()
    for p in processes: p.join()

    results = [q.get() for _ in range(5)]
    print("Results:", results)
```
So the queue will eventually hold: 0, 1, 4, 9, 16 (order not guaranteed).
```
Results: [0, 1, 4, 9, 16]
```
Again if you try to do `q.get()` 
At this point, the queue is empty!. 
`q.get()` will block forever, waiting for new data that never comes.
So you should do same number of q.get() as number of q.put() done.

#### 2. Using `Pipe`
Provides two connection objects for two-way communication.

```python
from multiprocessing import Process, Pipe

def worker(conn):
    conn.send("Hello from child")
    conn.close()

if __name__ == "__main__":
    parent_conn, child_conn = Pipe()
    p = Process(target=worker, args=(child_conn,))
    p.start()
    print(parent_conn.recv()) # Recieve message from child conn
    p.join()
```
Output:
```
Hello from child
```

#### 3. Using `Manager`
Provides a way to share Python objects across processes.

```python
from multiprocessing import Process, Manager

def worker(d, key,  value):
    d[key] = value

if __name__ == "__main__":
    with Manager() as manager:
        shared_dict = manager.dict()
        processes = [Process(target=worker, args=(shared_dict, i, i*i)) for i in range(5)]

        for p in processes: p.start()
        for p in processes: p.join()

        print(shared_dict)
```
Output:
```
{0: 0, 1: 1, 3: 9, 2: 4, 4: 16}
```

#### Synchronization in Multiprocessing
In multiprocessing, each process has its own memory space, so usually no race condition in normal variables.
Sometime just like in threads, processes may also needs locks. If some variable is shared across processes.

```python
from multiprocessing import Process, Lock

def worker(lock, i):
    with lock:
        print(f"Process {i} is working")

if __name__ == "__main__":
    lock = Lock()
    processes = [Process(target=worker, args=(lock, i)) for i in range(5)]

    for p in processes: p.start()
    for p in processes: p.join()
```
Output:
```
Process 1 is working
Process 0 is working
Process 3 is working
Process 2 is working
Process 4 is working
```
OS decides scheduling, so order changes each run.

---

## Asyncio
**Cooperative Multitasking**  
`asyncio` is python built-in framework for asynchronous programming.
Provides single threaded, single process concurrency using `event loop.`
Based on coroutines (`async def` function)

Good For:
- Ideal for I/O bound tasks (network requests, DB queries, file IO) - Avoid Blocking
- Use cooperative multitasking : tasks voluntarily give up control via `await`.

**Key Concept:**
1. `Coroutine`: A special function defined with `async def`
2. `Event Loop`: Schedular that runs coroutines concurrently
3. `await`: Suspends coroutines untill awaited task completes
4. `Task`: Wrapper around coroutine, scheduled by event loop.
5. `asyncio.gather()` : Run multiple coroutines concurrently.
6. `asyncio.run()` : Entry point to run async code.

#### Basic Example
```python
import asyncio

async def say_hello():
    print("Hello...")
    await asyncio.sleep(5)   # non-blocking sleep
    print("World!")

asyncio.run(say_hello())
print("Hello Outside coroutines")
```
Output:
```
Hello...
World!
Hello Outside coroutines
```
Only say_hello is registered.
At `await asyncio.sleep(5)`, the event loop has nothing else scheduled, so it just idles.

**Note:** `Hello Outside coroutines` is outside the event loop, so it only runs after the loop finishes.

#### Running Multiple Task (Using asyncio.gather)
```python
import asyncio

async def task(name, delay):
    print(f"{name} started")
    await asyncio.sleep(delay)     # non-blocking, lets event loop run other tasks
    print(f"{name} finished after {delay}s")
    return f"{name} result"

async def main():
    results = await asyncio.gather(
        task("Task1", 2),
        task("Task2", 1),
        task("Task3", 3)
    )
    print("Results:", results)

asyncio.run(main())
# Starts the event loop.
# Runs main() until completion.
```

Defines an async coroutine task.
Each task simulates some I/O by sleeping for delay seconds.
`await asyncio.sleep(delay)` does not block. 
It pauses this coroutine and allows other tasks to run on the event loop.

`asyncio.gather()` runs all provided tasks concurrently.
Each `task(...)` is a coroutine, and `gather` schedules them on the event loop.
`await asyncio.gather(...)` waits until all tasks finish
and returns a list of results in the same order as passed.

Output:
```
Task1 started
Task2 started
Task3 started
Task2 finished after 1s
Task1 finished after 2s
Task3 finished after 3s
Results: ['Task1 result', 'Task2 result', 'Task3 result']
```

#### Creating Tasks Explicitly
```python
import asyncio

async def worker(n):
    print(f"Worker {n} started")
    await asyncio.sleep(n)
    print(f"Worker {n} finished")

async def main():
    tasks = [asyncio.create_task(worker(i)) for i in [1,3,2]]
    await asyncio.gather(*tasks)

asyncio.run(main())
```
Output:
```
Worker 1 started
Worker 3 started
Worker 2 started
Worker 1 finished
Worker 2 finished
Worker 3 finished
```

#### Awaiting as They Complete
```python
import asyncio

async def download(n):
    await asyncio.sleep(n)
    return f"Downloaded {n}"

async def main():
    tasks = [asyncio.create_task(download(i)) for i in [3, 1, 2]]
    for task in asyncio.as_completed(tasks):
        result = await task
        print(result)

asyncio.run(main())
```
Output:
```
Downloaded 1
Downloaded 2
Downloaded 3
```

#### Mixing sync and Async
asyncio runs on one thread (event loop).
If you call a blocking function (like time.sleep(2)), the whole loop freezes → no other coroutine can run.
```python
import asyncio, time

def blocking_task():
    time.sleep(2)              # <-- blocks the thread
    return "Done (blocking)"
```
above blocking_task function is sync function. It can freeze whole loop.

```python
async def main():
    print("Start async")
    result = await asyncio.to_thread(blocking_task)
    print(result)
```
**asyncio.to_thread(blocking_task)**
1. Runs blocking_task in a separate thread, using the default thread pool.
2. Returns an awaitable (a coroutine), so you can await it.
3. While it runs, the event loop is free to handle other async tasks.

**await asyncio.to_thread(...)**
1. Suspends main() until the blocking task finishes.
2. Meanwhile, other coroutines (if any) can continue running.

**asyncio.run(main())**: Starts the event loop and runs main.

Output:
```
Start async
Done (blocking)
```

---

## Third Party Tools
#### Greenlets
1. Third-party (pip install greenlet).
2. Lightweight coroutines in same thread.
3. Explicit context switch using .switch().

```python
from greenlet import greenlet

def foo():
    print("Foo start")
    gr2.switch()
    print("Foo end")
    gr2.switch()

def bar():
    print("Bar start")
    gr1.switch()
    print("Bar end")

gr1, gr2 = greenlet(foo), greenlet(bar)
gr1.switch() # Start with gr1
```
Output:
```
Foo start  # Then switch to gr2
Bar start  # then swithch to gr1
Foo end    # then switch to gr2
Bar end
```

#### Gevents
1. Third Party (!pip install gevent)
2. High level concurrency built on `greenlet`.
3. Provides `gevent.spawn()`, `joinall()`, `gevent.sleep()`
4. Can monkey patch stdlib (socket, time etc) - makes blocking I/O non-blocking.

```python
import gevent

def task(name, delay):
    print(f"{name} start")
    gevent.sleep(delay)
    print(f"{name} done")

g1 = gevent.spawn(task, "T1", 2)
g2 = gevent.spawn(task, "T2", 1)
gevent.joinall([g1, g2])
```
Output:
```
T1 start
T2 start
T2 done
T1 done
[<Greenlet at 0x7dce7c5ca200: _run>, <Greenlet at 0x7dce7c5c9da0: _run>]
```

#### Joblib (High level parallelism)
1. Third party (pip install joblib)
2. Simple parallel for-loops (Parallel, delayed)
3. Good for CPU bound task (ML Processing, data science)
4. Native in scikit-learn for parallel training

Note: quick parallelism on one machine

```
from joblib import Parallel, delayed
import time

# Define a normal function
def square(x):
    time.sleep(1)   # simulate some work
    return x * x

# Run the function in parallel
results = Parallel(n_jobs=4)(
    delayed(square)(i) for i in range(6)
)

print("Results:", results)
```
Note: **Runtime: ~1.5s instead of 6s (because tasks run concurrently).**
Output:
```
Results: [0, 1, 4, 9, 16, 25]
```
1. `n_jobs=4` → use 4 workers (parallel processes or threads).
2. `delayed(square)(i)` → wraps the function call so Joblib can schedule it later.
3. `Parallel(...)(...)` → executes all tasks in parallel and collects results.

#### Dask
1. Third-party (pip install dask).
2. Scales from local → distributed cluster.
3. Provides dask.delayed, dask.array, dask.dataframe.
4. Best for big data, distributed ML, parallel pandas/numpy.

Note: scalable parallelism for big data & clusters.

```python
from dask import delayed, compute
import time

# Define a normal function
def square(x):
    time.sleep(1)   # simulate work
    return x * x

# Build a list of "delayed" tasks (lazy, not executed yet)
tasks = [delayed(square)(i) for i in range(6)]

# Trigger execution of all tasks in parallel
results = compute(*tasks)

print("Results:", results)
```
Note: **Runtime: ~1.5s instead of 6s (because tasks run concurrently).**

Output:
```
Results: (0, 1, 4, 9, 16, 25)
```
1. `delayed(square)(i)` → creates a lazy task (not executed immediately).
2. `tasks` → holds the computation graph.
3. `compute(*tasks)` → runs them in parallel using Dask’s scheduler.

----

## Practice Questions
### Threading
1. What is the Global Interpreter Lock (GIL), and how does it affect threading?
2. Difference between threads and processes in Python?
3. When is threading preferred over multiprocessing?

### Multiprocessing
4. How does multiprocessing bypass the GIL?
5. How do you share data between processes?
6. What are the drawbacks of multiprocessing (e.g., overhead, memory)?

### Executors
7. Difference between `ThreadPoolExecutor` and `ProcessPoolExecutor`?
8. How do futures (`concurrent.futures.Future`) work?
9. Example: Submitting tasks to a pool and collecting results.

### Asyncio
10. Difference between concurrency and parallelism?
11. How does the asyncio event loop work?
12. Compare asyncio coroutines with threading.

### Third-Party Tools
13. What is `gevent`, and how does it use greenlets?
14. How does `trio` differ from asyncio?
15. When would you choose a third-party async library over the standard library?
