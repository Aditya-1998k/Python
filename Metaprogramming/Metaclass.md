## Metaclass
A metaclass is `class of class`. 
In python class themselves are objects, and metaclasse defines how classes are created.

Python classes are instance of `type`.
```python
class MyClass:
    pass

print(type(MyClass))  # <class 'type'>
```
Here, type is the default metaclass, creating MyClass

**Why Use Metaclasses?**

Metaclasses allow you to control class creation:
1. Modify class attributes or methods automatically.
2. Enforce rules on subclasses (e.g., required attributes).
3. Register classes (for plugins, ORM models, etc.).
4. Automatically decorate or wrap methods.
5. Implement singleton patterns or other design patterns at class level.

#### Defining Custom Class
```python
# Define a custom metaclass
class MyMeta(type):
    def __new__(cls, name, bases, dct):
        print(f"Creating class {name}")
        return super().__new__(cls, name, bases, dct)

# Use the metaclass
class MyClass(metaclass=MyMeta):
    x = 10

Output:
Creating class MyClass
```
metaclass=MyMeta tells Python to use MyMeta to create MyClass.

`MyMeta.__new__` is called with:
- `cls` → the metaclass itself (MyMeta)
- `name` → name of the new class (MyClass)
- `bases` → tuple of base classes
- `dct` → dictionary of class attributes and methods
- `__new__` returns the class object.

`__new__ vs __init__` in Metaclasses:

Method	Purpose:
1. `__new__(cls, name, bases, dct)`	Called before the class is created. Can modify dct. Must return the new class object.
2. `__init__(cls, name, bases, dct)`	Called after the class is created. Can customize the class further but cannot replace it.

```python
class MyMeta(type):
    def __new__(cls, name, bases, dct):
        print("Inside __new__")
        dct['meta_attribute'] = 'added by metaclass'
        return super().__new__(cls, name, bases, dct)
    
    def __init__(cls, name, bases, dct):
        print("Inside __init__")
        super().__init__(name, bases, dct)

class MyClass(metaclass=MyMeta):
    pass

print(MyClass.meta_attribute)  # added by metaclass
```
Output:
```
Inside __new__
Inside __init__
added by metaclass
```
---

## Real World Example
### 1. Automatic Attribute Injection
```python
class AutoAttrs(type):
    def __new__(cls, name, bases, dct):
        dct['created_by_metaclass'] = True
        return super().__new__(cls, name, bases, dct)

class Person(metaclass=AutoAttrs):
    pass

print(Person.created_by_metaclass)  # True
```
Use case: Automatically tag classes for special processing (like ORM models like sqlalchemy).

### 2. Enforcing Class Rules
```python
class MustHaveNameMeta(type):
    def __new__(cls, name, bases, dct):
        if 'name' not in dct:
            raise TypeError(f"Class {name} must define a 'name' attribute")
        return super().__new__(cls, name, bases, dct)

class Person(metaclass=MustHaveNameMeta):
    name = "Alice"  # ✅ OK

class Animal(metaclass=MustHaveNameMeta):
    pass  # ❌ Raises TypeError
```
