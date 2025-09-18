## Context Managers
A context manager is a python object that sets something up when you `enter` the block of a code and do cleanup or something when you `exit` the
block of code.

```
                ┌───────────────────────┐
                │  with Facility():      │
                └──────────┬────────────┘
                           │
                           ▼
             ┌───────────────────────────┐
             │ Facility.__enter__()       │
             │   → "✅ Show your ID"      │
             └──────────┬────────────────┘
                        │
                        ▼
            ┌────────────────────────────┐
            │  Code inside the block      │
            │  "🚶 Inside the facility"   │
            └──────────┬─────────────────┘
                       │
                       ▼
           ┌─────────────────────────────┐
           │ Facility.__exit__()          │
           │   → "🎁 Collect goodies"     │
           └─────────────────────────────┘

```

Common example:
```python
with open("file.txt", "r"):
    data = f.read()
```
Explaination:
1. `open("file.txt", "r")` returns a context manager (the file object).
2. When entering the with block ---> the file is opened (`__enter__ runs).
3. When leaving the with block → the file is automatically closed (`__exit__` runs).

**Internal Structure:**
A context manager is any object that defines two special methods:
```python
class MyContext:
    def __enter__(self):
        # setup code
        return something   # this is what 'as ...' receives

    def __exit__(self, exc_type, exc_val, exc_tb):
        # cleanup code (always runs, even if there’s an error)
        return False  # if True, suppresses the exception
```
When Python exits a with block, it always calls `__exit__`.
If an exception occurred inside the block, Python passes details of the exception into these three arguments:
1. `exc_type`: the class of the exception (e.g., ZeroDivisionError)
2. `exc_val`: the actual exception object (e.g., ZeroDivisionError('division by zero'))
3. `exc_tb`: traceback object (where the error happened)
