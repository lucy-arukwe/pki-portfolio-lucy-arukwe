# Lab 03 — Simulate a Certificate Expiration Scenario

## Overview
This lab focused on understanding how certificate expiration works in practice, from detection to failure and 
replacement. 
Short-lived and expired certificates were created to simulate real scenarios, and OpenSSL tools were used to 
check validity and confirm expiration.

The core PKI concept explored was certificate validity and expiration handling, showing how systems detect expired 
certificates and why timely replacement is necessary to avoid service disruption.

---

## Environment
- Operating System: `Windows 11`
- Terminal Used: `Git Bash (MINGW64)`
- OpenSSL Version: `OpenSSL 3.5.5 27 Jan 2026 (Library: OpenSSL 3.5.5 27 Jan 2026)`

---

## Steps Performed
1. Created a short-lived certificate by generating a private key and CSR, then issuing a self-signed certificate with a 1-day validity using the `-days 1` flag.
   This simulated a certificate approaching expiration.
2. Inspected the validity window by using `openssl x509 -dates` to read the `notBefore` and `notAfter` fields.This confirmed the certificate would expire in 24 hours.
3. Tested expiration detection using:
   openssl x509 -in test_cert_short.pem -checkend 3600  
   openssl x509 -in test_cert_short.pem -checkend 86400  
   This confirmed whether the certificate would expire within specific time thresholds.
4. Created an already-expired certificate by setting a past validity window (January 1–2, 2023) using the `-not_before` and `-not_after` flags to simulate an
   expiration scenario.
5. Verified the expired certificate using `openssl verify`, which returned the "certificate has expired" error (Error 10).

6. Generated a replacement certificate by creating a new private key, generating a new CSR, and issuing a certificate with a 365-day validity period.
   This demonstrated the difference between renewal (same key) and replacement (new key).
 
---

## Results
Include the important outputs or findings from the lab.

- The short-lived certificate had the following validity window:
  notBefore=Mar 31 23:37:29 2026 GMT
  notAfter=Apr 1 23:37:29 2026 GMT
  
- For the short-lived certificate:
    `-checkend 3600` → Certificate will not expire (within 1 hour)
    `-checkend 86400` → Certificate will expire (within 24 hours)
- Verification Error for Expired Certificate:

      error 18 at 0 depth lookup: self-signed certificate
      error 10 at 0 depth lookup: certificate has expired
      error labs/week-05/submissions/expiration-stretch/test_cert_expired.pem: verification failed
  The important part here is **"certificate has expired"**, which causes the verification to fail.
  
- What changed in the replacement certificate compared to the expired one?
  Replacement Certificate Changes

Expired certificate:

notBefore=Jan 1 00:00:00 2023 GMT
notAfter=Jan 2 00:00:00 2023 GMT


The replacement certificate had:
- A new validity period
- A new serial number
- A new key

This shows that the expired certificate could not be reused and had to be replaced with a new one.


If you include screenshots, store them in the assets folder and reference them here:
![Generate Key and CSR](../../assets/screenshots/lab03-generate-key-and-csr.png)
![Short Certificate Creation](../../assets/screenshots/lab03-short-cert-creation.png)
![Short Certificate Dates](../../assets/screenshots/lab03-short-cert-dates.png)
![Check Expiration Tests](../../assets/screenshots/lab03-checkend-tests.png)
![Create Expired Certificate](../../assets/screenshots/lab03-expired-cert-creation.png)
![Expired Certificate Dates](../../assets/screenshots/lab03-expired-cert-dates.png)
![Expired Certificate Verification Error](../../assets/screenshots/lab03-expired-cert-verify-error.png)
![Replacement Key Generation](../../assets/screenshots/lab03-replacement-key-generation.png)
![Replacement CSR Creation](../../assets/screenshots/lab03-replacement-csr-creation.png)

---

## Key Findings

• Expiration is strict. Once the `notAfter` date passes, the certificate is no longer trusted and will be rejected.

• `-checkend` supports proactive monitoring. It can be used to detect certificates that are close 
to expiring and allow time for replacement before issues occur.

• Expired certificates fail verification. When a certificate expires, validation fails and secure connections can no longer be established.

• Replacement requires a new key. A new certificate is issued with a new key pair, which improves security compared to reusing an old key.

---

## Explanation
Why the results matter.
Certificate expiration is predictable, but outages still occur when certificates are not properly tracked or monitored. 
Without visibility into expiration dates, certificates can expire unexpectedly and cause service disruption.

Renewal and replacement serve different purposes. Renewal extends the validity of an existing certificate, usually with the same key, 
while replacement involves generating a new key and issuing a new certificate. 
Replacement is preferred when stronger security is needed or when the old key should no longer be trusted.

The `openssl x509 -checkend` command can be used in monitoring scripts to check if a certificate will expire within a specific time window. 
This allows alerts to be triggered in advance so that certificates can be updated before they expire.

Certificate inventory refers to maintaining a record of all certificates in use, including their locations, expiration dates, and ownership. 
At enterprise scale, this is necessary to prevent unexpected outages and to manage certificates efficiently across multiple systems.


---

## Challenges / Troubleshooting

Multi-line commands in Git Bash:

While generating keys and certificates, multi-line OpenSSL commands using backslashes (`\`) were initially used to improve readability. However, in Git Bash on Windows, 
this caused syntax errors and prevented the commands from running.

Running the same commands on a single line resolved the issue. For example:

`bash
openssl genrsa -out test_key.pem 2048`

The interactive CSR method (openssl req -new) was also used as an alternative, which prompted for input step-by-step and worked reliably.

Date flags not recognized:

While attempting to create an already-expired certificate, the original approach used the -startdate and -enddate flags. 
These were not supported in the OpenSSL version used, which caused the command to fail.

To resolve this, the -not_before and -not_after flags were used instead:
`openssl x509 -req -in test_csr.pem -signkey test_key.pem -out test_cert_expired.pem -set_serial 02 -not_before 20230101000000Z -not_after 20230102000000Z`

This successfully created a certificate with a past validity window, allowing the expiration scenario to be tested.

---

## Artifacts

- test_cert_short.pem
- test_cert_expired.pem
- test_cert_replacement.pem
