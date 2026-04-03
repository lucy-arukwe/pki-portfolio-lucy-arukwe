# Lab 02 — Check Certificate Revocation Status with OCSP

## Overview
Briefly describe what this lab was about in your own words.
What PKI concept or system behavior were you investigating?

---

## Environment
- Operating System:
- Terminal Used:
- OpenSSL Version (`openssl version`):
- Target site used:

---

## Steps Performed
Summarize the key steps you performed to complete the lab.
Do not copy the lab instructions — describe what you actually did.

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
