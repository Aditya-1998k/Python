## Reflection / Introspection

**Introspection:** The ability of program to examine itself at runtime
(objects, classes, functions, attributes etc.)

**Reflection:** The ability of program to modify itself at runtime (change attributes, 
call methods dynamically etc.)


Useful BUILT-IN Function:
1. **type(obj)** : Returns the type class of an objects.
```python
x = [1, 2, 3]
print(type(x))   # <class 'list'>
```
2. **dir(obj)** : List all attributes and method of an objects.
```
print(dir(x))  # shows methods like append, extend, etc.
```
3. **id(obj)** : Memory address (unique identity)
```python
print(id(x))
```
4. **callable(obj)** : check if object is callable (function, method etc)
```python
print(callable(len))   # True
print(callable(123))   # False
```
Note:
1. **function** : Standalone block of code defined using `def` or (`lambda)`
It belongs to the module (or namespace) where it's defined.
```python
def greet(name):
    return f"Hello {name}"

print(greet("Aditya"))   # "Hello Aditya"
print(type(greet))       # <class 'function'>
```
Here greet is just a function object, not tied to any class or instance.

2. **method**: A method is a function defined inside a class. It is usually
  called instance of the class. Python automatically passes the instance (`self`) as the
  first argument.
```
class Person:
    def greet(self, name):
        return f"Hello {name}, I am {self}"

p = Person()
print(p.greet("Aditya"))   # "Hello Aditya, I am <__main__.Person object...>"
print(type(p.greet))       # <class 'method'>
```
- `greet` (inside class) = function at class level.
- `p.greet` (bound to instance p) = method.

## Reflection Functions
1. `getattr()`
Get attribute dynamically.
```python
class Person:
    def __init__(self, name):
        self.name = name

p = Person("Aditya)
print(getattr(p, "name")
print(getattr(p, "name", "NA")
```
Output:
```
Aditya
NA
```

2. `setattr()`
Set attribute dynamically.
```
setattr(p, "age", 25)
print(getattr(p, "age")) # Output: 25
```

3. `hasattr()`
Check if attribute exists.
```python
print(hasattr(p, "name"))  # True
print(hasattr(p, "age"))   # True (since we added it)
```
4. `delattr()`
Delete attribute Dynamically.
```python
delattr(p, "age")
print(hasattr(p, "age")
```
Output:
```
False
```

**Usecase: Pluggin system via Reflection**

This is used in ORMs, rule-engine, dynamic API and plugin-system etc.
```python
class MathOps:
    def add(self, x, y): return x+y
    def subs(self, x, y): return x-y

ops = MathOps()

operation = "add"

new_func = getattr(ops, operation)
result = new_func(10,5)
print(result) # 15
```
Here reflection lets you call methods dynamically based on strings.


### Introspection
Introspection = **Looking inside** objects at runtime.
Python makes this easy, as everything in python is objects.

**Built-in Introspection Functions**
```python
x = [1, 2, 3]

print(type(x))           # <class 'list'>
print(isinstance(x, list))  # True
print(dir(x))            # shows all methods/attributes of list
print(callable(len))     # True (len is a function)
print(callable(42))      # False
```

**The Inspect Module**:

The `inspect` module = deeper introspection toolbox.
It lets you analyze functions, classes, methods, modules, code objects.

#### 1. Inspect Function Signature
```python
import inspect

def greet(name, times=1):
    """Say Hello multiple times."""
    for _ in range(times):
        print(f"Hello, {name}")

# Method signature
print(inspect.signature(greet))

# Function docs
print(inspect.getdoc(greet))
```
Output:
```
(name, times=1)
Say hello multiple times.
```
#### 2. Get Source Code
```python
import inspect

print(inspect.getsource(greet))
```
Output:
```
def greet(name, times=1):
    """Say hello multiple times."""
    for _ in range(times):
        print(f"Hello, {name}")
```
#### 3. Check if Object is Function, Method, Class, etc.
```python
class Person:
    def hello(self): pass

print(inspect.isfunction(greet))    # True
print(inspect.ismethod(Person().hello))  # True
print(inspect.isclass(Person))      # True
print(inspect.ismodule(inspect))    # True
```
#### 4. Get class members
```python
print(inspect.getmembers(Person))
```
This returns a list of (name, attribute) pairs → useful to see everything inside a class.

#### 5. Find Caller Information
```python
import inspect

def who_called_me():
    frame = inspect.stack()[1]
    print("Caller:", frame.function)

def test():
    who_called_me()

test()
```
Output:
```
Caller: test
```

#### Cheatseat (Introspection and Inspect)
```
| Task                                | Function                  |
|-------------------------------------|---------------------------|
| Get type                            | `type(obj)`               |
| Get attributes                      | `dir(obj)`                |
| Is instance of class                | `isinstance(obj, cls)`    |
| Is subclass                         | `issubclass(sub, base)`   |
| Is callable                         | `callable(obj)`           |
| Get docstring                       | `inspect.getdoc(obj)`     |
| Get source code                     | `inspect.getsource(obj)`  |
| Get function signature              | `inspect.signature(func)` |
| Is function                         | `inspect.isfunction(obj)` |
| Is method                           | `inspect.ismethod(obj)`   |
| Is class                            | `inspect.isclass(obj)`    |
| Is module                           | `inspect.ismodule(obj)`   |
| Get members of class/module         | `inspect.getmembers(obj)` |
```
