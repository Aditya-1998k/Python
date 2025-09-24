## SSL TLS with Sockets – Encrypted Communication
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

**generate_certs.sh**
```shell
#!/bin/bash
set -e

# Cleanup old files
rm -f ca_key.pem ca_cert.pem ca_cert.srl \
      server_key.pem server_cert.pem server.csr server_ext.cnf \
      client_key.pem client_cert.pem client.csr client_ext.cnf

echo "🔑 Generating CA..."
openssl genrsa -out ca_key.pem 2048
openssl req -x509 -new -nodes -key ca_key.pem -sha256 -days 365 \
    -out ca_cert.pem \
    -subj "/C=IN/ST=Karnataka/L=Bengaluru/O=MyCA/CN=MyRootCA"

echo "📜 Generating Server Certificate..."
openssl genrsa -out server_key.pem 2048
openssl req -new -key server_key.pem -out server.csr \
    -subj "/C=IN/ST=Karnataka/L=Bengaluru/O=Server/CN=127.0.0.1"

cat > server_ext.cnf <<EOF
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, keyEncipherment
extendedKeyUsage=serverAuth
subjectAltName = @alt_names

[alt_names]
IP.1 = 127.0.0.1
DNS.1 = localhost
EOF

openssl x509 -req -in server.csr -CA ca_cert.pem -CAkey ca_key.pem \
    -CAcreateserial -out server_cert.pem -days 365 -sha256 \
    -extfile server_ext.cnf

echo "👤 Generating Client Certificate..."
openssl genrsa -out client_key.pem 2048
openssl req -new -key client_key.pem -out client.csr \
    -subj "/C=IN/ST=Karnataka/L=Bengaluru/O=Client/CN=TestClient"

cat > client_ext.cnf <<EOF
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, keyEncipherment
extendedKeyUsage=clientAuth
subjectAltName = @alt_names

[alt_names]
DNS.1 = TestClient
EOF

openssl x509 -req -in client.csr -CA ca_cert.pem -CAkey ca_key.pem \
    -CAcreateserial -out client_cert.pem -days 365 -sha256 \
    -extfile client_ext.cnf

echo "✅ Certificates generated successfully!"
echo "Files created:"
ls -1 ca_cert.pem server_cert.pem server_key.pem client_cert.pem client_key.pem
```
Run the above file:
```bash
🕒 09:20 ➜  chmod +x generate_certs.sh 
🕒 09:20 ➜  ./generate_certs.sh 
🔑 Generating CA...
📜 Generating Server Certificate...
Certificate request self-signature ok
subject=C = IN, ST = Karnataka, L = Bengaluru, O = Server, CN = 127.0.0.1
👤 Generating Client Certificate...
Certificate request self-signature ok
subject=C = IN, ST = Karnataka, L = Bengaluru, O = Client, CN = TestClient
✅ Certificates generated successfully!
Files created:
ca_cert.pem
client_cert.pem
client_key.pem
server_cert.pem
server_key.pem
```

**server.py**
```python
import socket, ssl

context = ssl.create_default_context(ssl.Purpose.CLIENT_AUTH)
context.verify_mode = ssl.CERT_REQUIRED
context.load_cert_chain(certfile="server_cert.pem", keyfile="server_key.pem")
context.load_verify_locations(cafile="ca_cert.pem")

with socket.socket(socket.AF_INET, socket.SOCK_STREAM, 0) as sock:
    sock.bind(("127.0.0.1", 8443))
    sock.listen(5)
    with context.wrap_socket(sock, server_side=True) as ssock:
        print("Server listening on 127.0.0.1:8443 ...")
        conn, addr = ssock.accept()
        print("Connection from:", addr)
        print("Peer cert:", conn.getpeercert())
        data = conn.recv(1024)
        print("Received:", data.decode())
```
**client.py**
```python
import socket, ssl

context = ssl.create_default_context(
    ssl.Purpose.SERVER_AUTH, cafile="ca_cert.pem"
)
context.load_cert_chain(certfile="client_cert.pem", keyfile="client_key.pem")

with socket.create_connection(("127.0.0.1", 8443)) as sock:
    with context.wrap_socket(sock, server_hostname="127.0.0.1") as ssock:
        print("SSL established. Peer:", ssock.getpeercert())
        ssock.sendall(b"Hello secure world!")
```

