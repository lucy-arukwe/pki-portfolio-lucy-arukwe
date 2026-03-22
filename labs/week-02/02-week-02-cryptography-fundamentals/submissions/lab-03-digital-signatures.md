# Lab — Digital Signatures (Integrity + Authenticity)

## Overview
This lab focused on understanding how digital signatures work and the security properties they provide. An RSA key pair was generated, a file was signed using the private key, and the signature was verified using the public key. The file was then modified to observe how verification fails when content changes, showing how digital signatures detect tampering and confirm the identity of the signer. This is the same mechanism used when a Certificate Authority signs an X.509 certificate.

---

## Environment

- Operating System: Windows 11
- Terminal Used: Git Bash (MINGW64)
- OpenSSL Version: OpenSSL 3.5.5 27 Jan 2026 

---

## Steps Performed

1. Created the `submissions/signatures/` directory inside the Week 2 lab folder.
2. Created a plaintext file containing "Week 2 Digital Signature Lab - CVI" as the artifact to be signed.
3. Generated a 2048-bit RSA private key using OpenSSL.
4. Extracted the corresponding public key from the private key and saved it as a separate file.
5. Signed the artifact file using the private key with SHA-256 hashing, producing a `.sig` signature file.
6. Verified the signature against the artifact using the public key and received `Verified OK`.
7.  Modified the original file to simulate tampering and attempted verification again.  
8. Ran verification again on the modified file and received `Verification failure`.
9. Deleted the private key to ensure it was not committed to version control.

---

## Results

Before tampering:
`Verified OK`

After appending "tampered" to the file:
`Verification failure
error:02000068:rsa routines:ossl_rsa_verify:bad signature`

The signature that was valid before tampering became completely invalid after a single change to the file.


---

![Artifact file created](../../../../assets/screenshots/week-02/week02-lab03-artifact-created.png)


Key generation (RSA 2048-bit):

![Key generation](../../../../assets/screenshots/week-02/week02-lab03-key-generation.png)


Signature verification before modification:

![Verification success](../../../../assets/screenshots/week-02/week02-lab03-signature-verified.png)


Signature verification after file modification:

![Verification failed](../../../../assets/screenshots/week-02/week02-lab03-signature-verification-failed.png)


---

## Key Findings

- Signature verification succeeded when the file was unchanged, confirming both integrity and authenticity.
- A small modification caused verification to fail immediately, showing that signatures are tied to the exact content.
- The private key is used for signing and must remain secret, while the public key is used for verification and can be shared.
- Digital signatures combine hashing and asymmetric cryptography, and both are required for the process to work.

---

## Explanation

Digital signatures provide integrity and authenticity. Integrity ensures the content has not been altered, while authenticity confirms the content was created by the holder of the private key.

The process works by hashing the file and encrypting that hash using the private key. During verification, the file is hashed again and compared to the decrypted signature using the public key.

In PKI, Certificate Authorities use this same process to sign certificates. Systems verify those signatures using the CA’s public key, ensuring the certificate is legitimate and unchanged.

If a private key is exposed or committed to version control, it can be used to forge signatures and impersonate the signer. This is why the private key was deleted after completing the lab.

---

## Challenges / Troubleshooting

No major issues were encountered. Care was taken to ensure correct file paths and proper command usage. Verification failure after tampering confirmed expected behavior.

---

## Artifacts
- `labs/week-02/02-week-02-cryptography-fundamentals/submissions/signatures/artifact.txt` — the signed file (tampered version after modification)
- `labs/week-02/02-week-02-cryptography-fundamentals/submissions/signatures/artifact.sig` — the digital signature
- `labs/week-02/02-week-02-cryptography-fundamentals/submissions/signatures/public_key.pem` — the RSA public key used for verification
- `labs/week-02/02-week-02-cryptography-fundamentals/submissions/signatures/lab-03-digital-signatures.md` — this write-up
- Screenshots to this lab are stored in `assets/screenshots/`

---

*CVI PKI Career Pathway — Foundations Phase*
