# Lab 02 — Investigate Certificate Extensions

## Overview
This lab focused on inspecting the X.509v3 extensions of the Google leaf certificate retrieved in Lab 01.he focus was on understanding how extensions define and restrict certificate behavior, including which domains it covers, what cryptographic operations it permits, and whether it can act as a Certificate Authority.
---

## Environment
- OS: Windows 11  
- Terminal: Git Bash (MINGW64)  
- OpenSSL Version: OpenSSL 3.5.5 (27 Jan 2026)

---

## Extensions Found

### Subject Alternative Name (SAN)
DNS:*.google.com, DNS:*.appengine.google.com, DNS:*.bdn.dev, DNS:*.origin-test.bdn.dev, DNS:*.cloud.google.com, DNS:*.crowdsource.google.com, DNS:*.datacompute.google.com, 
DNS:*.google.ca, DNS:*.google.cl, DNS:*.google.co.in, DNS:*.google.co.jp, DNS:*.google.co.uk, DNS:*.google.com.ar, DNS:*.google.com.au, DNS:*.google.com.br, DNS:*.google.com.co, DNS:*.google.com.mx, 
DNS:google.com, DNS:*.youtube.com, DNS:youtube.com (and many more Google-owned domains)

### Key Usage
Digital Signature (critical)

### Extended Key Usage (EKU)
TLS Web Server Authentication

### Basic Constraints
CA:FALSE (critical)

---

## Observations

1. The SAN extension contains a large number of Google-owned domains, including multiple wildcard entries. This allows a single certificate to secure a wide range of services across Google’s infrastructure.

3. Key Usage is restricted to Digital Signature only, indicating that the certificate is used for authentication and integrity rather than key encipherment operations.

4. The Extended Key Usage field restricts the certificate to TLS Web Server Authentication. This ensures it is only valid for securing HTTPS connections and cannot

  be reused for other purposes such as code signing or email protection.

5. The Basic Constraints extension explicitly sets CA:FALSE, confirming that this certificate is a leaf certificate and cannot be used to issue other certificates.

7. These extensions collectively define how the certificate is validated during a TLS handshake. The SAN ensures hostname matching, EKU confirms the certificate is valid
   for server authentication, and Basic Constraints enforces its role within the trust hierarchy.

---

## Key Findings

- The extensive SAN list, including wildcard entries, enables Google to secure a large number of domains and services using a single certificate, supporting scalability
  in a distributed environment.
- Critical extensions such as Key Usage and Basic Constraints enforce strict validation rules. Any client that does not understand these extensions must reject the certificate.
- The CA:FALSE constraint is a key security control that prevents a leaf certificate from being used to issue other certificates, protecting the integrity of the PKI trust model.
- Restricting EKU to TLS Web Server Authentication limits the certificate’s use to its intended purpose, reducing the risk of misuse across different security contexts.

---

## Explanation

Certificate extensions define the operational boundaries of an X.509 certificate. The SAN extension serves as the authoritative source for domain validation, replacing the legacy reliance on the Common Name field. 
Modern clients require SAN to perform hostname verification.

Key Usage and Extended Key Usage work together as a layered control mechanism. Key Usage specifies the underlying cryptographic functions permitted by the certificate, 
while EKU defines the application-level contexts in which the certificate can be used.

The Basic Constraints extension establishes whether a certificate can act as a Certificate Authority. Setting CA:FALSE ensures that the certificate remains a leaf 
certificate, preventing it from being used to create unauthorized certificate chains.

Together, these extensions transform the certificate from a simple identity record into a tightly controlled security object that can only be used within clearly defined 
boundaries.

---

## Challenges / Troubleshooting

The `leaf_cert.pem` file initially produced no output when parsed. This was traced to a corrupted certificate file following a reset. The issue was resolved by 
re-retrieving the certificate using a complete `openssl s_client` output and extracting the leaf certificate cleanly using `openssl x509`.

---

## Artifacts

- `leaf_cert.pem` — reused from Lab 01, containing the X.509 certificate retrieved from google.com
