# Lab — Digital Signatures (Integrity + Authenticity)

## Goal

This lab builds operational understanding of digital signatures and the security properties of integrity and authenticity.

You will:

- Generate a signing key pair
- Sign a file using a private key
- Verify the signature using the public key
- Observe how tampering invalidates a signature

This is the same mechanism used to sign certificates in PKI systems.

---

## Part 1 — Setup

### Prerequisites

- OpenSSL installed
- Access to a local terminal
- Week 2 portfolio folder created

All commands must be executed locally.

---

## Part 2 — Execution Steps

### Step 1 — Create Artifact Directory

From the root of your repository:

mkdir -p labs/02-week-02-cryptography-fundamentals/submissions/signatures

### Step 2 — Create a File to Sign
echo "Week 2 Digital Signature Lab - CVI" > labs/02-week-02-cryptography-fundamentals/submissions/signatures/artifact.txt

### Step 3 — Generate a Private Key
openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 -out labs/02-week-02-cryptography-fundamentals/submissions/signatures/private_key.pem
  
### Step 4 — Extract the Public Key
openssl pkey \
  -in labs/02-week-02-cryptography-fundamentals/submissions/signatures/private_key.pem \
  -pubout \
  -out labs/02-week-02-cryptography-fundamentals/submissions/signatures/public_key.pem
  
### Step 5 — Sign the File
openssl dgst -sha256 \
  -sign labs/02-week-02-cryptography-fundamentals/submissions/signatures/private_key.pem \
  -out labs/02-week-02-cryptography-fundamentals/submissions/signatures/artifact.sig \
  labs/02-week-02-cryptography-fundamentals/submissions/signatures/artifact.txt

This produces a digital signature file.

### Step 6 — Verify the Signature
openssl dgst -sha256 \
  -verify labs/02-week-02-cryptography-fundamentals/submissions/signatures/public_key.pem \
  -signature labs/02-week-02-cryptography-fundamentals/submissions/signatures/artifact.sig \
  labs/02-week-02-cryptography-fundamentals/submissions/signatures/artifact.txt

Expected output:

Verified OK

### Step 7 — Tamper With the File
echo "tampered" >> labs/02-week-02-cryptography-fundamentals/submissions/signatures/artifact.txt

Now verify again using the same command.

The verification should fail.

## Part 3 — Observations
Document the following in your Week 2 notes:
- Why verification succeeds before tampering
- Why verification fails after modification
- Why digital signatures require both hashing and asymmetric cryptography
- How this relates to certificate signing in PKI

## Submission (Portfolio Repo)
Ensure the following files exist:

labs/02-week-02-cryptography-fundamentals/submissions/signatures/
  artifact.txt
  artifact.sig
  public_key.pem
  
### Important

### Do NOT commit private_key.pem.

Private keys must never be stored in version control.

Delete the private key after completing verification if necessary.

Commit and push your changes.

## Stretch (Optional)
- Inspect the public key file. What format is it in?
- Try signing with a different hash algorithm.
- Research how a Certificate Authority signs an X.509 certificate.

CVI PKI Career Pathway — Foundations Phase
