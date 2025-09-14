## Scaling Python for Performance (40x Faster Image Processing)

Core Problem:
1. Real-world context: semiconductor manufacturing image segmentation → convert irregular noisy regions into precise, non-overlapping rectangles.
2. Naive solution with OpenCV (cv2) + NumPy: worked but slow, memory heavy, single-threaded.

**Challenges**
1. GIL (Global Interpreter Lock): blocks true parallel execution in threads for CPU-bound tasks.
2. Recursion limits: deep recursive processing can hit RecursionError.
3. Scaling issues: naive code doesn’t use multi-core CPUs efficiently.

**Iterative Optimizations**
1. Naive Approach
2. cv2 + NumPy, single process.
1. Simple but slow and non-scalable.

Suppose one image where i identified yellow dot, which is irrelevant,
so i can remove those.
```
┌──────────────┐
│ Image Rect   │
│ (dots inside)│
└───────┬──────┘
        │
        ▼
 Remove Yellow Dots
        │
        ▼
 Remove Yellow Dots
        │
        ▼
 Remove Yellow Dots
 (repeated sequentially...)
```


**Multiprocessing**
1. Used multiprocessing to spread tasks across cores.
2. Speed ↑, but memory usage ↑ too.
3. Manager-based Sharing
- multiprocessing.Manager allowed shared state → less memory duplication.
- Tradeoff: slower IPC (inter-process communication).

```
                ┌────────────────────────┐
                │        Image Rect      │
                └────────┬───────────────┘
                         │ Split into chunks
   ┌───────────────┬──────────--─────-----──────
   ▼               ▼           ▼                ▼
[Process 1]   [Process 2]  [Process 3]      [Process 4]
 Remove        Remove       Remove            Remove
 Yellow Dots   Yellow Dots Yellow Dots     Yellow Dots
   │               │          │              │
   └───────┬───────┴──────────┴───────┬─---──┘
           ▼                          ▼
       Combined Cleaned Image (white dots only)

```

**Producer–Consumer Model**
1. Used Queue to dynamically distribute work.
2. Good load balancing, scales well.
3. More complex synchronization logic.

**ProcessPoolExecutor**
1. From concurrent.futures.
2. Cleaner syntax, automatic pooling, high CPU utilization.
3. Simplifies code compared to raw multiprocessing.

**Numba JIT**
1. @jit compilation for compute-heavy loops.
2. 5x–10x faster on numerical tasks.
3. Easy to apply, but limited scope (numeric functions).

**Key Lessons**
1. Threads ≠ performance for CPU-bound work → prefer multiprocessing or ProcessPoolExecutor.
2. Engineering design > library choice: architecture (queues, pooling, memory sharing) drives scalability.
3. Numba can unlock near-C speeds with minimal code changes.
4. Profiling is essential → bottlenecks vary with dataset/architecture.
5. 40x throughput gain achieved with smart design.

ProcessPoolExecutor
```python
from concurrent.futures import ProcessPoolExecutor
import numpy as np

def heavy_task(arr):
    # Example: sum of squares (could be segmentation logic)
    return np.sum(arr ** 2)

if __name__ == "__main__":
    data_chunks = [np.random.rand(1000, 1000) for _ in range(8)]

    with ProcessPoolExecutor() as executor:
        results = list(executor.map(heavy_task, data_chunks))

    print("Results:", results)
```
