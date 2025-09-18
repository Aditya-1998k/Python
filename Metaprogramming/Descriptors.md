## Descriptors – Control attribute access
A descriptor is a python object that controls access to another attribute through implementing any of these
special methods.

- `__get__(self, instance, owner)` → controls attribute reading
- `__set__(self, instance, value)` → controls attribute writing
- `__delete__(self, instance)` → controls attribute deletion

If a class defines any of these, it becomes a descriptor.

