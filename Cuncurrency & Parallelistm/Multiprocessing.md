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

## Inter-Process Communication (IPC)
Since each process have different memory, so sharing data between each other is difficult.
We have several ways to do the inter process communication.

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
⚠️ At this point, the queue is empty!. 
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

### Synchronization in Multiprocessing
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
