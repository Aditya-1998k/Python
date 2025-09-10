# XML-RPC (Python)
XML Remote Procedure Call. It is a protocol that allows a program
on one machine (client) to call functions on another machine(server) as
if they were local functions.

- HTTP/HTTPS == For transport
- XML == Encoding request/response

Older compared to JSON-RPC, REST or gRPC, but still used in legacy system.

### How It works
1. Client send as HTTP POST request to the server Endpoint
2. The request body is an XML document containing
- Method name
- parameters
3. Server parses the XML, executes the method, and return an XML response.

**Example Request**
```
<methodCall>
  <methodName>add</methodName>
  <params>
    <param><value><int>5</int></value></param>
    <param><value><int>3</int></value></param>
  </params>
</methodCall>
```

Here method name is `add` and param value are `5` and `3`.

**Example Response**
```
<methodResponse>
  <params>
    <param><value><int>8</int></value></param>
  </params>
</methodResponse>
```

Python has built-in libary for xmlrpc
- server : `xmlrpc.server.SimpleXMLRPCServer` (in zope app not needed as endpoint already exposed)
- client : `xmlrpc.client.ServerProxy`

Example:

**XML-RPC Server (server.py)**
```python
from xmlrpc.server import SimpleXMLRPCServer

def greet(name):
    return f"Hello {name}. This the XML-RPC server response for your xmlrpc call."

host = "localhost"
port = 8000
server = SimpleXMLRPCServer((host, port))

print(f"XML RPC server running on {host}:{port}")

# Register functions
server.register_function(greet, "greet")

# Run server forever
server.serve_forever()
```

**XML-RPC (client.py)**
```python
import xmlrpc.client as client

# Create connection with server
host = "localhost"
port = 8000
conn = client.ServerProxy(f"http://{host}:{port}")

print(conn.greet("Aditya"))
```

Now, first run the xmlrpc server and then client.py

Output:
```
Hello Aditya. This the XML-RPC server response for your xmlrpc call.
```

**ASCII Diagram**
```python
Client (xmlrpc.client)                            Server (SimpleXMLRPCServer)
----------------------------------------------------------------
server.greet("Aditya")  ----HTTP POST--------> <methodCall><methodName>greet</methodName>...
                                               Execute greet("Aditya") → return Response
         <---HTTP Response--- -----------------<methodResponse>Hello Aditya</methodResponse>
```

### Advantages
1. Very simple to implement.
2. Cross-language support (clients exist in Python, Java, PHP, etc.).
3. No need to design REST APIs or schemas.

### Disadvantages
1. Uses XML → verbose and slow compared to JSON/Protobuf.
2. No support for streaming or advanced features (like gRPC).
3. Mostly considered legacy, rarely used in modern systems.

