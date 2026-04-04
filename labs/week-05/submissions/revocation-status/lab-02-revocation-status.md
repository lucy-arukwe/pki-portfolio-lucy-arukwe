# Lab 02 — Check Certificate Revocation Status with OCSP 

## Overview
This lab focused on understanding how certificate revocation is verified in real-world PKI systems. A live certificate was retrieved from a production website, its trust chain was examined, the OCSP responder URL was extracted from the certificate extensions, and the responder was queried to verify the certificate’s current revocation status.

The core PKI concept being investigated was certificate revocation checking via OCSP (Online Certificate Status Protocol), specifically how systems verify that a certificate has not been revoked before trusting it, even if it has not yet expired. This reinforces the idea that validity alone does not guarantee trust.

This work was carried out over a one week period, allowing for a deeper understanding of both the structure of certificates and how revocation mechanisms operate in practice.

This hands-on experience demonstrated that certificate trust is dynamic: it depends not only on cryptographic signatures and validity dates, but also on whether the certificate authority has actively revoked the certificate due to compromise, misuse, or other security concerns. This highlights how PKI systems continuously evaluate trust rather than assuming it is permanent.

---

## Environment
- Operating System: Windows 11
- Terminal Used: Git Bash (MINGW64)
- OpenSSL Version: `OpenSSL 3.5.5 27 Jan 2026 (Library: OpenSSL 3.5.5 27 Jan 2026)`
- Target site used: github.com

---

## Steps Performed

1. Connected to github.com using `openssl s\_client -connect github.com:443`, which establishes a TLS connection and outputs the server’s certificate.
2. Verified the leaf certificate by inspecting key fields such as Subject (CN=github.com), Issuer (Sectigo CA), and validity dates to confirm it was correctly retrieved
3. Retrieved the full certificate chain using `openssl s\_client -showcerts`, which displays all certificates in the trust chain (leaf, intermediate CA, and root CA).
4. Extracted the issuer certificate by copying the second certificate block from the chain output and saving it as `issuer\_cert.pem`, since this intermediate certificate is required for OCSP queries.
5. Located the OCSP responder URL by inspecting the Authority Information Access extension within the leaf certificate.
6. Located the OCSP responder URL by inspecting the Authority Information Access extension within the leaf certificate.
Extracted the OCSP URL programmatically using `grep` and `sed`, allowing the value to be stored and reused in the query.

7. Queried the OCSP responder using `openssl ocsp` with both the leaf and issuer certificates, sending the request to `http://ocsp.sectigo.com` to check revocation status.

8. Captured and analyzed the OCSP response, which returned `Cert Status: good`, confirming that the certificate has not been revoked.
---

## Results

- What was the Subject and Issuer of the leaf certificate?
`Subject: CN=github.com
Issuer: CN=Sectigo Public Server Authentication CA DV E36`
The leaf certificate represents the website (github.com), while the issuer certificate represents the Certificate Authority that signed it. This shows that the certificate was not self-signed, but issued as part of a trust chain.
- What OCSP URL did you find in the Authority Information Access extension?
The OCSP responder URL found in the certificate is http://ocsp.sectigo.com.
- What CRL Distribution Point URL did you find?
No CRL Distribution Point URL was found in the certificate. This suggests that revocation checking for this certificate relies primarily on OCSP, which is commonly used in modern PKI for faster, real-time status checks.
- What was the OCSP response status for the certificate?
OCSP Response Status: successful
Cert Status: good
This Update: Mar 29 22:10:11 2026 GMT
Next Update: Apr 5 22:10:10 2026 GMT
What this means:
   `Status: good = Certificate has not been revoked`
- What do "This Update" and "Next Update" tell you?
"This Update" indicates when the OCSP response was generated.
"Next Update" shows how long the response can be trusted before checking again.


If you include screenshots, store them in the assets folder and reference them here:
![Description](../../assets/screenshots/week-05/your-screenshot.png)

---

## Key Findings

• Certificate trust is not based on just one factor. Even if a certificate is valid and correctly signed, it can still be revoked. All checks (signature, expiration, and revocation) must pass for it to be trusted.

• OCSP provides real-time status. Instead of checking a full list like CRL, it returns the current status of a single certificate when needed.

• OCSP responses can be cached. The "This Update" and "Next Update" values show how long the result can be reused before checking again.

• The issuer certificate is required for OCSP. It helps identify which Certificate Authority issued the certificate before its status can be checked.

• Not all certificates use CRL. Many modern certificates rely mainly on OCSP, especially for faster and more efficient validation.

---

## Explanation
Explain why the results matter.

- Why does an OCSP query require both the leaf certificate and the issuer certificate?

The OCSP responder uses both certificates to identify the certificate being checked.  
The leaf certificate provides the serial number, while the issuer certificate identifies the Certificate Authority that issued it.  

Without the issuer certificate, the system cannot determine which CA to query or properly verify the OCSP response.

- What is the difference between OCSP and CRL in practice?
OCSP:
- Checks one certificate at a time
- Real-time query/response
- Small and fast
- Best for web browsers checking individual sites

CRL:
- List of ALL revoked certificates from a CA
- Downloaded as a file
- Can be large (megabytes)
- Better for checking many certificates offline

Think of it like checking if your driver's license is suspended: OCSP is calling the DMV to ask about your specific license. CRL is downloading the entire list of suspended licenses and searching through it yourself.
For high-traffic systems, OCSP is generally preferred because it is faster and avoids downloading large lists.

- What would happen if a system trusted a revoked certificate because OCSP was unavailable?
Most browsers use `soft fail`, if they can't reach the OCSP responder, they allow the connection anyway. This keeps the internet working when OCSP servers have outages, but it means a revoked certificate might be accepted if OCSP is down.

High-security systems can use `hard fail`,  reject the certificate if OCSP can't be checked. This is more secure but can block legitimate connections.

If a system trusts a revoked certificate because OCSP is unavailable, it usually means a `soft fail` occurred. In this case, the system allows the connection to continue to avoid disruption, even though it introduces a security risk.

A `hard fail` would block the connection if the certificate status cannot be verified, which is more secure but can cause legitimate services to become unavailable.

Organizations may run their own OCSP responder to improve performance, maintain control, and protect privacy, especially in internal or high-security environments.

---

## Challenges / Troubleshooting
1. Extracting the issuer certificate:
The full chain output contained multiple certificates. I used nano to open the file, identified the second certificate block (the intermediate CA), and copied it to a separate file.

2. No CRL found:
After inspecting the certificate, no CRL Distribution Point was listed. This is intentional, Sectigo configured this certificate to use OCSP only, which is becoming the standard approach.


---

## Artifacts

- leaf_cert.pem
- issuer_cert.pem
- ocsp_response.txt
