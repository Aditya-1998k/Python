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

#### Asynchronous Execution with `apply_sync`
apply_async() → submit tasks asynchronously.

```python

```
