## NumPy / Pandas Internals – Vectorization, broadcasting
Vectorization means writing the code that operates on entire arrays/vectors
at once, instead of looping through elements one by one in python.

It uses low-level optimized C/Fortran loops under the hood (in NumPy, Pandas, etc.), so it’s both:
- Cleaner (fewer loops in your code)
- Faster (avoids Python’s interpreter overhead)

**With and Without Numpy Vectorization**
```python
import numpy as np
import time

# Python loop
data = list(range(1_000_000))
start = time.time()
result_py = [x**2 for x in data]   # slow, pure Python loop
end = time.time()
print(f"Total Time: {end - start}")
 
# NumPy vectorized
arr = np.arange(1_000_000)
start = time.time()
result_np = arr ** 2               # fast, uses C loops
end = time.time()
print(f"Total time: {end-start}")
```

`Vectorization` means treating data as whole vectors/matrices, not as individual elements.
