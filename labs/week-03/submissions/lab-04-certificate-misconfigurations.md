# Lab 04 — Detect Certificate Misconfigurations

## Overview
This lab involved analyzing four common certificate misconfiguration scenarios and explaining why each would cause TLS validation to fail. The PKI concept explored is how certificate fields and extensions must be correctly configured for browsers and clients to establish trust in a TLS connection.

---

## Scenario 1 — Missing Subject Alternative Name

**Would modern browsers trust this certificate?**
No. Modern browsers will reject this certificate even if the CN matches the domain.

**Analysis:**
[Explain why SAN is required, why CN is not sufficient, and what error users would see]

Modern browsers require the Subject Alternative Name (SAN) extension to validate which domains a certificate is valid for. The Common Name (CN) field was historically used for this purpose, but reliance on CN alone has been deprecated. Without a SAN extension, browsers cannot confirm that the certificate matches the requested domain, even if the CN appears correct.  

A user would likely see an error such as "Your connection is not private" or a hostname mismatch error like "NET::ERR_CERT_COMMON_NAME_INVALID"
---
 
## Scenario 2 — Incorrect Extended Key Usage

**Would a browser accept this certificate for a web server?**
No. A browser would reject this certificate for HTTPS use.

**Analysis:**
[Explain what EKU defines, what value is required for HTTPS, and what error users would see]
Extended Key Usage (EKU) defines the specific purpose a certificate is allowed to serve. For HTTPS, the certificate must include "TLS Web Server Authentication". A certificate that only includes "Client Authentication" is intended for identifying clients to a server, not for identifying a server to users.  
Using it on a web server would result in a certificate usage error, and the browser would reject the connection as untrusted.
---

## Scenario 3 — Expired Certificate

**What happens if this certificate is used today?**
The TLS connection will fail immediately because browsers reject expired certificates.

**Analysis:**
[Explain why expiration fails validation, why lifecycle management matters, and what users would see]
Certificates are only valid within a defined time period. Once the "Not After" date has passed, the certificate is no longer trusted, even if everything else is correct. This is a strict requirement in TLS validation.  

This is why certificate lifecycle management is critical — certificates must be monitored and renewed before they expire. Failure to do so can cause sudden service outages.  
A user would typically see an error such as "Your connection is not private" with a message indicating that the certificate has expired.

---

## Scenario 4 — Missing Intermediate Certificate

**What error would a browser likely display?**
The browser would display a trust error such as "ERR_CERT_AUTHORITY_INVALID" or "SEC_ERROR_UNKNOWN_ISSUER".

**Analysis:**
[Explain how chains establish trust, why servers must include intermediates, and why browser behavior varies]
Certificate chains establish trust by linking the server certificate to a trusted root CA through intermediate certificates. If the intermediate certificate is missing, the browser cannot complete the chain of trust and therefore cannot verify the certificate.  

Servers are required to provide the full certificate chain (excluding the root). Without the intermediate, validation fails. Some browsers, like Chrome, may attempt to retrieve the missing intermediate automatically using AIA (Authority Information Access), while others, like Firefox, will fail immediately. This difference explains why behavior can vary across browsers.

---

## Key Takeaway
In 2-3 sentences, explain why certificate misconfiguration is one of the most common causes of PKI outages.

Certificate misconfiguration is one of the most common causes of PKI outages because multiple components — extensions, validity periods, and certificate chains — must all be correctly configured at the same time. A single issue, such as an expired certificate, missing extension, or incomplete chain, can break trust and disrupt service. These types of errors occur frequently in real-world environments and often only become visible when users encounter browser warnings.
