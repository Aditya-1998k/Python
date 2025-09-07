# ProcessPoolExecutor – Abstraction for Multiprocessing
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
  
**Basic Examples**
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
