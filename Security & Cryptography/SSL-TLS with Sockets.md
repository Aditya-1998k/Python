## SSL/TLS with Sockets – Encrypted Communication
SSL/TLS (Secure Socket Layer/ Transport Layer Security) provides encrypted communication over insecure networks.

This ensures:
1. `Confidentiality` : Data is encrypted
2. `Integrity` : Data is not tampered.
3. `Authentication` : Identities of parties can be verified using certificates.

To get all this features we needs some type of certificates and keys to allow
secured transmission on unsecured channel.
- CA (Certificate Authority) : Signs certificates and established trusts
- Private key : Secret key for encryption/decryption and signing
- Certificate : Public identity, signed by CA

For testing we can create:
- `ca_certificate.pem` == Trusted CA certificate
- `client_certificate.pem` == Client certificate (signed by CA)
- `private_key.pem` == Client’s private key

#### Generating Test Certificates with OpenSSL (Very Basic only server private key and self signed server certificate)
```shell
# 1. Create server private key
openssl genrsa -out server_key.pem 2048

# 2. Create self-signed server certificate
openssl req -new -x509 -key server_key.pem -out server_cert.pem -days 365 -subj "/C=IN/ST=Karnataka/L=Bengaluru/O=Self/CN=127.0.0.1"
```
This will generate:
- server_key.pem → server private key
- server_cert.pem → server certificate

**client.py**
```python
import socket, ssl

HOST = "127.0.0.1"
PORT = 8443

context = ssl.create_default_context()
context.check_hostname = False  # disable hostname verification for local test
context.verify_mode = ssl.CERT_NONE  # skip verifying server cert

with socket.create_connection((HOST, PORT)) as sock:
    with context.wrap_socket(sock, server_hostname=HOST) as ssock:
        print("SSL established.")
        ssock.sendall(b"Hello from client!")
        data = ssock.recv(1024)
        print("Received:", data.decode())
```

**server.py**
```python
import socket, ssl

HOST = "127.0.0.1"
PORT = 8443

context = ssl.SSLContext(ssl.PROTOCOL_TLS_SERVER)
context.load_cert_chain(certfile="server_cert.pem", keyfile="server_key.pem")

with socket.socket(socket.AF_INET, socket.SOCK_STREAM, 0) as sock:
    sock.bind((HOST, PORT))
    sock.listen(5)
    print(f"Server listening on {HOST}:{PORT} ...")
    with context.wrap_socket(sock, server_side=True) as ssock:
        conn, addr = ssock.accept()
        print("Connection from:", addr)
        data = conn.recv(1024)
        print("Received:", data.decode())
        conn.sendall(b"Hello from server!")

```
<img width="365" height="101" alt="image" src="https://github.com/user-attachments/assets/ab015360-dbb8-4e76-a9a4-bd8f68e6358d" />

<img width="364" height="81" alt="image" src="https://github.com/user-attachments/assets/2e8438ca-01dc-4696-82f8-c4b97554bcc1" />


Here, disabled host name verification, this is very basic example of our setup.
But you should have private_key, ca_certificate and 
```
ca_certificate.pem
ca_private_key.pem
client_certificate.pem
private_key.pem
server_certificate.pem
```
