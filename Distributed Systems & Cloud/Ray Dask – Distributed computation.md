## Ray/Dask – Distributed computation

Both `Ray` and `Dask` are python frameworks that lets you scale your computation from a laptop to cluster.
So we can do distributed computation on different cluster. This will help in GIL mitigation by distributing
task across multiple cores and nodes.

### 1. Ray
This is a General purpose framework for a distributed Execution (tasks, actors, ML Training etc)
- Core Idea : **Remote function and actors**
- Started at RISELab
- Integrate with various Machine Learning library (pytorch, Tensorflow, HF, XGBoosts)

Install Ray: `pip install "ray[default]`

```python
def square(x):
    return x * x

results = [square(i) for i in range(5)]
print(results)  # [0, 1, 4, 9, 16]
```

The above defined function is a normal function which runs sequentially (one by One).
But with **Ray**, we can run the same code parallel across multiple cores or even
mulitple machines.

**Turning Functions into Remote Tasks**
```python
import time
import ray

# Starting Ray runtime on local machine
ray.init()

@ray.remote
def squere(x):
    time.sleep(2)
    print(f"Processing {x}..")
    return x*x

# Call remote function (returns a "future", not the actual result yet)
start = time.time()
futures = [square.remote(i) for i in range(5)]

# Get results (Ray will run them in parallel)
results = ray.get(futures)
end = time.time()
print("Results:", results)
print(f"Total time taken {end - start} secs")
```
<img width="1352" height="144" alt="image" src="https://github.com/user-attachments/assets/12ff72b6-1a9f-4c30-a0b0-62130991f425" />


What happens here:
- `@ray.remote` = tells Ray that this function can be executed remotely (on a worker).
- `square.remote(2)` = submits the task to Ray, it doesn’t execute immediately.
- Instead, it returns a future object (a placeholder).
- `ray.get(futures)` = waits for all tasks to finish and collects results.
- And Even though i added 2 sec sleep time, total execution time was around 2.14 sec which way less than 10 sec for sequential execution

### 2. Dask
Focused on big data/ parallel collections.
Provides familiar API
- Dask Dataframe
- Dask Array
- Dask Bag
Lazily builds a DAG (Directed acyclic graph) of tasks.

It executes in parallel on:
1. Threads
2. Clusters
3. Processors

**Without Dask with Numpy:**
```python
import numpy as np

# Create a 10000 x 10000 array in memory
x = np.random.random((10000, 10000))

# Define computation
y = (x + x.T) - x.mean(axis=0)

# Here everything is computed eagerly (right away)
print(y.shape)  # (10000, 10000)
```
Issues:
1. NumPy will immediately allocate `~800 MB` of memory for x.
2. `Operations (+, transpose, mean, etc.)` are done eagerly, so intermediate arrays are also materialized → big memory spike.
3. `Single-threaded` (unless NumPy is linked with MKL/OpenBLAS).
4. Doesn’t scale to `multiple cores or machines`.

**With Dask (Lazy, Parallel, Distributed)**
`sudo apt install graphviz`

```python
import dask.array as da
import webbrowser

# Example Dask computation
x = da.random.random((10000, 10000), chunks=(1000, 1000))
y = x + x.T

# Save the visualization
filename = "task-graph.svg"
y.visualize(filename=filename)

# Compute
result = y.compute()
print("Result shape:", result.shape)

# Open in default browser
webbrowser.open(filename)
```
what's happening:
1. `da.random.random((10000, 10000), chunks=(1000, 1000))`
Instead of creating one giant 10000×10000 array (~800 MB),
Dask splits it into chunks of 1000×1000.
Think of it like 100 blocks by 100 blocks:
```
[Block(0,0)] [Block(0,1)] [Block(0,2)] ...
[Block(1,0)] [Block(1,1)] [Block(1,2)] ...
```
Each block can be computed independently, even on different cores/machines.
At this point, no actual data is created — just a lazy placeholder.

2. `y = (x + x.T) - x.mean(axis=0)`
- Dask builds a task graph:
- Compute x blocks.
- Transpose them (block-wise).
- Add them.
- Compute mean across axis 0.
- Subtract from sum.
- Still no data is generated. y is just a graph of operations.

3. `y.compute()`
Now Dask executes the graph:
- Each block operation is assigned to a worker (thread/process/cluster node).
- Tasks are executed in parallel.
- Intermediate results are streamed (not all loaded into memory at once).
- Finally, result is a NumPy array with the actual values.

ASCII Diagram
```
Task Graph for y = (x + x.T) - x.mean(axis=0)

   ┌─────────────┐
   │  Block(0,0) │───┐
   └─────────────┘   │
   ┌─────────────┐   │   ┌─────────────┐
   │  Block(0,1) │───┼──▶│   Add op    │───▶ ...
   └─────────────┘   │   └─────────────┘
        ...          │
   ┌─────────────┐   │
   │  Block(1,0) │───┘
   └─────────────┘

   + mean(axis=0) calculation (parallel)

   + subtraction (parallel)

   ↓

 Final result: NumPy array (10000 x 10000)
```
