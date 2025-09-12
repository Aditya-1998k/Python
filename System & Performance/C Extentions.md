Python is slow for heavy computation due to GIL and interpreted nature.
`Cython` and `Numba` let you run **c-speed** code while writing mostly in python.

### 1. Cython
1. Write python-like syntax, add static types and compile to C.
2. Best for tight loops, numeric code and algorithms.
3. Requires a .pyx file + compilation steps.

file_name : **mymodule.pyx**
```cython
def sum_loop(int n):
    cdef int i, total = 0
    for i in range(n):
        total += i
    return total
```

```bash
pip install cython
```

Now compile the file with cython
```bash
(test) gaditya@lptl-gaditya:~/instances/projects$ cythonize -i mymodule.pyx
Compiling /home/gaditya/instances/projects/mymodule.pyx because it changed.
[1/1] Cythonizing /home/gaditya/instances/projects/mymodule.pyx
```

file_name: **benchmark.py**
```python
import time

def sum_loop_py(n):
    total = 0
    for i in range(n):
        total += i
    return total


if __name__ == "__main__":
    N = 10_000_000
    
    start = time.time()
    result = sum_loop_py(N)
    end = time.time()
    
    print(f"Python result: {result}")
    print(f"Python time: {end - start:.4f} sec")

    start = time.time()
    result = mymodule.sum_loop(N)
    end = time.time()
    
    print(f"Cython result: {result}")
    print(f"Cython time: {end - start:.4f} sec")
```
Output:
```bash
Python result: 49999995000000
Python time: 0.8223 sec
Cython result: 49999995000000
Cython time: 0.0049 sec
```

### 2. Numba
Just add a decorator `@jit` or `@njit` and JIT compiles python functions
into optimized machine code.

Works best with Numpy array and math-heavy code.
No compilation step required.

`pip install numba`

```python
from numba import njit
import numpy as np
import time

@njit
def fast_num_numba(arr):
    total = 0
    for x in arr:
        total += x
    return total


def sum_loop_py(arr):
    total = 0
    for i in arr:
        total += i
    return total


if __name__ == "__main__":
    N = 10_000_000
    arr = np.arange(N)
    
    start = time.time()
    result = sum_loop_py(arr)
    end = time.time()
    
    print(f"Python result: {result}")
    print(f"Python time: {end - start:.4f} sec")

    start = time.time()
    result = mymodule.fast_num_numba(arr)
    end = time.time()
    
    print(f"Numba result: {result}")
    print(f"Numba time: {end - start:.4f} sec")
```
