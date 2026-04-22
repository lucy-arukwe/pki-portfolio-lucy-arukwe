# Lab 01 — Enterprise Certificate Deployment Analysis

## Overview

This lab focused on analyzing a live enterprise TLS certificate deployment to understand how a large organization structures its public-facing PKI infrastructure. Rather than troubleshooting a failure, the goal was to profile a production certificate deployment by examining the certificate itself, validating the trust chain, reviewing the TLS configuration, and identifying where TLS termination occurs within the architecture.

The target selected was **bankofamerica.com**, a major financial institution with security controls and certificate practices that reflect a mature enterprise environment.

---

## Steps Performed

1. **Retrieved the certificate**  
   Connected to `bankofamerica.com:443` using OpenSSL and captured both the leaf certificate and the full certificate chain output.

2. **Parsed certificate fields**  
   Extracted and documented the validity period, Subject, Issuer, and Subject Alternative Name entries from the certificate.

3. **Analyzed the certificate chain**  
   Examined the trust path from the leaf certificate to the intermediate CA and root CA.

4. **Determined the TLS termination point**  
   Reviewed certificate details and server response headers to determine whether TLS terminates at the application server, a load balancer, or a CDN edge.

5. **Evaluated the TLS configuration**  
   Tested support for modern and deprecated TLS versions, checked for HSTS enforcement, and verified OCSP stapling status.

6. **Reviewed Certificate Transparency logs**  
   Searched public CT logs to assess issuer consistency, certificate validity patterns, and signs of unexpected certificate issuance.

---

## Results

### Step 1 — Certificate Retrieval

**Connection status:** Successful  
**Chain depth:** 3 certificates (leaf → intermediate → root)

The connection to `bankofamerica.com:443` completed successfully. The server presented a complete three-certificate chain, showing a valid trust path from the service certificate to a trusted public root.


![Connection success and certificate chain depth](../../../../assets/screenshots/week-07/lab-01/connection-success.png)

---

### Step 2 — Certificate Parsing

#### Validity Window
- **Not Before:** Sep 16 00:00:00 2025 GMT
- **Not After:** Sep 15 23:59:59 2026 GMT
- **Remaining validity:** approximately 148 days

#### Subject
- **Common Name (CN):** bankofamerica.com
- **Organization (O):** Bank of America Corporation
- **Jurisdiction:** Delaware, United States
- **Business Category:** Private Organization
- **Serial Number:** 2927442
- **Locality:** Charlotte, North Carolina

#### Issuer
- **Issuer CN:** DigiCert EV RSA CA G2
- **Known public CA:** Yes — DigiCert

#### Subject Alternative Names
- **Total SAN entries:** 1
- **DNS:** bankofamerica.com
- **Wildcard entries:** None

The SAN configuration is narrowly scoped to the main domain rather than a broad set of subdomains. This suggests a tightly controlled certificate deployment for the primary public-facing endpoint.

#### Certificate Type
**Extended Validation (EV)**

The certificate includes fields such as jurisdiction, business category, and corporate serial number, which are characteristic of EV certificates. The issuer name, **DigiCert EV RSA CA G2**, also supports this classification. This means the certificate validates the organization’s legal identity in addition to domain ownership.


![Certificate validity dates and subject fields](../../../../assets/screenshots/week-07/lab-01/cert-validity.png)


![Subject Alternative Name extension](../../../../assets/screenshots/week-07/lab-01/san-section.png)

---

### Step 3 — Certificate Chain Analysis

#### Chain Structure

**Certificate 0 — Leaf**
- **Subject:** CN=bankofamerica.com, O=Bank of America Corporation
- **Issuer:** CN=DigiCert EV RSA CA G2

**Certificate 1 — Intermediate CA**
- **Subject:** CN=DigiCert EV RSA CA G2, O=DigiCert Inc
- **Issuer:** CN=DigiCert Global Root G2

**Certificate 2 — Root CA**
- **Subject:** CN=DigiCert Global Root G2, O=DigiCert Inc
- **Issuer:** CN=DigiCert Global Root G2 (self-signed)

**Chain completeness:** Complete

The chain contains all expected layers: leaf certificate, intermediate CA, and root CA. This forms a valid and complete trust path.


![Certificate chain structure showing leaf, intermediate, and root](../../../../assets/screenshots/week-07/lab-01/chain-structure.png)


---

### Step 4 — TLS Termination Analysis

#### Server Response Headers
- **Server:** CloudFront
- **X-Cache:** FunctionGeneratedResponse from cloudfront
- **Via:** 1.1 c084db96d6570031a568083708ffe$c.cloudfront.net (CloudFront)
- **X-Amz-Cf-Pop:** YUL62-P4
- **X-Amz-Cf-Id:** VFvxIz_7TwII_9Ay_sde025SmbRmcVqbx98E8m9zxiK7qTAu3wpAMw==

#### Evidence
- Multiple CloudFront-specific headers appear in the server response.
- The header `X-Amz-Cf-Pop: YUL62-P4` identifies a CloudFront edge location in Montreal.
- The certificate is issued by DigiCert rather than by a CDN-managed certificate authority.

#### Conclusion
TLS terminates at the **CloudFront CDN edge**.

This indicates that Bank of America uses Amazon CloudFront for content delivery and edge-layer protection while still maintaining control of its own EV certificate. That approach supports both performance and brand trust.

![CloudFront headers showing CDN termination](../../../../assets/screenshots/week-07/lab-01/cdn-headers.png)

---

### Step 5 — TLS Configuration

**SSL Labs status:** Not available

Bank of America appears to restrict SSL Labs testing, which is common among financial institutions for security and compliance reasons. As a result, the TLS configuration was validated manually.

#### TLS Version Support
- **TLS 1.2:** Supported
- **TLS 1.3:** Supported
- **TLS 1.0:** Not supported
- **TLS 1.1:** Not supported

Testing showed support for only modern TLS versions. Deprecated versions are disabled.

#### HTTP Strict Transport Security (HSTS)
- **Configured:** Yes
- **Max-Age:** 31536000 seconds (1 year)

This ensures that browsers continue to use HTTPS for future connections to the site.

#### OCSP Stapling
- **Status:** Not enabled
- **Server response:** “no response sent”

Since OCSP stapling is not enabled, clients must perform separate revocation checks.

![TLS version testing and HSTS configuration](../../../../assets/screenshots/week-07/lab-01/tls-config.png)

---

### Step 6 — Certificate Transparency Log Analysis

#### Certificate Issuance History
Public CT logs showed a large number of historical certificate issuances for `bankofamerica.com` and related subdomains.

#### Issuer Consistency
- **Historical certificates (2017–2018):** Symantec Corporation
- **Current certificate (2025–2026):** DigiCert EV RSA CA G2

This reflects the CA transition that followed DigiCert’s acquisition of Symantec’s certificate business.

#### Unexpected Issuers
Certificates issued by Let’s Encrypt were found for domains such as:
- `secure-bankofamerica-login-com.tk`
- `secure-bankofamerica-login-com.ml`

These are not legitimate Bank of America domains. Their naming pattern and use of free TLDs are consistent with phishing infrastructure.

#### Validity Period Pattern
- **Historical pattern:** approximately 1-year validity
- **Current pattern:** 1-year validity (Sep 2025 to Sep 2026)

This matches a paid enterprise CA renewal model rather than a 90-day issuance cycle such as Let’s Encrypt.

![Certificate Transparency logs showing historical issuances](../../../../assets/screenshots/week-07/lab-01/ct-logs.png)

---

## Key Findings

### EV Certificate Deployment
Bank of America uses an Extended Validation certificate that includes organizational identity details such as jurisdiction, business category, and corporate serial number. This provides stronger identity assurance than a domain-only validation model.

### Complete Certificate Chain
The server presents a full three-certificate chain:
**leaf → DigiCert EV RSA CA G2 → DigiCert Global Root G2**

This allows clients to validate the trust path without needing to retrieve missing intermediates.

### CDN Architecture with Organization-Controlled Certificate
TLS terminates at CloudFront edge locations, but the certificate remains organization-managed rather than CDN-managed. This combines CDN performance benefits with direct control over certificate identity and branding.

### Modern TLS Configuration
Only TLS 1.2 and TLS 1.3 are supported. Deprecated versions such as TLS 1.0 and TLS 1.1 are disabled, which aligns with current security expectations.

### No OCSP Stapling
OCSP stapling is not enabled, so revocation checking depends on separate client-side OCSP requests.

### CA Migration History
Certificate Transparency data shows a transition from Symantec to DigiCert, reflecting long-term PKI lifecycle management and industry-wide CA changes.

### CT Logs as a Brand Protection Tool
CT logs revealed lookalike phishing domains using Let’s Encrypt certificates. This shows why CT monitoring is valuable not only for certificate inventory but also for detecting impersonation attempts.

---

## Explanation

### Why EV Certificates Matter for Financial Institutions
Extended Validation certificates provide stronger identity assurance by verifying the legal existence of the organization behind the website. For a financial institution, that level of validation supports customer trust and reduces the risk of brand impersonation.

### Why TLS Terminates at the CDN Edge
Terminating TLS at CloudFront edge locations reduces latency by serving traffic from geographically distributed infrastructure. It also strengthens resilience by absorbing malicious traffic before it reaches backend systems. In this deployment, certificate ownership remains with the organization even though TLS is terminated at the edge.

### Why TLS 1.0 and TLS 1.1 Are Disabled
Older protocol versions are no longer considered secure and are deprecated by modern standards. Disabling them ensures that connections use stronger cryptographic protections available in TLS 1.2 and TLS 1.3.

### Why HSTS Matters
The `Strict-Transport-Security` header prevents browsers from falling back to HTTP after an HTTPS connection has already been established. This helps protect against downgrade and SSL stripping attacks.

### Why OCSP Stapling Improves Efficiency
When OCSP stapling is enabled, the server includes revocation status information directly in the handshake. This reduces latency and improves privacy by removing the need for separate client requests to the OCSP responder. In this case, that optimization is not enabled.

### Why CT Log Monitoring Is Important
Certificate Transparency logs provide visibility into every certificate issued by trusted public CAs. This makes them useful for detecting unauthorized certificates, identifying phishing infrastructure, and monitoring long-term CA usage trends.

---

## Challenges / Troubleshooting

### SSL Labs Opt-Out
Bank of America does not appear to allow SSL Labs testing. This is a common restriction in financial environments. 

Manual OpenSSL testing was used instead to confirm TLS behavior.

- **TLS versions**
  - `echo | openssl s_client -connect bankofamerica.com:443 -tls1_2`
  - `echo | openssl s_client -connect bankofamerica.com:443 -tls1_3`
  - `echo | openssl s_client -connect bankofamerica.com:443 -tls1_1`
  - **Result:** TLS 1.2 and TLS 1.3 supported; deprecated versions disabled

- **HSTS**
  - `curl -sI https://bankofamerica.com | grep -i "strict-transport-security"`
  - **Result:** Configured with `max-age=31536000`

- **OCSP stapling**
  - `echo | openssl s_client -connect bankofamerica.com:443 -status`
  - **Result:** Not enabled (“no response sent”)

### Certificate Transparency Search Issues
Initial `crt.sh` searches returned server-side errors. Retrying the query resolved the issue, and alternate CT sources were available as a fallback.

### Git Authentication for Push
An early `git push` attempt failed because authentication was not configured. This was resolved by setting up Git Credential Manager and completing browser-based GitHub authorization.

---

## Artifacts

### Files Generated
- `enterprise_cert.pem` — Bank of America leaf certificate
- `full_chain_output.txt` — Full certificate chain output
- `lab-01-enterprise-certificate-analysis.md` — Final write-up

### Screenshots Captured
- `step-01-connection-success.png` — Certificate retrieval and chain depth
- `step-02-cert-validity.png` — Validity dates and Subject fields
- `step-02-san-section.png` — Subject Alternative Names
- `step-03-chain-structure.png` — Complete certificate chain
- `step-04-cdn-headers.png` — CloudFront headers showing TLS termination
- `step-05-tls-versions.png` — TLS version support testing
- `step-06-ct-logs.png` — Certificate Transparency log results

**Screenshot location:**  
`assets/screenshots/week-07/lab-01/`
