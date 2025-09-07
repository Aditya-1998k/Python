What is ThreadpoolExecutor?

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

**Submitting Tasks with submit()**
```python
from concurrent.futures import ThreadPoolExecutor
import time

def task(n):
    print(f"Task {n} starting...")
    time.sleep(2)
    return f"Task {n} done"

with ThreadPoolExecutor(max_workers=3) as executor:
    futures = [executor.submit(task, i) for i in range(5)]

    for f in futures:
        print(f.result()) # Waits unitll result is not ready
```
