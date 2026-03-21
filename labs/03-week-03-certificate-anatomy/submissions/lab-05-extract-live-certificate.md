# Lab 05 — Extract a Certificate from a Live Website

## Overview
This lab involved connecting to github.com using OpenSSL to retrieve its live TLS certificate and inspect its fields and extensions. The PKI concept explored is how to extract and analyze a real-world certificate directly from a live TLS connection, similar to how security engineers perform TLS troubleshooting and incident response.
---

## Environment
- OS: Windows 11
- Terminal used:  Git Bash (MINGW64)
- OpenSSL version: `OpenSSL 3.5.5 27 Jan 2026 `
- Website used: github.com

---

## Certificate Fields Found

| Field                    | Value from your output                                                     |
|--------------------------|--------------------------------------------------------------------------- |
| Subject                  | CN=github.com                                                              |
| Issuer                   | C=GB, O=Sectigo Limited, CN=Sectigo Public Server Authentication CA DV E36 |
| Not Before               | Mar 6 00:00:00 2026 GMT                                                    |
| Not After                | Jun 3 23:59:59 2026 GMT                                                    |
| Public Key Algorithm     | id-ecPublicKey (Elliptic Curve, P-256)                                     |
| Subject Alternative Name | github.com, www.github.com                                                 |
| Key Usage                | Digital Signature                                                          |
| Extended Key Usage       | TLS Web Server Authentication                                              |

---

## Observations
1. What organization does this certificate belong to?
The certificate belongs to GitHub, as shown in the Subject field (CN=github.com), confirming it was issued to GitHub’s web server.
  
2. Which Certificate Authority issued it?
The certificate was issued by Sectigo Limited through their intermediate CA, Sectigo Public Server Authentication CA DV E36.

3. When does it expire?
The certificate expires on June 3, 2026.

4. What domains are listed in the SAN field?
The SAN field includes github.com and www.github.com, covering both the root domain and the www subdomain.

7. What is this certificate authorized to be used for?
The certificate is authorized for TLS Web Server Authentication, as shown in the Extended Key Usage extension. The Key Usage extension permits Digital Signature operations, which are used during the TLS handshake to verify identity.
