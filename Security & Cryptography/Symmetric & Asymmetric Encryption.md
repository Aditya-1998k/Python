## Cryptography Library – Symmetric & Asymmetric Encryption
The cryptography library in python provides high level recipes and low level primitives
for secure encryption.

### 1. Symmetric Encryption
One key is used for both `encryption` and `decryption`.
It will be fast and used for data encryption. It is fast so if data is huge
then we can use this for huge data encryption.
Example Algorith : `AES` and ChaCha20 


##### **Example of Encrypting Member Id with AES with Fernet**
- Generate and store a fernet key only once. Keep it safe and secret (.env or vault)
- When storing the member ID encrypt it using Fernet and save in DB.
- When fetching the user, decrypt the value back to original.

```python
from cryptography.fernet import Fernet

fernet_key = Fernet.generate_key()
cipher = Fernet(fernet_key)


mbr_id = "12345"

# Store in DB
encrypted_id = cipher.encrypt(mbr_id.encode())
print(f"Encrypted Data stored in database {encrypted_id}")


# Fetch from DB
decrypted_id = cipher.decrypt(encrypted_id).decode()
print(f"Decrypted Data fetched from db {decrypted_id}")
```
Output:
```
Encrypted Data stored in database b'gAAAAABo1gY4ZijWFBMIkkrNkDYnIItIzS9UxyCMoK-Vr6G73uhGgUVDGOzxhB4YK6aLgEpOkH8xzcQoUhb5_78nRpE0wbwyYQ=='
Decrypted Data fetched from db 12345
```
Note:
1. Same key must be used for encryption and decryption.
2. If the key changes → old encrypted values cannot be decrypted.
3. Store the Key securily, not in codebase
4. Fernet provides: `AES encryption` and `HMAC-SHA256` for integrity


### 2. Asymmetric Encryption (Public/Private Key)
Asymmetric Encryption uses two key, public and private.
- Public Key - encrypt/verify
- Private Key - Decrypt/sign

This is slower, but also enables the key exchange and digital signatures.

Example Algorithm : RSA and ECC

**RSA Encrypt/Decrypt**
```

from cryptography.hazmat.primitives.asymmetric import rsa, padding
from cryptography.hazmat.primitives import hashes

# Generate RSA key-pair
private_key = rsa.generate_private_key(public_exponent=65537, key_size=2048)
publice_key = private_key.public_key()

# Encrypt with public_key
message = b"Top Secret"

cipher_text = publice_key.encrypt(message, padding.OAEP(mgf=padding.MGF1(algorithm=hashes.SHA256()), algorithm=hashes.SHA256(), label=None))
print("Encrypted Data:", cipher_text)

# Decrypt with private key
plain_text = private_key.decrypt(cipher_text, padding.OAEP(mgf=padding.MGF1(algorithm=hashes.SHA256()), algorithm=hashes.SHA256(), label=None))
print("Decrypted Data: ", plain_text)
```
Output:
```
Encrypted Data: b'^.\x00\xcf\xae\x90^\xf7\x14;\x92d\xce?\xfd\xf2\x14\xec\x93\xfc\xbf\x874\xee\x9dAi\xfe\xa6\xf0\xd1v\xbdk=\xad\x03\xe8\x92\x8f\xcc|\xd
f\xfd8\xb6\x91%5\xc8\x0fG8\xf0/\xbb\x7f\xe7!\xc1\n\xda \x7f\xeb\xf7\xf2X?\xd0xBL\r\x94\xd5\xf6K\xa5\xa1%t%-\x8a\xbc\x03\x90#%`\xb9\x80\xf0\x0c\xb1\x07
\x88\x93\x8df\xb9\xcb\x05\x98\x88\x0c*\xdaa\x87N&\xe4+H\xf6\x93\xa4\x13B\xa9T\xc0\x02-\xcb\xd6\x02pFou\xfe{\xed?&v\xf5ZH\x81\x04\xcad\xeb\xb8\xfb7\x83
\xdd\x9a\xe1h9\xbd\xfbRn\x0e\x95\xb5\xbcu\xc6\x88\x12Z\x14\x02\xc0\x02\x1c\x87\x17\xe1\x05\x7fc\x06\xc3w\xcd+\xa1\xc2\xce\xa5o{\xdfW8K\xa0\xbda\xe9d\x
8d\xcd\xaf5\xbf\xf0oF\x8f\xf8K\xb4\xccx"\xf3\xaf9\xa4c.\xb0>\x0fS\x15$[\xd9\xf5\x83\x80\r\x84\xf6\x82I\t\xca\x96\x94\x10\xde\xd3\x14\x87;S\xcd\xe6]j\x
fe\x10\xe0\xf3'
Decrypted Data:  b'Top Secret'
```
Note:
1. Useful for small message with high security
2. Public key can be shared openly but private key must be kept secret.
