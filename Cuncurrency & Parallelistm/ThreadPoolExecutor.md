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

executor.submit(func, *args) schedules a function to run in a thread.
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

