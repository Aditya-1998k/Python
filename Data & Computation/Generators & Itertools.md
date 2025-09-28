### Generators in Python
A lazy producer of values in python. Instead of holding
everything in memory, it yields only one item at one time.

Usefule when:
- Large Dataset
- Streams
- Infinite Sequence


Eager versus lazy refers to the timing of resource loading or computation. 
Eager loading or execution fetches all data or performs all computations immediately, with the initial request. 
In contrast, lazy loading or execution defers the fetching or computation of data or resources until 
they are actually needed, resulting in faster initial loads but potentially more complex handling. 
The choice between the two depends on whether all related data is always required (eager)
or if it's more efficient to load it only when accessed (lazy). 

**Normal List vs Generator**
```python
# Normal list (stores everything in memory)
nums = [i*i for i in range(5)]
print(nums)   # [0, 1, 4, 9, 16]

# Generator (lazy, yields one by one)
nums_gen = (i*i for i in range(5))
print(nums_gen)        # <generator object ...>
print(list(nums_gen))  # [0, 1, 4, 9, 16
```
Generator doesn’t compute until you iterate over it.

**Infinite Sequence**
```python
def infinite_counter(start=0):
    while True:
        yield start
        start += 1

counter = infinite_counter()
print(next(counter))  # 0
print(next(counter))  # 1
print(next(counter))  # 2
```
Can handle infinite data without crashing.

### Itertools (The standard library for iterators)
```
itertools.count() → infinite counter.
itertools.cycle() → repeat items forever.
itertools.chain() → join multiple iterables.
itertools.islice() → slice generators (like lists).
itertools.combinations/permutations → useful in interviews.
```

Example:
```python
import itertools

# Cycle through pattern
pattern = itertools.cycle(["red", "green", "blue"])
for _ in range(6):
    print(next(pattern))

# Generate combinations
letters = ["A", "B", "C"]
for combo in itertools.combinations(letters, 2):
    print(combo)
```
Output:
```
red, green, blue, red, green, blue
('A', 'B'), ('A', 'C'), ('B', 'C')
```

### Why Generators & Itertools Matter?
1. Efficient memory Usages
2. Foundation for pipelines and coroutines
3. How would you process 10GB of data without loading all at once?
4. What's the difference between return and yeild?

```python
def generator_function():
    yield 1
    yield 2
    yield 3

gen = generator_function()
print(next(gen))  # 1
print(next(gen))  # 2
print(next(gen))  # 3
```
- Turns the function into a generator.
- Doesn’t end immediately — it pauses and can resume later.
- Produces values one by one (lazy evaluation).
- return → one-shot value, function ends.
- yield → produces a sequence lazily, function state is saved between calls.

### Processing 10GB File
List:
```python
lines = open("big.txt").readlines()
```
Memory:
```
+-------------------------+
|         Memory          |
+-------------------------+
| [Line1, Line2, ...,     |
|  LineN]  (10GB loaded!) |
+-------------------------+
```
Entire file is loaded into RAM at once → memory explosion 💥

Using generator (lazy, one item at a time):
```python
def read_file(path):
    with open(path) as f:
        for line in f:
            yield line
```
Output:
```
+-------------------------+
|         Memory          |
+-------------------------+
| Line1 (process, discard)|
| Line2 (process, discard)|
| Line3 (process, discard)|
| ...                     |
+-------------------------+
```
At any moment, only one line is in memory → works even for 10GB / 100GB files.
