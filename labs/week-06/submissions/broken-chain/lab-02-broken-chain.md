# Lab 02 — Diagnose a Broken Certificate Chain

**Week 6 · PKI Incident Diagnosis & Troubleshooting**
**CVI PKI Career Pathway — Phase 1 Foundations**

---

## Incident Summary

**Target system:** Radiology imaging platform (simulated via incomplete-chain.badssl.com) 

**Diagnosed by:** Lucy Arukwe

**Date of diagnosis:** April 6, 2026


---

### What failed
TLS failed because the server presented only the leaf certificate and did not include the required intermediate CA certificate, preventing successful chain validation/trust chain.


---

### Evidence

The leaf certificate subject matched the expected hostname pattern, with SAN entries showing *.badssl.com and badssl.com.
The Authority Information Access extension included a CA Issuers URI: http://r13.i.lencr.org/.
OpenSSL returned: Verify return code: 21 (unable to verify the first certificate).
The error showed a chain validation failure rather than an expiration or hostname mismatch issue.

Evidence overview: 

Subject CN: *.badssl.com

Issuer CN: R13 (Let's Encrypt intermediate CA)

Verification error: unable to verify the first certificate

Verify return code: 21

Certificates sent by server: 1 (leaf only — intermediate missing)

CA Issuers URI: http://r13.i.lencr.org/

---

### Why it failed

A valid TLS connection depends on a complete certificate chain that links the server’s certificate to a trusted root authority. In this case, the server provided only the leaf certificate, which was signed by the R13 intermediate CA. Without that intermediate certificate, the client could not verify the signature on the leaf certificate or build a path to the trusted root (ISRG Root X1). Even though the certificate itself was valid and the root CA was trusted, the missing intermediate broke the chain, causing the TLS handshake to fail with error code 21: "unable to verify the first certificate."
This reinforces the Week 6 troubleshooting lesson that a certificate can appear fine on its own while the real problem is a server-side chain configuration error.


---

### Chain status

The chain was not structurally complete as presented by the server. The primary issue was a missing intermediate CA certificate, and there was no stronger evidence of a separate expiration, revocation, or hostname mismatch problem. The chain failure was therefore the direct cause of the TLS error.

The expected chain structure should have been:

- **Leaf certificate:** CN=*.badssl.com
- **Intermediate CA:** CN=R13, O=Let's Encrypt (MISSING from server configuration)
- **Root CA:** CN=ISRG Root X1 (trusted in system trust store)

Once the missing R13 intermediate was manually retrieved and provided to OpenSSL via the `-untrusted` flag, chain validation succeeded, confirming that the root CA was trusted and the chain structure was valid when complete.

---

### Remediation path

1. **Retrieve the missing intermediate certificate** — Download the R13 intermediate CA certificate from Let's Encrypt's certificate repository at http://r13.i.lencr.org/ or from https://letsencrypt.org/certificates/
2. **Convert the certificate to PEM format** Convert the certificate to PEM format if provided in DER format.
3. **Locate the server's TLS configuration file** — Identify where the radiology imaging platform stores its certificate configuration (e.g., `/etc/nginx/ssl/`, `/etc/apache2/ssl/`, or application-specific config directories)
4. **Install the intermediate certificate** — Configure the server to present the full chain (leaf + intermediate).
5. **Update the TLS configuration** to reference the complete certificate chain
6. **Restart the web server or TLS service** to apply the new configuration
7. **Test the connection** using `openssl s_client -connect [server]:443` and verify that:
   - The Certificate chain section shows 2 certificates (leaf + intermediate)
   - The Verify return code is 0 (ok)
   - No "unable to verify" errors appear
8. **Verify from a client browser** that the radiology imaging platform loads without certificate warnings

---

### Prevention
A strong preventive measure would be to include a post-renewal validation step that checks whether the server is presenting the full certificate chain, not just the leaf certificate. This would catch missing intermediates immediately after deployment before users are affected.
Automated checks using tools such as openssl s_client can detect missing intermediates before they impact users.

---

## Diagnostic Steps

### Step 1 — Retrieve

**Commands used:**

```
openssl s_client -connect incomplete-chain.badssl.com:443 -showcerts </dev/null 2>/dev/null | openssl x509 -outform PEM > leaf_cert.pem

openssl s_client -connect incomplete-chain.badssl.com:443 </dev/null 2>&1
```

**What you observed:**
The first command successfully retrieved and saved the leaf certificate to `leaf_cert.pem`. The second command revealed the chain structure sent by the server and displayed the verification failure. The Certificate chain section showed only one certificate (depth 0), confirming that the server was not sending the intermediate CA certificate. The connection output ended with `Verify return code: 21 (unable to verify the first certificate)`, indicating that the client could not validate the leaf certificate's signature because the issuing intermediate CA was missing from the presented chain.

---

### Step 2 — Parse

**Commands used:**

```
openssl x509 -in leaf_cert.pem -text -noout

openssl x509 -in leaf_cert.pem -noout -text | grep -A3 "Authority Information Access"

```

**Key fields from the certificate:**

|Field         | Value                            |
|--------------|----------------------------------|
| Subject CN   | *.badssl.com                     |
| Issuer       | C=US, O=Let's Encrypt, CN=R13    |
| Not Before   | Mar 25 21:11:49 2026 GMT         |
| Not After    | Jun 23 21:11:48 2026 GMT         |
| SAN entries  | DNS:*.badssl.com, DNS:badssl.com |



**What was found:**
The leaf certificate itself is valid — its validity period is current (expires June 23, 2026), and the Subject CN and SAN entries correctly match the target domain. The Issuer field shows that the certificate was signed by the R13 intermediate CA from Let's Encrypt. The Authority Information Access extension provided the critical clue for remediation: it contained a CA Issuers URI at `http://r13.i.lencr.org/`, which is the location where the missing intermediate certificate can be downloaded. This confirmed that the certificate is structurally sound, and the failure is purely due to the server not being configured to present the R13 intermediate alongside the leaf certificate.

---

### Step 3 — Validate the Chain

**Commands used:**

```
openssl verify leaf_cert.pem

curl -s "http://r13.i.lencr.org/" -o intermediate.der

openssl x509 -inform DER -in intermediate.der -out issuer_cert.pem

openssl verify -untrusted issuer_cert.pem leaf_cert.pem
```

**Result:**
**Before adding the intermediate:**
error 20 at 0 depth lookup: unable to get local issuer certificate

**After adding the intermediate:**
leaf_cert.pem: OK


**What you found:**

This step confirmed that the failure was caused by an incomplete certificate chain. The leaf certificate could not be validated by itself because OpenSSL did not have the missing intermediate certificate needed to build the path to a trusted root. Once the missing intermediate was supplied manually, validation succeeded, proving that the main issue was server chain configuration rather than a bad or expired leaf certificate.

---

### Step 4 — Check Revocation and Trust

**Command used:**

```
openssl s_client -connect incomplete-chain.badssl.com:443 -showcerts </dev/null 2>/dev/null | grep "Verify return code"
```

**What you found:**

The command still returns `Verify return code: 21 (unable to verify the first certificate)` indicating that the server is still not presenting the intermediate certificate. There is no indication of revocation issues. The root CA is trusted, and the failure is strictly due to incomplete chain presentation.

---

## Reflection

This lab clarified the difference between a certificate issue and a server configuration issue. The certificate itself was valid, with correct dates, proper signatures, and a trusted root CA. The failure occurred because the server did not present the full certificate chain.

Step 3 required the most attention, particularly understanding how the -untrusted flag works. The intermediate CA is not a trust anchor, but a link that allows the client to build a path from the leaf certificate to a trusted root.

This also highlighted the value of the Authority Information Access extension. It provides a reliable way to locate and retrieve the missing intermediate when the server is not configured correctly.

---

*CVI PKI Career Pathway — Foundations Phase*
