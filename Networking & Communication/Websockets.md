# WebSockets
1. Full Duplex (Both sides can send and recieve at the same time, eg: Phonecall, whatsapp etc) over single TCP connections
2. Unlike HTTP (request/response), websockets allow `server --> client` push.
3. Used in chat apps, live dashboards, games, notifications.

**HTTP vs WebSockets**

HTTP
```
Client -------------- Request --------------> Server
Client <------------- Response ------------- Server
[Connection closed]
```

WebSockets
```
Client        ⇆⇆⇆⇆⇆⇆⇆⇆⇆⇆⇆⇆⇆⇆⇆⇆         Server
      (messages back & forth anytime)
```

Python Libarary for WebSockets:
1. websockets - Popular asycio based websocket libarary
2. FastAPI / Starlette  - Fast API native support
3. Socket.IO (python-socketio) - High level abstraction (rooms, events)

**websockets**

```bash
python -m venv test
source test/bin/activate
pip install websockets
```

**server.py**
```python
import asyncio
import websockets

async def echo(websockets):
    async for message in websockets:
        print(f"Recieved message: {message}")
        await websockets.send(f"Echo: Hey, Recieved your message. Thanks")

async def main():
    async with websockets.serve(echo, "localhost", 8765):
        print("WebSocket server running on ws://localhost:8765")
        await asyncio.Future() # Run forever

asyncio.run(main())
```

**client.py**
```python
import asyncio
import websockets

async def hello():
    async with websockets.connect("ws://localhost:8765") as websocket:
        await websocket.send("Hello Websockets !")
        reply = await websocket.recv()
        print(f"Server replied: {reply}")

asyncio.run(hello())
```
Output:

server.py
```
WebSocket server running on ws://localhost:8765
Recieved message: Hello Websockets !
```
client.py
```
Server replied: Echo: Hey, Recieved your message. Thanks
```

Note:
1. Uses ws:// or wss:// (secure WebSockets).
2. Connection stays open until explicitly closed.
3. Ideal for real-time bidirectional updates.
