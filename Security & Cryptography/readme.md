## 🔐 Security & Cryptography

This section covers key cryptography concepts, practical implementations, and hands-on examples using Python.  

---

### 📑 Contents

1. [SSL/TLS with Sockets](./SSL-TLS%20with%20Sockets.md)  
   - Basics of TLS/SSL  
   - Certificates, CA trust, and mutual authentication  
   - Python socket + SSL examples (server & client)  

2. [Hashlib & HMAC](./Hashlib%20&%20HMAC.md)  
   - Cryptographic hashing (SHA256, SHA512, etc.)  
   - Pre-image & collision resistance  
   - Message authentication using HMAC  
   - Python examples with `hashlib` & `hmac`  

3. [Symmetric & Asymmetric Encryption](./Symmetric%20&%20Asymmetric%20Encryption.md)  
   - Symmetric encryption (AES, Fernet)  
   - Asymmetric encryption (RSA, key pairs)  
   - Use-cases: data encryption, digital signatures, key exchange  
   - Python examples with `cryptography` library  

---

### 📌 Key Concepts

- **Hashing** → One-way, fixed-length output, used for integrity checks.  
- **HMAC** → Adds a secret key to hashing for authenticity.  
- **Symmetric Encryption** → Same key for encryption & decryption (e.g., Fernet, AES).  
- **Asymmetric Encryption** → Public/private key pairs (e.g., RSA).  
- **SSL/TLS** → Provides encrypted and authenticated communication using certificates.  

---

### 🎯 Top Questions

#### 🔑 Hashing & HMAC
1. What is the difference between hashing and encryption?  
2. Explain pre-image resistance and collision resistance.  
3. How does HMAC improve security over plain hashing?  
4. Can you use MD5 for security? Why or why not?  

#### 🔐 Symmetric Encryption
5. What is the main drawback of symmetric encryption?  
6. How does Fernet in Python ensure both confidentiality and integrity?  
7. Why must the encryption key be stored securely?  

#### 🔑 Asymmetric Encryption
8. Difference between symmetric and asymmetric encryption.  
9. How does RSA work (in short)?  
10. What are digital signatures and how do they work?  

#### 🌐 SSL/TLS
11. How does SSL/TLS establish a secure connection?  
12. What role does a Certificate Authority (CA) play?  
13. Why does the browser trust HTTPS websites?  
14. What is mutual TLS and when is it used?  
15. How do private and public keys interact in TLS handshake?  

---
