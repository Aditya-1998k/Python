### Garbage Collector (gc module)
Python uses reference counting + cyclic garbage collector for cleaning unused objects.
`gc` module let's you interact with garbage collector.

How to use:
```python
import gc

# Force garbage collection
gc.collect()

# Debug objects tracked by GC
print(gc.get_stats())
```

Some time we need garbage collection:
1. **Circular Reference Problem**
Reference couting alone can't cleanup objects if they reference each other. Here `GC` is needed.

```python
import gc

class Node:
    def __init__(self, name):
        self.name = name
        self.ref = None
    def __del__(self):
        print(f"Deleting {self.name}")

# Create circular reference
a = Node("A")
b = Node("B")
a.ref = b
b.ref = a

# Remove external references
a = None
b = None

print("Forcing GC...")
gc.collect()   # GC cleans up circular reference
```

Real Life scenario:
Linked list, trees, graphs or ORM objects (like sqlalchemy relationship) often creates
cyclic references. Without gc memory will leak because reference counter will never 
become 0.

2. **Debugging Memory Leaks**:
`gc` can help in finding uncollected objects (Potential memory leaks).

```python
import gc

gc.set_debug(gc.DEBUG_LEAK)

class Leaky:
    pass

# Create objects with cycle
objs = [Leaky() for _ in range(5)]
for i in range(len(objs)-1):
    objs[i].ref = objs[i+1]
objs[-1].ref = objs[0]   # full cycle

print("Collecting...")
gc.collect()

print("Unreachable:", gc.garbage)  # Shows leaked objects
```
Real-life scenario:
In web apps (Flask/Django/FastAPI), cyclic references in request handlers or caching layers may cause memory leaks.

3. **Manual GC in Performance-Critical Code**: 
Some workloads (e.g., large batch jobs) may pause due to GC. You can:
- Disable GC during a heavy batch.
- Re-enable & collect after.

```python
import gc

gc.disable()
big_data = [list(range(10000)) for _ in range(1000)]
gc.enable()
gc.collect()
print("Manual GC done")
```
