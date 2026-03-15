# Week 02 Key Concepts (Symmetric Encryption)
## Observations from the cryptography fundamentals labs.

## Observations
### Why the encrypted file is unreadable

After encrypting the plaintext file using AES-256-CBC, the output file (`plaintext.txt.enc`) contained unreadable characters. 
This is because the original data was transformed into ciphertext using a cryptographic algorithm and a key derived from the password. 
Without the correct key, the encrypted data cannot be interpreted in a meaningful way.
In this lab, the same key (derived from your password via PBKDF2) both encrypts and decrypts, that's what makes it symmetric. 
The security of the entire operation rests on keeping that key secret.

### What would happen if the wrong password were used
If the wrong password is used during the decryption process, OpenSSL will not be able to correctly derive the encryption key. 
As a result, the decryption will fail or produce corrupted output that does not match the original plaintext file.

### What security property symmetric encryption provides
Symmetric encryption provides **confidentiality**. 
This means that the data is protected from unauthorized access because only someone with the correct secret key or password can decrypt and read the original information.

### Why TLS uses symmetric encryption for data transfer
TLS uses symmetric encryption for data transfer because it is significantly faster and more efficient than asymmetric encryption when handling large amounts of data. 
Once a secure session is established, symmetric encryption allows communication to remain confidential while maintaining high performance.

