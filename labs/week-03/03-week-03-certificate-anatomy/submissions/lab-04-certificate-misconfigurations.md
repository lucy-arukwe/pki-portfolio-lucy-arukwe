# Lab 04 — Detect Certificate Misconfigurations

## Overview
This lab focused on analyzing common certificate misconfiguration scenarios and explaining why each would fail TLS validation in a production environment. Rather than executing commands, the objective was to apply knowledge of certificate fields and extensions to reason through real-world failure cases.

---

## Environment
- OS: Windows 11  
- Terminal: Git Bash (MINGW64)  
- OpenSSL Version: OpenSSL 3.5.5 (27 Jan 2026)  

---

## Scenario 1 — Missing Subject Alternative Name

**Would modern browsers trust this certificate?**  
No. Modern browsers would reject the certificate and display a security warning.

**Analysis:**  
The Subject Alternative Name (SAN) extension is required for domain validation in modern TLS implementations. Earlier approaches relied on the Common Name 
(CN) field, but this method was deprecated due to inconsistent behavior and security limitations. Modern browsers such as Chrome and Firefox no longer use the CN 
for hostname validation and require SAN to be present.

A certificate containing only `CN=example.com` without a SAN extension cannot be validated against the requested domain. As a result, the connection fails, 
and users are presented with errors such as **"NET::ERR_CERT_COMMON_NAME_INVALID"** or **"Your connection is not private."**

---

## Scenario 2 — Incorrect Extended Key Usage

**Would a browser accept this certificate for a web server?**  
No. The certificate would be rejected during the TLS handshake.

**Analysis:**  
The Extended Key Usage (EKU) extension defines the allowed application contexts for a certificate. For HTTPS, the certificate must include 
**TLS Web Server Authentication** (OID 1.3.6.1.5.5.7.3.1).

A certificate that includes only **Client Authentication** is intended for authenticating users or systems to a server, not for identifying a server to a client. 
During the TLS handshake, the browser validates that the certificate is authorized for server authentication. If the required EKU value is missing, 
the connection is rejected, often with errors such as **"ERR_SSL_SERVER_CERT_BAD_FORMAT"** or similar TLS failures.

---

## Scenario 3 — Expired Certificate

**What happens if this certificate is used today?**  
The TLS handshake fails, and the browser blocks the connection.

**Analysis:**  
Certificate validity is strictly enforced during TLS validation. The client checks whether the current date falls within the certificate’s 
**Not Before** and **Not After** range. If the certificate has expired, it is no longer trusted regardless of its cryptographic integrity.

Expiration limits the risk window associated with key compromise and enforces regular certificate renewal. Without proper lifecycle management, 
expired certificates can lead to service outages. Users will typically see errors such as **"NET::ERR_CERT_DATE_INVALID"** along with security warnings in the browser.

---

## Scenario 4 — Missing Intermediate Certificate

**What error would a browser likely display?**  
The browser would display a trust chain error such as **"NET::ERR_CERT_AUTHORITY_INVALID"** or **"SEC_ERROR_UNKNOWN_ISSUER."**

**Analysis:**  
Certificate validation depends on the ability to build a complete trust chain from the server certificate to a trusted root CA. If the intermediate certificate 
is not provided during the TLS handshake, the client cannot establish this chain.

Servers are responsible for including all intermediate certificates. While some browsers may attempt to retrieve missing intermediates using Authority Information 
Access (AIA), this behavior is not consistent and should not be relied upon. Many clients, particularly mobile or non-browser systems, will fail immediately if the chain is incomplete.

---

## Key Takeaway

Certificate misconfigurations are a common cause of PKI-related outages because validation depends not only on cryptographic correctness but also on proper configuration 
of fields, extensions, and trust chains. Issues such as missing SAN entries, incorrect EKU values, expired certificates, or incomplete chains can all lead to immediate 
connection failures. Effective certificate management requires consistent validation, monitoring, and lifecycle control to prevent these avoidable failures.

---

## Artifacts
- `lab-04-certificate-misconfigurations.md` — analysis of certificate misconfiguration scenarios
