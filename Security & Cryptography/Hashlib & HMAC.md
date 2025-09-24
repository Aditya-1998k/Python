### Hashing
A hash is a fixed-length string generated from input data.

**Properties:**
1. Deterministic (same input = same Hash)
2. It will be fast, as it will follow one pattern
3. Hard to reverse to input by attacker (only they can do brute force)
4. It is almost impossible to get same output for two different input.
5. Usages of this, we can convert the password in hash and store locally and if user type password
convert it and compare.

Some common Algorithm for Hashing: `MD5`, `SHA-1`, `SHA-256`, `SHA-512`

#### Python `hashlib` as standard library.

Example: SHA-256
```python
import hashlib

data = b"Hello World"

hash_object = hashlib.sha256(data)
print(hash_object.hexdigest())
```
output:
```
a591a6d40bf420404a011733cfb7b190d62c65bf0bcda32b57b277d9ad9f146e
```
Note:
1. Data is byte string, as hashlib does not work on plane string
2. `hashlib.sha256(data)` calls sha256 algorithm on data
3. `.hexdigest()` = converts the raw hash (binary digest) into a hexadecimal string (readable form).
4. `SHA-256` always outputs `256 bits` = `32 bytes` = `64 hex characters`
5. Even a small change in data, change huge in hexcode

Doing Hashing Algorithm incremently:
```python
import hashlib

hash_object = hashlib.sha256()
hash_object.update(b"Hello")
hash_object.update(b"World")

print(hash_object.hexdigest())
```

#### HMAC (Hash Based Message Authentication Code)
HMAC = Hash + Secret Key 
1. It ensures data integrity and Authenticity
2. Even if someone know the data, they cannot generate correct hash with secret key

```python
import hmac
import hashlib

key = b"mypwd"
data = b"Hello World"

hm = hmac.new(key, data, hashlib.sha256)
print(hm.hexdigest())
```
