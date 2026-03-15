---
### REFLECTION: TLS INSPECTIONS

## 1. Certificate Authorities & Trust

Certificate Authorities (CAs) help establish trust on the internet. They verify the identity of websites and issue digital certificates that confirm a website is legitimate. When a browser connects to a website, 
it checks the certificate and verifies that it was signed by a trusted CA.

Browsers and operating systems maintain a list of trusted root certificate authorities. If a website’s certificate can be traced back through a chain of trusted CAs in this list, the connection is considered secure. 
If the certificate cannot be verified, the browser usually shows a warning to the user.

## 2. Symmetric Encryption After Handshake

Symmetric encryption is used after the TLS handshake because it is much faster and more efficient than asymmetric encryption. 
During the handshake, asymmetric cryptography is used to securely exchange keys and verify identities.

Once the secure connection is established, both the client and server share a symmetric session key. This key is then used to encrypt the rest of the communication. 
Using symmetric encryption makes the connection faster while still keeping the data confidential and secure.

## 3. Reading a Cipher Suite

The cipher suite describes the cryptographic algorithms used to secure a connection.

For example: ` AEAD-CHACHA20-POLY1305-SHA256`

` AEAD ` – Authenticated Encryption with Associated Data, meaning encryption and authentication are combined for better security.

` ChaCha20 ` – The symmetric encryption algorithm used to encrypt the data.

` Poly1305 ` – The authentication algorithm used to ensure the integrity of the data.

` SHA256 ` – The hashing algorithm used during the TLS handshake and key derivation.

Together, these components ensure that the communication is encrypted, the data remains unchanged, and both parties can trust the connection.
