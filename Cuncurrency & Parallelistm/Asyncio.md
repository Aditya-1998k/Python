# Asyncio (async/await) – Cooperative Multitasking
`asyncio` is python built-in framework for asynchronous programming.
Provides single threaded, single process concurrency using `event loop.`
Based on coroutines (`async def` function)

Good For:
- Ideal for I/O bound tasks (network requests, DB queries, file IO) - Avoid Blocking
- Use cooperative multitasking : tasks voluntarily give up control via `await`.

**Key Concept:**
1. `Coroutine`: A special function defined with `async def`
2. `Event Loop`: Schedular that runs coroutines concurrently
3. `await`: Suspends coroutines untill awaited task completes
4. `Task`: Wrapper around coroutine, scheduled by event loop.
5. `asyncio.gather()` : Run multiple coroutines concurrently.
6. `asyncio.run()` : Entry point to run async code.

#### Basic Example
```python
import asyncio

async def say_hello():
    print("Hello...")
    await asyncio.sleep(5)   # non-blocking sleep
    print("World!")

asyncio.run(say_hello())
print("Hello Outside coroutines")
```
Output:
```
Hello...
World!
Hello Outside coroutines
```
Only say_hello is registered.
At await asyncio.sleep(5), the event loop has nothing else scheduled, so it just idles.
`Hello Outside coroutines` is outside the event loop, so it only runs after the loop finishes.

#### Running Multiple Task (Using asyncio.gather)
```python
import asyncio

async def task(name, delay):
    print(f"{name} started")
    await asyncio.sleep(delay)     # non-blocking, lets event loop run other tasks
    print(f"{name} finished after {delay}s")
    return f"{name} result"

async def main():
    results = await asyncio.gather(
        task("Task1", 2),
        task("Task2", 1),
        task("Task3", 3)
    )
    print("Results:", results)

asyncio.run(main())
# Starts the event loop.
# Runs main() until completion.
```

Defines an async coroutine task.
Each task simulates some I/O by sleeping for delay seconds.
`await asyncio.sleep(delay)` does not block → it pauses this coroutine and allows other tasks to run on the event loop.

`asyncio.gather()` runs all provided tasks concurrently.
Each `task(...)` is a coroutine, and `gather` schedules them on the event loop.
`await asyncio.gather(...)` waits until all tasks finish and returns a list of results in the same order as passed.

Output:
```
Task1 started
Task2 started
Task3 started
Task2 finished after 1s
Task1 finished after 2s
Task3 finished after 3s
Results: ['Task1 result', 'Task2 result', 'Task3 result']
```

#### Creating Tasks Explicitly
```python
import asyncio

async def worker(n):
    print(f"Worker {n} started")
    await asyncio.sleep(n)
    print(f"Worker {n} finished")

async def main():
    tasks = [asyncio.create_task(worker(i)) for i in [1,3,2]]
    await asyncio.gather(*tasks)

asyncio.run(main())
```
Output:
```
Worker 1 started
Worker 3 started
Worker 2 started
Worker 1 finished
Worker 2 finished
Worker 3 finished
```

#### Awaiting as They Complete
```python
import asyncio

async def download(n):
    await asyncio.sleep(n)
    return f"Downloaded {n}"

async def main():
    tasks = [asyncio.create_task(download(i)) for i in [3, 1, 2]]
    for task in asyncio.as_completed(tasks):
        result = await task
        print(result)

asyncio.run(main())
```
Output:
```
Downloaded 1
Downloaded 2
Downloaded 3
```

