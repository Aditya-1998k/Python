# System & Performance in Python  

This section focuses on low-level optimizations, memory management, multithreading I/O, and tools for profiling and extending Python for better performance.  

## Topics Covered  
- **C Extensions:** Extending Python with C for performance-critical code.  
- **Memory Management:** Understanding Python’s memory model, reference counting, and garbage collection.  
- **Multiprocessing Shared Memory:** Sharing data efficiently between processes without serialization overhead.  
- **Multithreaded I/O:** Handling concurrent I/O-bound tasks with threads.  
- **Profiling:** Measuring execution time, memory usage, and performance bottlenecks.  

---

## 📌 Questions  

### C Extensions  
1. Why would you write a C extension for Python instead of using pure Python?  
2. How do Python’s C extensions interact with the Global Interpreter Lock (GIL)?  

### Memory Management  
3. How does Python manage memory internally (reference counting & garbage collection)?  
4. What’s the difference between shallow copy and deep copy in Python memory handling?  
5. How would you reduce memory leaks in a long-running Python process?  

### Multiprocessing Shared Memory  
6. How does `multiprocessing.shared_memory` improve performance over `multiprocessing.Queue` or `Pipe`?  
7. What are some potential pitfalls when using shared memory across processes?  

### Multithreaded I/O  
8. Why is multithreaded I/O more efficient than multiprocessing for network-bound tasks?  
9. How does Python’s GIL affect multithreaded I/O performance?  

### Profiling  
10. What tools can you use to profile CPU and memory usage in Python, and when would you use them?  

---

👉 This section helps you **optimize Python programs** by diving deeper into **system-level features** and performance tuning.  
