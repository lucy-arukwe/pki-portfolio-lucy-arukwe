# Lab 03 — Verify a Certificate Chain

## Overview
This lab focused on retrieving the full TLS certificate chain from github.com, examining each certificate in the hierarchy, and verifying the chain using OpenSSL. The objective was to understand how root, intermediate, and leaf certificates work together to establish trust within a PKI system.

---

## Environment
- OS: Windows 11  
- Terminal: Git Bash (MINGW64)  
- OpenSSL Version: OpenSSL 3.5.5 (27 Jan 2026)  
- Website: github.com  

---

## Chain Verification Result: 
`server.pem: OK`


The chain was successfully verified using Mozilla’s CA bundle (`cacert.pem`) as the trusted CA file, with the intermediate certificate supplied using the `-untrusted` flag.

---

## Certificate Roles

| Certificate | Subject | Issuer | CA:TRUE/FALSE |
|---|---|---|---|
| server.pem | CN=github.com | CN=Sectigo Public Server Authentication CA DV E36 | CA:FALSE |
| intermediate.pem | CN=Sectigo Public Server Authentication CA DV E36 | CN=Sectigo Public Server Authentication Root E46 | CA:TRUE |
| root.pem | CN=Sectigo Public Server Authentication Root E46 | CN=USERTrust ECC Certification Authority | CA:TRUE |

---

## Observations

1. The chain consists of three levels: a leaf certificate, an intermediate CA, and a root CA, forming a standard PKI trust hierarchy.

2. The root certificate, Sectigo Public Server Authentication Root E46, is a trusted authority that ultimately anchors the chain. It traces back to USERTrust ECC Certification Authority, which is included in client trust stores.

3. The intermediate CA, Sectigo Public Server Authentication CA DV E36, has CA:TRUE and is responsible for issuing the leaf certificate.

4. The leaf certificate, issued to CN=github.com, has CA:FALSE, confirming that it cannot issue other certificates and is strictly used for end-entity authentication.

5. The trust relationship is established through Issuer-to-Subject matching. Each certificate’s Issuer corresponds to the Subject of the certificate above it, forming a continuous and verifiable chain.

6. Intermediate certificates provide an important security layer by allowing the root CA private key to remain offline. This reduces exposure and limits the impact of potential compromise.

---

## Key Findings

- The chain follows a standard three-tier PKI model: root CA → intermediate CA → leaf certificate.
- Elliptic curve cryptography is used across the chain, with the leaf and intermediate using P-256 and the root using P-384, indicating a strong and modern cryptographic configuration.
- Both the intermediate and root certificates include CA:TRUE and appropriate Key Usage values such as Certificate Sign and CRL Sign, confirming their authority to issue and manage certificates.
- The leaf certificate includes CA:FALSE and limited Key Usage, enforcing its role as an end-entity certificate and preventing misuse.

---

## Explanation

Certificate chain validation is the process that allows clients to establish trust in a TLS connection. A leaf certificate alone is not sufficient; it must be linked through one or more intermediate certificates to a trusted root CA stored in the client’s trust store.

This trust path is built through Issuer-to-Subject relationships between certificates. Each step in the chain confirms the authenticity of the certificate below it.

Intermediate CAs play a critical role by separating certificate issuance from the root authority. This allows the root CA’s private key to remain securely offline while still enabling scalable certificate issuance. If an intermediate CA is compromised, it can be revoked without requiring a complete rebuild of the trust hierarchy.

---

## Challenges / Troubleshooting

Initial verification using `root.pem` as the CAfile resulted in an “unable to get issuer certificate” error. This occurs because OpenSSL on Windows does not automatically use the system’s certificate store.

The issue was resolved by downloading Mozilla’s trusted CA bundle (`cacert.pem`) and using it as the CAfile. With the intermediate certificate supplied using the `-untrusted` flag, the chain verification completed successfully and returned `server.pem: OK`.

---

## Artifacts

- `server.pem` — leaf certificate retrieved from github.com  
- `intermediate.pem` — intermediate CA certificate  
- `root.pem` — root CA certificate  
