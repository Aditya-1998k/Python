Normally processes do not share memory (each has it's own address space).
To shared data between the process you need IPC (Inter-process communiciation) like pipes, queues or sockets.

1. Shared memory let's multiple processes directly access the same block of memory - faster than pickling/unpickling
2. Introduced in python 3.8 via `multiprocessing.shared_memory`

**Key benefits:**
1. SharedMemory object → Allocates a memory block accessible across processes.
2. np.ndarray interface → You can wrap shared memory in NumPy arrays for efficient numeric work.
3. No copy overhead → Data is read/written directly.
4. Need cleanup → Must call .close() and .unlink().

```python
from multiprocessing import Process, shared_memory
import numpy as np

def worker(name):
    # Attach to existing shared memory by name
    shm = shared_memory.SharedMemory(name=name)
    arr = np.ndarray((5,), dtype=np.int64, buffer=shm.buf)
    arr[0] += 100   # modify shared memory
    shm.close()

if __name__ == "__main__":
    # Create shared memory block
    shm = shared_memory.SharedMemory(create=True, size=5 * np.int64().nbytes)
    arr = np.ndarray((5,), dtype=np.int64, buffer=shm.buf)
    arr[:] = [1, 2, 3, 4, 5]  # initialize

    print("Before worker:", arr)

    # Start worker process
    p = Process(target=worker, args=(shm.name,))
    p.start()
    p.join()

    print("After worker:", arr)

    # Cleanup
    shm.close()
    shm.unlink()
```
Output:
```
Before worker: [1 2 3 4 5]
After worker:  [101   2   3   4   5]
```
This shows processes directly modifying shared data without sending copies.
