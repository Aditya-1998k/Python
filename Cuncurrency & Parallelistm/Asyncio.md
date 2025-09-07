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


