## Finding a Needle in the Haystack – Debugging Performance Issues

Why Performance Debugging Feels Like a “Needle in a Haystack”
1. Severe performance incidents = urgent debugging.
2. Challenge: finding the true bottleneck (CPU, memory, GC, I/O, etc.).
3. Need a systematic approach instead of guesswork.

### 1. Performance Profiler

Types of Profilers
1. **Deterministic (Tracing) Profilers**
   - Record every function call, return, and exception.
   - Example: cProfile (built-in).
   - Higher overhead, but accurate.
Example:
```python
import cProfile
import pstats

def slow_function():
    sum([i**2 for i in range(10_000)])

cProfile.run("slow_function()", "profile.out")
p = pstats.Stats("profile.out")
p.sort_stats("cumulative").print_stats(10)
```

2. **Statistical (Sampling) Profilers**
- Take periodic snapshots of the stack.
- Much lower overhead.
- Good for production profiling.

**Tools:**
- `py-spy` (runs outside the process, no code changes).
- `Pyinstrument` (Python-level sampling profiler).
- `Austin` (lightweight sampling profiler).

Example with Pyinstrument:
```bash
pip install pyinstrument
pyinstrument myscript.py
```
**Note**: You can explore `py-spy`, `Pyinstrument` and `Austin`

### 2. Memory Profiling
Focus on memory footprint
- prevent OOM (Out of Memory) issues
- Reduce cloud costs.

Tools:
- `memory_profiler` / `mprof` (line-by-line memory usage).
- `tracemalloc` (built-in, tracks memory allocations, snapshots + diffs).
- `memray` (powerful allocator profiler).

Example with memory_profiler:
```python
from memory_profiler import profile

@profile
def allocate():
    x = [i for i in range(10**6)]
    return x

allocate()
```
Run with:
```bash
mprof run myscript.py
mprof plot
```

Example with tracemalloc:
```python
import tracemalloc

tracemalloc.start()

# Code to profile
x = [i for i in range(10**6)]
y = [str(i) for i in range(10**6)]

snapshot = tracemalloc.take_snapshot()
for stat in snapshot.statistics("lineno")[:5]:
    print(stat)
```

### 3. Garbage Collection (GC) in Python
Python uses generational garbage collection:
- `gen0` : young objects (collected often).
- `gen1` : survive 1 cycle.
- `gen2` : survive many cycles.

Cyclic references: gen0 cannot clean them (needs higher gen collection).

You can tune thresholds to trigger GC.

✅ Example:

import gc

print(gc.get_threshold())   # default thresholds (700, 10, 10)

# Set custom threshold (trigger collection earlier/later)
gc.set_threshold(500, 5, 5)

# Force collection
gc.collect()

4. Flamegraphs

Visualization of where CPU time is spent.

py-spy and memray can generate flamegraphs.

✅ Example with py-spy:

py-spy record -o profile.svg -- python myscript.py


Then open profile.svg in a browser.

5. Case Study (Key Lessons)

Combine CPU + memory profiling for a holistic view.

Start with sampling profiler in prod (low overhead).

Use deterministic profiling locally for detail.

Use snapshots & diffs (tracemalloc) to find memory leaks.

Tune GC thresholds to optimize collection and avoid pauses.

6. Key Takeaways

🔍 Use the right profiler for the problem:

cProfile → function-level bottlenecks.

py-spy/Pyinstrument → low-overhead, sampling in prod.

memory_profiler/tracemalloc/memray → memory leaks & footprint.

⚡ Systematic debugging:

Detect bottleneck (CPU vs Memory).

Profile with the right tool.

Interpret flamegraphs / stats.

Fix & validate.

🧹 Don’t forget GC tuning — sometimes overhead is due to garbage collection, not your code.
