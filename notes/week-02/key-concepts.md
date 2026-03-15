---
## Cryptography Fundamentals

## Digital Trust

One of the main ideas this week was digital trust. The lesson was explained using the airport security model. Just like in an airport where passengers must show identification before entering secure areas, systems on the internet must verify identity before allowing secure communication. Identity verification helps ensure that users and systems are who they claim to be.

## Encryption

Encryption protects information by transforming readable data into an unreadable format. Only someone with the correct key can convert it back into its original form. This protects sensitive data when it is stored or transmitted.

## Symmetric Cryptography

Symmetric encryption uses one shared key for both encryption and decryption. This was compared with a shared access badge. If two people both have the same badge, they can enter the same secure area.
Symmetric encryption is fast and efficient, which makes it good for encrypting large amounts of data. One commonly used algorithm is AES (Advanced Encryption Standard).

The main weakness of symmetric encryption is the key distribution problem. Both parties must already have the same secret key before communication begins. If the key is sent over an insecure channel, someone could intercept it.


## Asymmetric Cryptography

Asymmetric cryptography was designed to solve the key distribution problem. Instead of one key, it uses two keys: a public key and a private key.

This was explained using a passport system analogy. A passport proves your identity and allows you to enter secure locations. Similarly, public key cryptography helps verify identity in digital systems.

Common asymmetric algorithms include RSA and ECC (Elliptic Curve Cryptography). RSA is widely used for digital signatures and key exchange, while ECC provides strong security with smaller key sizes and is often used in modern systems.

## TLS Handshake

The TLS handshake is the process that establishes a secure connection between a client and a server. It combines asymmetric and symmetric cryptography.

First, asymmetric cryptography is used to authenticate the server and securely exchange a session key. After that, symmetric encryption is used for the rest of the communication because it is faster.

TLS 1.3 is the modern standard and replaced older SSL versions.


## Digital Certificates

Digital certificates act like digital passports. They prove that a public key belongs to a specific organization or server.

Certificates contain information such as the public key, the organization name, and the issuing Certificate Authority (CA). Certificate Authorities sign these certificates to create trust.

Certificates also have a lifecycle that includes issuance, active use, expiration, renewal, and revocation if the certificate is compromised.

## PKI Risks and Failures

Public Key Infrastructure can fail if certain problems occur. For example, if a certificate expires, if the trust chain is broken, or if a Certificate Authority’s private key is compromised. If a CA’s private key were stolen, attackers could issue fake certificates and impersonate trusted websites.

## CIA Triad

The CIA triad represents three core security goals: confidentiality, integrity, and availabilty.

Confidentiality protects data from unauthorized access. Integrity ensures that data has not been altered or tampered with. Availability ensures that systems and data remain accessible to authorized users when they are needed.

## Hashing

Hashing creates a unique digital fingerprint of data. A hash function takes an input and produces a fixed-length output. If even a small change is made to the data, the hash changes completely.
Common hashing algorithms include SHA-256.

## Digital Signatures

Digital signatures combine hashing and asymmetric cryptography. First, a hash of the data is created. Then the hash is signed using a private key. Anyone with the public key can verify the signature.
This proves two things: that the data came from the correct sender and that the data has not been modified.

Hybrid Cryptography

Most real-world systems use hybrid cryptography, which combines symmetric and asymmetric encryption. Asymmetric cryptography is used for authentication and secure key exchange, while symmetric encryption is used for efficient and fast data transmission.

By combining both approaches, hybrid cryptography helps support the goals of the CIA triad. Symmetric encryption protects confidentiality, hashing helps maintain integrity, and strong system design helps ensure availability of secure services. Together, these mechanisms enable trusted and secure communication systems such as HTTPS and TLS.

## Lab Observations

During the labs, I saw how these concepts work in practice. When I hashed a file and then modified it, the hash value changed completely. When I signed a file and then tampered with it, the signature verification failed. This demonstrated how cryptography can detect unauthorized changes and protect data integrity.

