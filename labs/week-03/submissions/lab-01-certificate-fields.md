# Lab 01 — Inspect X.509 Certificate Fields

## Overview
This lab involved connecting to google.com using OpenSSL to retrieve 
its live X.509 certificate. The certificate was then parsed to identify 
the core fields that define identity and trust in a PKI system. This 
exercise builds practical understanding of how certificates are 
structured and what each field represents in a real-world context.

---

## Environment
- OS: Windows `11`
- Terminal used: `Git Bash (MINGW64)`
- OpenSSL version: `OpenSSL 3.5.5 27 Jan 2026`

---

## Certificate Fields

| Field                | Value                                            |
|----------------------|--------------------------------------------------|
| Version              | 3 (0x2)                                          |
| Serial Number        | aa:23:02:42:8e:f4:39:7e:10:bb:2c:32:93:1c:fc:2e |
| Signature Algorithm  | ecdsa-with-SHA256                                |
| Issuer               | C=US, O=Google Trust Services, CN=WE2            |
| Subject              | CN=*.google.com                                  |
| Not Before           | Feb 23 18:19:56 2026 GMT                         |
| Not After            | May 18 18:19:55 2026 GMT                         |
| Public Key Algorithm | id-ecPublicKey (prime256v1, 256 bit)             |

---

## Observations

1. Who issued the certificate?
Google Trust Services issued this certificate through their 
intermediate CA named WE2.

2. What domain or organization does it represent?
The certificate represents *.google.com, a wildcard certificate 
covering all subdomains under google.com.

3. When does it expire?
The certificate expires on May 18, 2026.

4. What public key algorithm is used?
Elliptic Curve cryptography (id-ecPublicKey) is used with the P-256 
curve at 256 bits, a modern, efficient alternative to RSA that 
provides strong security with a smaller key size.

5. Why does the Issuer field matter in a PKI system?
The Issuer field identifies which Certificate Authority has signed 
and validated the certificate. It forms the backbone of the chain 
of trust. Without a recognized and trusted issuer, the certificate 
cannot be verified and the connection would not be trusted by 
browsers or clients.
