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


