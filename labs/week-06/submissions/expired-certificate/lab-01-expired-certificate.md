# Lab 01 — Diagnose an Expired Certificate

**Week 6 · PKI Incident Diagnosis & Troubleshooting**
**CVI PKI Career Pathway — Phase 1 Foundations**

---

## Incident Summary

**Target system:** portal.metrogeneral.org (simulated via expired.badssl.com)
**Diagnosed by:** Lucy Arukwe
**Date of diagnosis:** April 5, 2026

---

### What failed

The TLS connection failed because the certificate had expired and was no longer within its valid date range.

---

### Evidence
- The certificate validity period showed **Not Before: Apr 9 00:00:00 2015 GMT** and **Not After: Apr 12 23:59:59 2015 GMT**. 
- The certificate was no longer within its validity window at the time of testing, meaning it had been expired for **4013 days** as of April 7, 2026.
- OpenSSL reported a verification failure during the TLS connection, indicating that the certificate had expired.
- The certificate chain appeared structurally present enough for diagnosis, so the primary issue was expiration rather than a missing intermediate or broken chain.
- An OCSP responder URL was present in the certificate, but revocation was not the primary cause of the failure.

`OVERVIEW`
- **Not After date:** Apr 12 23:59:59 2015 GMT
- **Current date:** April 5, 2026
- **Days expired:** Approximately 4,011 days (over 10 years)
- **Verification error:** `certificate has expired`
- **Chain status:** Structurally complete (3 certificates: leaf → intermediate → root)

---

### Why it failed

X.509 certificates contain a Validity field with Not Before and Not After timestamps that define the certificate's operational window. This certificate's Not After date was April 12, 2015, meaning the certificate expired over 10 years ago. When a client attempts to establish a TLS connection, it validates the certificate's validity period against the current system time. Because the current date falls outside the certificate's validity window, the client rejects the certificate and terminates the TLS handshake. This connects directly to the certificate lifecycle concepts from earlier weeks: expired certificates cannot be trusted because the issuing CA no longer vouches for the binding between the public key and the subject identity.

---

### Chain status

The chain is complete and correctly structured:

* Leaf certificate: `*.badssl.com`
* Intermediate: COMODO RSA Domain Validation Secure Server CA
* Root CA: COMODO CA Limited

There are no issues with the chain itself. The failure is caused entirely by the expired leaf certificate.


---

### Remediation path

1. Confirm that the certificate currently installed on the server is expired.
2. Generate a new private key if a fresh key pair is required.
3. Create a new Certificate Signing Request (CSR) for the correct hostname.
4. Submit the CSR to a trusted Certificate Authority.
5. Complete the required validation process with the CA.
6. Review the new certificate to confirm the subject, SAN entries, and validity dates are correct.
7. Install the replacement certificate on the web server.
8. Update the TLS configuration so the server uses the new certificate.
9. Test the connection with `openssl s_client` to confirm the certificate is now valid.
10. Verify in a browser that the site loads without certificate warnings.
11. Document the incident and update certificate inventory records.
12. Review renewal ownership and alerting to reduce the risk of recurrence.
---

### Prevention
Implement automated certificate lifecycle monitoring that alerts the IT team at 60, 30, and 7 days before expiration. For critical systems, automated renewal (such as ACME-based workflows) can prevent this type of issue entirely.


---

## Diagnostic Steps

Document each step of the PKI Diagnostic Framework as you worked through it.

### Step 1 — Retrieve

**Command used:** 

```
openssl s_client -connect expired.badssl.com:443 -showcerts </dev/null 2>/dev/null | openssl x509 -outform PEM > expired_cert.pem
```

**What you observed:**

The certificate was successfully retrieved. Running the connection without suppressing errors showed `certificate has expired`, confirming the failure during the TLS handshake.

---

### Step 2 — Parse

**Command used:**

```
openssl x509 -in expired_cert.pem -text -noout
openssl x509 -in expired_cert.pem -noout -dates
```

**Key fields from the certificate:**

| Field       | Value                                         |
| ----------- | --------------------------------------------- |
| Subject CN  | *.badssl.com                                  |
| Issuer      | COMODO RSA Domain Validation Secure Server CA |
| Not Before  | Apr 9 00:00:00 2015 GMT                       |
| Not After   | Apr 12 23:59:59 2015 GMT                      |
| SAN entries | DNS:*.badssl.com, DNS:badssl.com              |


**What you found:**

[What the parsed certificate told you about the failure]
The certificate's validity period ended on April 12, 2015 — over 10 years before the current date.
Parsing the certificate confirmed that the leaf certificate was expired. The **Not After** date had already passed, so the certificate was outside its allowed validity window. That directly explained why browsers and TLS clients rejected the connection. The subject and SAN values were relevant to the expected hostname pattern, so the major issue was not hostname mismatch (Identity) or chain structure, but certificate expiration. The issuer was a recognized CA, which further supported the conclusion that the failure was lifecycle-related rather than caused by self-signing or an obviously untrusted issuer.

---

### Step 3 — Validate the Chain

**Command used:**

```
openssl s_client -connect expired.badssl.com:443 -showcerts </dev/null 2>/dev/null
```

**Result:**

Chain is structurally complete. Three certificates were present in the TLS handshake:
1. Server certificate (*.badssl.com)
2. Intermediate CA certificate (COMODO RSA Domain Validation Secure Server CA)
3. Root CA certificate (COMODO CA Limited)
Error recorded: `verify error: num:10:certificate has expired`
Which means primary validation error was certificate expiration.


**What you found:**

The certificate chain itself is intact, there are no missing intermediates or broken trust paths. The only error reported was the expiration failure. This confirms that the remediation can focus solely on replacing the expired leaf certificate without needing to address chain construction issues.

---

### Step 4 — Check Revocation and Trust

**Command used:**

```
openssl x509 -in expired_cert.pem -noout -text | grep -A1 "OCSP"
```

**What you found:**
An OCSP responder URL was present in the certificate. Revocation checking is still part of certificate validation in general, but in this incident it was not the main remediation concern. Even if the certificate were not revoked, it would still fail because it had already expired. That means the immediate restoration path is certificate replacement, not revocation troubleshooting.

---

## Reflection

This lab reinforced the importance of the PKI Diagnostic Framework's systematic approach, instead of jumping to assumptions.
The certificate had an obvious problem, but working through each step still helped rule out other causes such as chain failure or revocation. 
The most important point in this lab came in Step 3, where the difference between a complete chain and a valid certificate became clear. The chain itself was intact, but the expired leaf certificate still caused the connection to fail. This made it clear that expiration does not break the chain structure — it simply makes the certificate unusable. As a result, the correct fix is to replace the certificate, not to repair the chain.

---

*CVI PKI Career Pathway — Foundations Phase*
