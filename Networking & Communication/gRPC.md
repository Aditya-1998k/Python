A high performance RPC framework developed by Google.
Uses Protocol Buffers (protobuf) as its Interface Definition Language (IDL).
Support Multi-language (Python, Go, C++, Java etc). Built on `HTTP/2` which is
efficient, multiplexed and support streaming.

**Communication type:**
1. Unary RPC → Single request, single response
2. Server streaming → Client sends one request, server streams multiple responses
3. Client streaming → Client streams multiple requests, server sends one response
4. Bidirectional streaming → Both send streams


**gRPC Flow**
1. Define service and message type in .proto file
2. Use Protoc compiler to generate Python client/server stubs.
3. Implement server logic (using gRPC Python API)
4. Client call server method as if it's local.

Installation Steps:
```
python -m venv grpc
source grpc/bin/activate

pip install grpcio grpcio-tools

grpcio == Core gRPC runtime for Python (needed to run client & server).
grpcio-tools == Provides the protoc compiler plugin to generate Python code from .proto files.
```
**Step 1: Create a file helloword.proto**
```proto
syntax = "proto3";

service Greeter {
  rpc SayHello (HelloRequest) returns (HelloReply);
}

message HelloRequest {
  string name = 1;
}

message HelloReply {
  string message = 1;
}
```

**Step 2: Run this file**
```
python -m grpc_tools.protoc -I. --python_out=. --grpc_python_out=. helloworld.proto
```
This generates:
- helloworld_pb2.py
- helloworld_pb2_grpc.py

These two files are the heart of gRPC in Python, and they’re generated automatically from your .proto
- `helloworld_pb2.py` → Contains Python classes for the protobuf messages (HelloRequest, HelloReply) that handle serialization/deserialization.
- `helloworld_pb2_grpc.py` → Contains gRPC service definitions (stubs for clients and base classes for servers) that let you implement and call RPC methods.

**Step 3: Write your server.py and client.py**

**server.py**
```python
import grpc
from concurrent.futures import ThreadPoolExecutor
import helloworld_pb2
import helloworld_pb2_grpc

class Greeter(helloworld_pb2_grpc.GreeterServicer):
    def SayHello(self, request, context):
        return helloworld_pb2.HelloReply(message=f"Hello, {request.name}")

def serve():
    server = grpc.server(futures.ThreadPoolExecutor(max_workers=5))
    helloworld_pb2_grpc.add_GreeterServicer_to_server(Greeter(), server)
    server.add_insecure_port('[::]:50051')
    server.start()
    print("gRPC server started on port 50051")
    server.wait_for_termination()

if __name__ == "__main__":
    serve()
```

**client.py**
```python
import grpc
import helloworld_pb2
import helloworld_pb2_grpc

def run():
    channel = grpc.insecure_channel("localhost:50051")
    stub = helloworld_pb2_grpc.GreeterStub(channel)
    response = stub.SayHello(helloworld_pb2.HelloRequest(name="Aditya"))
    print("Server replied:", response.message)

if __name__ == "__main__":
    run()
```

Output:
```
Server replied: Hello, Aditya
```

**Usages:**

Where is gRPC used?
1. Microservices communication (internal APIs).
2. Real-time systems (chat, video streaming).
3. Cloud-native apps (Kubernetes, service meshes like Istio).
