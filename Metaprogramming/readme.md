
# 📘 Metaprogramming in Python

Metaprogramming is writing code that **generates, manipulates, or modifies other code**.  
It allows programs to treat code as data, enabling powerful abstractions such as custom ORMs, rule engines, and DSLs.

---

## 📂 Topics Covered

1. **Decorators** – Modify or enhance functions and classes without changing their source code.
2. **Context Managers** – Define controlled setup/teardown logic using `__enter__` and `__exit__`.
3. **Descriptors** – Control attribute access (`__get__`, `__set__`, `__delete__`).
4. **Metaclasses** – Control how classes themselves are created.
5. **AST (Abstract Syntax Trees)** – Analyze and transform Python code dynamically.
6. **Reflection & Introspection** – Inspect objects, functions, and classes at runtime using `inspect`, `getattr`, `setattr`, etc.

---

## 🧑‍💻 Why Learn Metaprogramming?

- Build **custom frameworks and ORMs** (like SQLAlchemy, Django ORM).
- Implement **rule engines** and **DSLs (Domain-Specific Languages)**.
- Write **cleaner abstractions** and **reduce boilerplate**.
- Enable **dynamic behavior** (plugins, decorators, logging, validation, etc.).

---

## ❓ Common Questions

### 🔹 Decorators
1. What are decorators in Python, and how do they work internally?
2. Difference between `@staticmethod`, `@classmethod`, and instance methods?
3. How would you write a decorator that accepts arguments?

### 🔹 Context Managers
4. What problem do context managers solve?
5. How does the `with` statement work internally?
6. Difference between writing a custom context manager with a class vs `contextlib.contextmanager`.

### 🔹 Descriptors
7. What is a descriptor in Python? Explain `__get__`, `__set__`, and `__delete__`.
8. Difference between data and non-data descriptors.
9. How do properties (`@property`) use descriptors internally?

### 🔹 Metaclasses
10. What are metaclasses, and why are they called "classes of classes"?
11. How does `__new__` in a metaclass differ from `__init__`?
12. Real-world use cases of metaclasses (e.g., Django ORM).

### 🔹 AST
13. What is an Abstract Syntax Tree (AST), and how is it used in Python?
14. Example: how would you transform all `+` operations into `-` using AST?
15. How does Python internally use AST during execution?

### 🔹 Reflection / Introspection
16. What’s the difference between reflection and introspection?
17. How would you dynamically get and call a method of a class using `getattr`?
18. Explain the role of the `inspect` module in Python introspection.

---

## 📖 Next Steps

- Try building a **mini ORM** using metaclasses.
- Write a **rule engine** using AST and descriptors.
- Explore how frameworks like **Django, Flask, and SQLAlchemy** leverage metaprogramming.

---

✨ This section builds the foundation for writing **dynamic, flexible, and framework-level Python code**.
