# ⚡ System & Performance in Python

System-level programming and performance optimization are crucial for building **efficient, scalable, and high-performance Python applications**.  
This section covers techniques for managing memory, leveraging concurrency, profiling bottlenecks, and extending Python with C for speed.

---

## 📂 Topics Covered

1. **C Extensions** – Speed up Python with native C code.
2. **Memory Management** – Understand Python’s memory model, GC, reference counting, and optimizations.
3. **Multiprocessing & Shared Memory** – Parallel processing with `multiprocessing` and memory sharing for large data.
4. **Multithreaded I/O** – Handling concurrent I/O tasks efficiently using threads.
5. **Profiling** – Identifying performance bottlenecks with `cProfile`, `timeit`, and other tools.

---

## 🧑‍💻 Why Learn System & Performance?

- Optimize **CPU-bound** and **I/O-bound** workloads.
- Improve **latency and throughput** in real-world apps.
- Handle **large datasets** without excessive memory usage.
- Extend Python with **low-level C/C++ for speed**.
- Profile and eliminate bottlenecks in production.

---

## ❓ Common Interview Questions

### 🔹 C Extensions
1. How do C extensions improve Python performance?
2. Difference between Cython, ctypes, and CPython API?
3. When would you choose to write C extensions?

### 🔹 Memory Management
4. How does Python manage memory internally?
5. What is reference counting, and how does it lead to memory leaks?
6. Explain the role of Python’s garbage collector (GC).

### 🔹 Multiprocessing & Shared Memory
7. Difference between multithreading and multiprocessing in Python?
8. How does `multiprocessing.shared_memory` help with large data?
9. Why does Python’s GIL affect threading but not multiprocessing?

### 🔹 Multithreaded I/O
10. Why is multithreading useful for I/O-bound but not CPU-bound tasks?
11. Difference between synchronous, multithreaded, and async I/O?
12. Example: How would you implement a multithreaded file downloader?

### 🔹 Profiling
13. How do you profile a Python program?
14. Difference between `timeit` and `cProfile`?
15. How do you identify and optimize hot paths in Python code?

---

## 📖 Next Steps

- Write a **C extension** for a math-heavy function.
- Experiment with **memory profiling** tools like `tracemalloc`.
- Build a **parallel data processing pipeline** with multiprocessing.
- Compare **multithreading vs asyncio** for I/O workloads.
- Use **profiling** to tune a slow-running Python project.

---

✨ Mastering system-level performance makes your Python applications **faster, more efficient, and production-ready**.
