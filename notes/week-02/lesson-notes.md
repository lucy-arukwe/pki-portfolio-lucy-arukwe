# Week 2 Lesson Notes — Cryptography Fundamentals (Just Enough)

---

## 1. Core Concept

Cryptography is the technical mechanism that makes digital trust possible.

This week focuses on three foundational security properties:

- **Confidentiality** → Protected by encryption  
- **Integrity** → Protected by hashing  
- **Authenticity** → Protected by digital signatures  

These are not abstract concepts. They are the building blocks behind:

- HTTPS
- Code signing
- Certificate validation
- Secure API communication
- Enterprise PKI systems

If Week 1 introduced trust, Week 2 explains how trust is technically enforced.

---

## 2. Why It Matters

In enterprise environments:

- Data must be encrypted in transit.
- Systems must verify that files are not altered.
- Certificates must prove identity.
- Software updates must be trusted.

Without cryptography:

- Anyone could impersonate a server.
- Data could be intercepted and modified.
- Malicious software could be distributed as legitimate.

Cryptography enforces rules that humans cannot reliably enforce alone.

---

## 3. Technical Breakdown

### A. Symmetric Encryption (Confidentiality)

**Definition:**  
A method of encryption that uses one shared secret key for both encryption and decryption.

**Example:** AES

**Flow:**
1. Data is encrypted using a secret key.
2. Ciphertext is produced.
3. The same key decrypts the data back to plaintext.

**Trust Implication:**  
If the key is exposed, confidentiality is lost.

**Where It Appears:**  
TLS uses symmetric encryption for actual data transfer after a secure session is established.

---

### B. Hashing (Integrity)

**Definition:**  
A one-way mathematical function that produces a fixed-length fingerprint of data.

**Example:** SHA-256

**Properties:**
- Deterministic
- Fixed output length
- Sensitive to small input changes
- Not reversible

**Flow:**
1. Input data is processed by a hash algorithm.
2. A unique digest is generated.
3. Any change to the input changes the digest.

**Trust Implication:**  
Hashing allows systems to detect tampering.

**Where It Appears:**
- Certificate signatures
- File integrity validation
- Software distribution
- Git commit tracking

---

### C. Digital Signatures (Integrity + Authenticity)

**Definition:**  
A cryptographic mechanism that proves both integrity and identity.

**Components:**
- Private key (used to sign)
- Public key (used to verify)
- Hash of the data

**Flow:**
1. Data is hashed.
2. The hash is encrypted with a private key.
3. The resulting signature is attached to the data.
4. The public key verifies the signature.

**Trust Implication:**  
Only the private key holder can produce a valid signature.

If verification fails:
- The data was altered  
OR  
- The signature is not from the expected signer  

**Where It Appears:**
- X.509 certificate signatures
- Code signing
- Secure software updates
- Signed configuration files

---

## 4. Common Misconceptions

**Misconception 1:**  
Encryption and hashing are the same.

Reality:  
Encryption protects secrecy.  
Hashing detects change.

---

**Misconception 2:**  
Digital signatures encrypt data.

Reality:  
Digital signatures do not encrypt the original data.  
They sign a hash of the data.

---

**Misconception 3:**  
Asymmetric encryption protects all TLS traffic.

Reality:  
Asymmetric cryptography establishes identity and negotiates keys.  
Symmetric encryption protects the data.

---

## 5. Where This Shows Up

### Web Security
- HTTPS certificates are digitally signed.
- TLS sessions use symmetric encryption.

### Internal Enterprise Systems
- Internal certificate authorities sign device certificates.
- File integrity monitoring uses hashing.

### Cloud Environments
- API authentication relies on signatures.
- Identity systems use certificates for trust validation.

### DevOps Workflows
- Code signing ensures software authenticity.
- Hashes verify container images.
- Git uses hashing for commit integrity.

---

## Mental Model

Encryption protects secrets.

Hashing protects integrity.

Digital signatures protect identity and integrity.
Together, these form the enforcement layer of digital trust.

Identity → Verified  
Trust → Enforced  
Verification → Automated

Week 3 will build on this foundation by showing how certificates combine these mechanisms into a structured trust model.
