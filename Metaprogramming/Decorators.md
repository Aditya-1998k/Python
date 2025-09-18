## What is Decorator?
A decorator is just a function that takes another function (or class) as input
and returns a modified function (or class).

```python
@decorator
def my_function():
    print("Hello")
```

is same as below code

```python
def my_function():
    print("Hello")

decorator(my_function)
```

Example:
```python
import time

def timer(func):
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        end = time.time()
        print(f"{func.__name__} took {end - start:.4f} seconds")
        return result
    return wrapper

@timer
def slow_function():
    time.sleep(2)
    return "Done"

slow_function()
```
Output:
```
slow_function took 2.0021 seconds
```

Explaination:
1. `timer` is a decorator function which takes input as function (`func`)
2. Inside timer we define another function `wrapper`
3. `wrapper` accept any positional and keyword arguments (*args, **kwargs), which
makes flexible enough decorator to wrap any function
4. `result = func(*args, **kwargs)` calls the original function with it's argument (args or kwargs)
5. `func.__name_` return the name of original function
6. `return wrapper`, means nothing but returning the decorated function with decorator wrapper
7. So, when you are calling `slow_function()`, actually you are calling `wrapper()` of decorator function.
8. So, returning `wrapper` is important, because python internally calls `wrapper` function
