# HTTP Clients – requests, httpx, aiohttp

Https clients let python program send requests and recieve responses
from the web servers. (i.e, REST API)

Commonly used Libarary:
1. `requests` - Synchronous + simple + most popular
2. `httpx` - Modern + (sync+async) + http/2
3. `aiohttp` - async only + High concurrency

They are called HTTP client libraries because:
1. They run on the client side of a client–server model.
2. Their job is to send HTTP requests (GET, POST, PUT, DELETE, etc.) to a server and receive responses.
3. They do not host endpoints like a server framework (e.g., Flask, FastAPI, Django). Instead, they act as the consumer of APIs.


## requests
Easier to use and widely adopted http client libary. 

Best for:
- Script & API Testing
- Automation and Image or CSV or file downloading
- CLI tools
- Low traffic apps
- Integration with services like slack or GITHUB API
- Web Scrapping with small loads

```python
import requests
import json

url = "https://httpbin.org/get"
response = requests.get(url)

# Pretty print JSON response
print(json.dumps(response.json(), indent=4))
```
Output:
```
{
    "args": {},
    "headers": {
        "Accept": "*/*",
        "Accept-Encoding": "gzip, deflate",
        "Host": "httpbin.org",
        "User-Agent": "python-requests/2.25.1",
        "X-Amzn-Trace-Id": "Root=1-68c239a0-256bce4a29df989703182730"
    },
    "origin": "157.50.179.139",
    "url": "https://httpbin.org/get"
}
```

## HTTPX (Sync + Async)
- Drop-in replacement for `request` client library.
- Support sync + async code.
- Built in HTTP/2 support,
- Connection pooling (Reuse the connection for multiple request instead of creating new one each request).

```bash
python -m venv test
source test/bin/activate
pip install httpx
```

```python
import httpx
import asyncio

url = "https://jsonplaceholder.typicode.com/todos/1"

async def main(url):
    async with httpx.AsyncClient() as client:
        response = await client.get(url)
        print(response.json())
asyncio.run(main(url))
```
Output:
```
{'userId': 1, 'id': 1, 'title': 'delectus aut autem', 'completed': False}
```

Usages:
1. Modern API requiring HTTP/2 (eg: gRPC-web, GraphQL apis)
2. Microservices communication (async servers)
3. REST APIs client in production grade apps

## Aihttp (async only)
1. Designed for asyncio. 
2. Great for many parallel requests.
3. Also supports `websockets` for realtime apps.

```bash
python -m venv test
source test/bin/activate
pip install aiohttp
```


```python
import aiohttp, asyncio

url = "https://jsonplaceholder.typicode.com/todos/1"

async def main(url):
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as resp:
            data = await resp.json()
            print(data)
asyncio.run(main(url))
```
Output:
```
{'userId': 1, 'id': 1, 'title': 'delectus aut autem', 'completed': False}
```

Usecases:
1. High-concurrency web scraping
2. Chat applications (via WebSockets)
3. Real-time dashboards (fetching multiple APIs concurrently)
4. Backend services making 1000s of requests concurrently
