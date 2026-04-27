# Lab 01 — Inspect X.509 Certificate Fields

## Overview
This lab focused on retrieving a live TLS certificate from google.com and parsing it using OpenSSL to examine its core X.509 fields. 
The objective was to understand how certificates represent identity within a PKI system by identifying the issuing authority, the entity being validated, 
and the certificate’s validity period.

---

## Environment
- OS: Windows 11  
- Terminal: Git Bash (MINGW64)  
- OpenSSL Version: OpenSSL 3.5.5 (27 Jan 2026)

---

## Certificate Fields

| Field                                | Value                                           |
|--------------------------------------|-------------------------------------------------|
| Version                              | 3 (0x2)                                         |
| Serial Number                        | 6e:3f:5c:f0:61:00:75:86:10:aa:98:b7:4a:a6:e7:cc |
| Signature Algorithm                  | ecdsa-with-SHA256                               |
| Issuer                               | C=US, O=Google Trust Services, CN=WE2           |
| Not Before                           | Mar 30 08:35:17 2026 GMT                        |
| Not After                            | Jun 22 08:35:16 2026 GMT                        |
| Subject                              | CN=*.google.com                                 |
| Public Key Algorithm                 | id-ecPublicKey (256 bit, NIST P-256)            |

---

## Observations

1. The certificate is issued by Google Trust Services, specifically the intermediate CA labeled WE2.
   This indicates that the certificate is not signed directly by a root CA but by an intermediate authority within Google’s trust hierarchy.

2. The Subject is a wildcard domain (*.google.com), allowing the certificate to secure multiple subdomains under the google.com namespace.
   This approach supports scalability and simplifies certificate management across distributed services.

3. The validity period is short, approximately 84 days. Short-lived certificates reduce the risk window in case of key compromise and align with
   modern certificate lifecycle practices.

4. The certificate uses ECDSA with a 256-bit key on the NIST P-256 curve. This reflects a shift toward elliptic curve cryptography, which provides strong
   security with improved performance compared to traditional RSA.

6. The Issuer field plays a critical role in trust validation. It identifies the Certificate Authority responsible for issuing the certificate, allowing clients
   to verify the certificate through a trusted chain up to a root CA.

---

## Key Findings

- The use of ECDSA with SHA-256 highlights a modern cryptographic approach focused on efficiency and strong security.
- The wildcard subject enables coverage of multiple subdomains using a single certificate, which is common in large-scale environments.
- The short validity window reflects current best practices that prioritize frequent certificate rotation.
- The Basic Constraints extension includes CA:FALSE, confirming that the certificate is a leaf certificate and cannot be used to issue other certificates.

---

## Explanation

X.509 certificates form the foundation of trust in PKI systems. The Issuer field establishes the chain of trust, allowing clients to validate the certificate 
against a trusted root authority. The Subject identifies the entity being secured, while the validity window limits how long the certificate remains trusted.

The use of elliptic curve cryptography demonstrates an industry shift toward more efficient algorithms that maintain strong security guarantees. The CA:FALSE constraint 
ensures that the certificate cannot be misused to sign other certificates, preserving the integrity of the trust hierarchy.

---

## Challenges / Troubleshooting

The certificate retrieval and parsing process completed successfully without errors. Output validation was performed by confirming that all expected fields were present in 
the OpenSSL output.

---

## Artifacts

- `leaf_cert.pem` — X.509 certificate retrieved from google.com
