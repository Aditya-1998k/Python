### What is Threading?
A thread is a light weight unit of execution inside a **process**. Thread shares
same memory space (unlike processes).

**Best for I/O Bound Task:**
- Network Calls
- File I/O
- API Requests

**Inefficient for CPU Bound Task** Because of Python GIL (Global interpreter Lock)

Basic Example:
```python
import threading
import time

def worker(name):
    print(f"Thread {name} starting")
    time.sleep(2)
    print(f"Thread {name} Ending")

# Creating Thread
t = threading.Thread(target=worker, args=("A",))
t.start()

# Main Thread continue
print("Main thread running...")

# Wait for thread to finish (Optional if you want main thread to wait)
t.join()

print("Main thraed finish")
```

Note: t.join() tells the main program: "Wait until this thread finishes before continuing."
```
Execution flow:
Thread A starting
Main thread is running...
(Main waits until Thread A finishes)
Thread A finished
Main thread finished
```
without t.join() the main thread does not wait for the worker thread. but it also doesn’t exit until non-daemon threads are done.
```
Execution flow:
Thread A starting
Main thread running...
Main thraed finish
Thread A Ending
```

### Daemon vs Non-Daemon Threads
1. **Non-Daemon Thread** :
Default thread in python. The program wait for them to finish before exiting.
Even if main thread ends, python won't shutdown untill all non-daemon are done.

```python
import threading, time

def worker():
    time.sleep(3)
    print("Worker finished")

# Non-daemon (default)
t = threading.Thread(target=worker)
t.start()

print("Main finished")
```
Output:
```
Main finished
Worker finished
```
The program didn’t exit immediately — it waited 3 seconds for the worker thread.

2. **Daemon Thread**: 
Background thread that kills automatically when main program exits. Program does not weight for
daemon thread to finish. Useful for background task like logging, monitoring or cleanup that does not
need to block program exits.

```python
import threading, time

def log_worker():
    time.sleep(3)
    print("Worker finished")

# Daemon thread
t = threading.Thread(target=log_worker, daemon=True)
t.start()

print("Main finished")
```
Output:
```
Main finished
```
The worker never prints Worker finished — because the program ended before it could finish.

Note: Think of non-daemon as “important work must finish” and daemon as “background helper, stop if main ends.”
