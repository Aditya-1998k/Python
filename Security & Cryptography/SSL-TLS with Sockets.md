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

#### Generating Test Certificates with OpenSSL
1. **Create a CA**
   ```shell
   # Generate CA private key
    openssl genrsa -out ca_private_key.pem 2048
    
    # Self-signed CA certificate
    openssl req -x509 -new -nodes -key ca_private_key.pem -sha256 -days 365 \
        -out ca_certificate.pem
   ```
   <img width="809" height="322" alt="image" src="https://github.com/user-attachments/assets/a7a6826e-9e19-4d99-a63c-6440f7301c47" />

2. **Create a Client Key**
   ```shell
   openssl genrsa -out private_key.pem 2048
   ```
3. **Generate a CSR (Certificate Signing Request)**
   ```shell
   openssl req -new -key private_key.pem -out client.csr
   ```
   <img width="738" height="389" alt="image" src="https://github.com/user-attachments/assets/cdd71c6a-e4f4-4650-86e4-d795efa93640" />

4. **Sign Client Certificate with CA**
   ```shell
   openssl x509 -req -in client.csr -CA ca_certificate.pem -CAkey ca_private_key.pem \
    -CAcreateserial -out client_certificate.pem -days 365 -sha256
   ```

Final Artifacts:
- ca_certificate.pem
- client_certificate.pem
- private_key.pem
<img width="391" height="129" alt="image" src="https://github.com/user-attachments/assets/b2cdd09e-b3ad-4b50-95f7-716c65cfa69c" />

**client.py**
```python
import socket, ssl

# Create SSL context with CA
context = ssl.create_default_context(
    ssl.Purpose.SERVER_AUTH,
    cafile="ca_certificate.pem"
)

# Load client certificate + private key
context.load_cert_chain(
    certfile="client_certificate.pem",
    keyfile="private_key.pem"
)

# Connect to server securely
with socket.create_connection(("127.0.0.1", 8443)) as sock:
    with context.wrap_socket(sock, server_hostname="localhost") as ssock:
        print("SSL established. Peer:", ssock.getpeercert())
        ssock.sendall(b"Hello secure world!")
```

**server.py**
```python
import socket, ssl

context = ssl.create_default_context(ssl.Purpose.CLIENT_AUTH)
context.verify_mode = ssl.CERT_REQUIRED
context.load_cert_chain(certfile="client_certificate.pem", keyfile="private_key.pem")
context.load_verify_locations(cafile="ca_certificate.pem")

with socket.socket(socket.AF_INET, socket.SOCK_STREAM, 0) as sock:
    sock.bind(("127.0.0.1", 8443))
    sock.listen(5)
    with context.wrap_socket(sock, server_side=True) as ssock:
        conn, addr = ssock.accept()
        print("Connection from:", addr)
        print("Peer cert:", conn.getpeercert())
        data = conn.recv(1024)
        print("Received:", data)
```
