## Greenlets
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

### Gevents
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

### Joblib (High level parallelism)
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

### Dask
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

