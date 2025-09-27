Both `Ray` and `Dask` are python frameworks that lets you scale your computation from a laptop to cluster.
So we can do distributed computation on different cluster. This will help in GIL mitigation by distributing
task across multiple cores and nodes.

### Ray
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
- And Even though i added 2 sec sleep time, total execution time was around 2.5sec which way less than 10 sec for sequential execution


