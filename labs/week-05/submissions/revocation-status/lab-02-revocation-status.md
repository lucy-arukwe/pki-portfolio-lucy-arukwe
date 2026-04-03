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

1.
2.
3.
4.
5.

---

## Results
Include the important outputs or findings from the lab.

- What was the Subject and Issuer of the leaf certificate?
- What OCSP URL did you find in the Authority Information Access extension?
- What CRL Distribution Point URL did you find?
- What was the OCSP response status for the certificate?
- What do "This Update" and "Next Update" tell you?

If you include screenshots, store them in the assets folder and reference them here:
![Description](../../assets/screenshots/week-05/your-screenshot.png)

---

## Key Findings
Document the most important observations from the lab.

•
•
•

---

## Explanation
Explain why the results matter.

- Why does an OCSP query require both the leaf certificate and the issuer certificate?
- What is the difference between OCSP and CRL in practice?
- What would happen if a system trusted a revoked certificate because OCSP was unavailable?

---

## Challenges / Troubleshooting
Document any issues encountered and how you resolved them.

---

## Artifacts
List the files generated during this lab.

- leaf_cert.pem
- issuer_cert.pem
- ocsp_response.txt
