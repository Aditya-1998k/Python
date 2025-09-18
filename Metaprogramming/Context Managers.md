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
        if exc_type or exc_val:
              return False  # Raise Exception outside
        else:
              return True # Raise No exception outside (event if exc_tb is there)
```
When Python exits a with block, it always calls `__exit__`.
If an exception occurred inside the block, Python passes details of the exception into these three arguments:
1. `exc_type`: the class of the exception (e.g., ZeroDivisionError)
2. `exc_val`: the actual exception object (e.g., ZeroDivisionError('division by zero'))
3. `exc_tb`: traceback object (where the error happened)
4. If no error occurred: `exc_type`, `exc_val`, `exc_tb` --> all are None

**Return value of __exit__** (Imprtant

Return False (or None)
```
The exception (if any) is propagated after cleanup.
Meaning: you still see the error if there is any error.
```
```
Return True →
The exception is suppressed (ignored).
Python thinks: “I handled it, don’t raise it further.”
```

### Examples
```python
class Facility:
    def __enter__(self):
        print("✅ Show ID")
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        print("Collect goodies")
        if exc_type:
            print(f"Error: {exc_type.__name__} - {exc_val}")
        return False   # change to True to suppress
```

**Case A (if return False):**
If any error occurs, it will raise outside
```python
with Facility():
    1/0
print("This line will not run")
```
Output:
```
✅ Show ID
Collect goodies
Error: ZeroDivisionError - division by zero
Traceback (most recent call last):
  ...
ZeroDivisionError: division by zero
```
Error escapes the block and crashes program.

**Case B (if return True):**
Since `__exit__` function Returning True, means python
will understand you have handled error gracefully.
```python
with Facility():
    1 / 0
print("This line WILL run")
```
Output:
```
✅ Show ID
Collect goodies
Error: ZeroDivisionError - division by zero
This line WILL run
```
Why useful?
1. Clean setup + teardown logic in one place.
2. Prevents forgetting cleanup (like closing files, releasing locks, disconnecting from DB).
3. Exception-safe `__exit__` always runs.

### Why contextlib?
Writing a full class with `__enter__` and `__exit__` can feel heavy for simple usecase.
Python's `context` module provides tools that let's you write context manager with simple functions or 
decorators, instead of class.

