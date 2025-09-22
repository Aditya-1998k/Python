# Reflection / Introspection – inspect, getattr, setattr.

**Introspection:** The ability of program to examine itself at runtime
(objects, classes, functions, attributes etc.)

**Reflection:** The ability of program to modify itself at runtime (change attributes, 
call methods dynamically etc.)


### Useful BUILT-IN Function
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
- **function** : Standalone block of code defined using `def` or (`lambda)`
It belongs to the module (or namespace) where it's defined.
```python
def greet(name):
    return f"Hello {name}"

print(greet("Aditya"))   # "Hello Aditya"
print(type(greet))       # <class 'function'>
```
Here greet is just a function object, not tied to any class or instance.

- **method**: A method is a function defined inside a class. It is usually
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
