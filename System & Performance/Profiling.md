Profiling helps in analyzing where your program spends times and memory. Helps in
optimize performance bottlenecks.

### 1. cProfile
cProfile measures function call counts and execution time. Good for finding
which functions are slow.

```python
import cProfile
import time

def slow_function():
    total = 0
    for i in range(10_000_000):
        total += i
    return total

def fast_function():
    return sum(range(10_000_000))

cProfile.run("slow_function()")
```

Output:
```
4 function calls in 0.714 seconds

Ordered by: standard name

 ncalls  tottime  percall  cumtime  percall   filename:lineno(function)
    1    0.000    0.000    0.714    0.714      <string>:1(<module>)
    1    0.714    0.714    0.714    0.714      perf.py:4(slow_function)
    1    0.000    0.000    0.714    0.714      {built-in method builtins.exec}
    1    0.000    0.000    0.000    0.000      {method 'disable' of '_lsprof.Profiler' objects}
```

We can generate dumps file also to visualize with snakeviz. We can create a decorator with
cProfile which we can use on any function

```python
import cProfile
from functools import wraps

def profile_and_dump():
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            pr = cProfile.Profile()
            pr.enable()
            try:
                return func(*args, **kwargs)
            finally:
                pr.disable()
                pr.dump_stats("/home/gaditya/instances/projects/perf_check.dumps")
        return wrapper
    return decorator

@profile_and_dump()
def slow_function():
    total = 0
    for i in range(10**6):
        total += i
    return total

slow_function()
```
It will generate a file perf_check.dumps which you can visualize using `snakeviz`

```bash
pip install snakeviz
snakeviz perf_check.dumps
```



