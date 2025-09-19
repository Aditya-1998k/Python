## Descriptors – Control attribute access
A descriptor is a python object that controls access to another attribute through implementing any of these
special methods.

- `__get__(self, instance, owner)` → controls attribute reading
- `__set__(self, instance, value)` → controls attribute writing
- `__delete__(self, instance)` → controls attribute deletion

If a class defines any of these, it becomes a descriptor.

#### Why to use Descriptor?
They allow you to customize attribute access:
1. Validation (type check, range, format etc)
2. Computed Properties
3. Lazy Loading/ Caching
4. Logging/Debugging

```python
class PositiveNumber:
    def __get__(self, instance, owner):
        return instance.__dict__.get(self.name)

    def __set__(self, instance, value):
        if value < 0:
            raise ValueError(f"{self.name} must be >= 0")
        instance.__dict__[self.name] = value

    def __set_name__(self, owner, name):
        self.name = name  # auto-called to know attribute name


class Person:
    age = PositiveNumber()

    def __init__(self, age):
        self.age = age
```

Usages:
```python
p = Person(25)
print(p.age)     # 25

p.age = -5       # ValueError: age must be >= 0
```

```
           ┌─────────────----┐
           │   p = Person()  │
           └───────┬─────----┘
                   │
                   ▼
          ┌───────────────────-─┐
          │ Person.age          │  ← class attribute (descriptor)
          │ type: PositiveNumber│
          └─────────┬──────────-┘
                    │
   ┌────────────────┴─────────────────--┐
   │ p.age = 30                         │
   │ └─ triggers PositiveNumber.__set_  │
   │      ├─ checks if value >= 0       │
   │      └─ stores in p.__dict__['age']│
   └──────────────────────────────────--┘
                    │
                    ▼
           ┌───────────────────┐
           │ print(p.age)      │
           │ └─ triggers__get__│
           │      └─ returns p.__dict__['age'] │
           └───────────────────┘
```
