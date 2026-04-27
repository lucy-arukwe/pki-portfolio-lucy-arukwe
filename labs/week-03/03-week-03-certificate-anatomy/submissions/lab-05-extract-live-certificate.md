# Lab 05 — Extract a Certificate from a Live Website

## Overview
This lab focused on retrieving the leaf TLS certificate from github.com using OpenSSL and inspecting its fields and extensions.The PKI concept explored is how to extract and analyze a real-world certificate directly from a live TLS connection, similar to how security engineers perform TLS troubleshooting and incident response.

---

## Environment
- OS: Windows 11  
- Terminal: Git Bash (MINGW64)  
- OpenSSL Version: OpenSSL 3.5.5 (27 Jan 2026)  
- Website: github.com  

---

## Certificate Fields Found

| Field                    | Value                                                                   |
|--------------------------|-------------------------------------------------------------------------|
| Subject                  | CN=github.com                                                           |
| Issuer                    C=GB,O=Sectigo Limited,CN=Sectigo Public Server Authentication CA DV E36 |
| Not Before               | Mar 6 00:00:00 2026 GMT                                                 |
| Not After                | Jun 3 23:59:59 2026 GMT                                                 |
| Public Key Algorithm     | id-ecPublicKey (256 bit, NIST P-256)                                    |
| Subject Alternative Name | DNS:github.com, DNS:www.github.com                                      |
| Key Usage                | Digital Signature (critical)                                            |
| Extended Key Usage       | TLS Web Server Authentication                                           |

---

## Observations

1. What organization does this certificate belong to?
The certificate belongs to GitHub, as shown in the Subject field `(CN=github.com)`, confirming it was issued to GitHub’s web server.

2. Which Certificate Authority issued it?
The issuing authority is Sectigo Limited, through the intermediate CA **Sectigo Public Server Authentication CA DV E36**, indicating a publicly trusted certificate chain.

3. When does it expire?
The validity window runs from March 6, 2026 to June 3, 2026, resulting in a short lifecycle of approximately 90 days.

4. What domains are listed in the SAN field?
The SAN field contains only two entries; github.com and www.github.com, covering both the root domain and the www subdomain.


5. The Extended Key Usage restricts the certificate to **TLS Web Server Authentication**, ensuring it is used only for HTTPS server identity validation.

---

## Key Findings

- The certificate uses ECDSA with SHA-256 and a P-256 key, aligning with modern cryptographic           standards that prioritize performance and strong security.
- The SAN field covers only two domains (github.com and www.github.com), reflecting GitHub's focused    domain structure compared to Google's wildcard-heavy approach.
- The 90-day validity period supports improved security by reducing the exposure window in the event     of key compromise.
- The Basic Constraints extension includes **CA:FALSE**, confirming that the certificate is a leaf      certificate and cannot be used to issue other certificates.

---

## Explanation

Retrieving and inspecting live certificates is a core skill in PKI operations. The fields observed in this lab tell a complete story about the certificate's identity, trust chain, permitted uses, and lifecycle. The Issuer field confirms which CA vouches for GitHub's identity, the SAN confirms which domains are covered, and the EKU confirms the certificate's intended purpose. Being able to extract this information directly from a live TLS connection,  without relying on browser UI, is essential for diagnosing TLS failures, verifying certificate deployments, and conducting security assessments.

---

## Challenges / Troubleshooting

No significant issues were encountered.

---

## Artifacts

- `github_cert.pem` — leaf certificate retrieved from github.com
