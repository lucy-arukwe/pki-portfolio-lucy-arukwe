# Lab 03 — Diagnose a Hostname and SAN Mismatch

**Week 6 · PKI Incident Diagnosis & Troubleshooting**
**CVI PKI Career Pathway — Phase 1 Foundations**

---

## Incident Summary

**Target system:** Staff scheduling portal (simulated via wrong.host.badssl.com)

**Diagnosed by:** Lucy Arukwe

**Date of diagnosis:** April 6, 2026

---

### What failed
TLS failed because the hostname being accessed did not match the identities listed in the certificate’s Subject Alternative Name (SAN). The wrong.host.badssl.com test site is specifically designed to demonstrate a hostname mismatch, and browsers reject it for that reason.

---

### Evidence


- **Hostname accessed:** wrong.host.badssl.com
- **Subject CN:** *.badssl.com
- **SAN entries:** DNS:*.badssl.com, DNS:badssl.com
- **Mismatch:** The hostname `wrong.host.badssl.com` is a two-level subdomain (wrong.host), but the wildcard certificate `*.badssl.com` only matches single-level subdomains. The SAN does not explicitly list `wrong.host.badssl.com`, resulting in a hostname validation failure.
- **OpenSSL verify return code:** 0 (ok) — indicating the certificate chain is valid, but OpenSSL's command-line tool does not enforce hostname validation by default


---

### Why it failed
Modern TLS clients validate the hostname against the SAN, not just the Subject CN. The certificate presented on this site is valid for *.badssl.com and badssl.com, but that wildcard only covers one subdomain level. It matches names like test.badssl.com, but it does not match wrong.host.badssl.com, which has an extra label. This is a cryptographic identity mismatch, the CA signed the certificate for specific hostnames, and those signatures cannot be transferred to a different hostname. Because the hostname is not listed in the SAN, the client cannot confirm the server’s identity, and the connection is rejected.

---

### Chain status

The certificate chain is structurally valid and complete. The server presents the full chain (leaf, intermediate, and root), and the connection passes standard trust checks. The failure is not related to chain integrity or trust but is strictly caused by a mismatch between the requested hostname and the certificate’s SAN.

---

### Remediation path
1. **Generate a new Certificate Signing Request (CSR)** for the correct hostname (`staff.metrogeneral.org`)
2. **Submit the CSR to a trusted Certificate Authority** for issuance
3. **Complete domain validation** as required by the CA (DNS, HTTP, or email validation)
4. **Receive the new certificate** with the correct SAN entries (must include `DNS:staff.metrogeneral.org`)
5. **Install the new certificate** on the staff scheduling portal server, replacing the old certificate for `scheduling.metrogeneral.org`
6. **Update the web server configuration** to reference the new certificate and private key
7. **Restart the web server** to apply the new configuration
8. **Test the connection** from multiple browsers to verify that `staff.metrogeneral.org` loads without certificate warnings
9. **Update internal documentation** to reflect the new hostname and certificate details


A DNS CNAME only changes how the name is resolved behind the scenes. It does not change the hostname the browser believes it is connecting to. If a user types staff.metrogeneral.org, the browser still validates the certificate against staff.metrogeneral.org, even if DNS points that name somewhere else. If that hostname is not listed in the certificate SAN, the warning remains. The only real fix is a certificate that includes the correct hostname.

In non-technical terms: Adding a CNAME is like calling someone by a nickname, but their ID still shows their real name. If the names don’t match, you’re not let in. The certificate is the ID, and it has to match the exact name being used.

---

### Prevention
Any change to service hostnames should trigger a certificate review before deployment. Certificates must be updated alongside DNS changes to ensure that all active hostnames are included in the SAN. A simple validation step before going live can prevent this type of failure.


---

## Diagnostic Steps

Using the PKI Diagnostic Framework

### Step 1 — Retrieve

**Command(s) used:**

```
openssl s_client -connect wrong.host.badssl.com:443 -servername wrong.host.badssl.com </dev/null 2>/dev/null | openssl x509 -outform PEM > mismatch_cert.pem

openssl s_client -connect wrong.host.badssl.com:443 -servername wrong.host.badssl.com </dev/null 2>&1

```

**What you observed:**

The certificate was successfully retrieved and saved as mismatch_cert.pem. The connection output ended with Verify return code: 0 (ok), showing that the connection passed trust and chain checks in that context. This made it clear that the problem was not simple expiration or an obviously broken chain.

---

### Step 2 — Parse

**Command(s) used:**

```
openssl x509 -in mismatch_cert.pem -text -noout

openssl x509 -in mismatch_cert.pem -noout -text | grep -A5 "Subject Alternative Name"

```

**Key fields from the certificate:**

| Field       | Value                            |     
|-------------|----------------------------------|
| Subject CN  | *.badssl.com |
| Issuer      | C=US, O=Let's Encrypt, CN=R13    |
| Not Before  | Mar 24 20:02:52 2026 GMT         |
| Not After   | Jun 22 20:02:51 2026 GMT         |
| SAN entries | DNS:*.badssl.com, DNS:badssl.com |



**What you found:**

The certificate is valid and correctly signed. However, the SAN does not include the hostname being accessed. The wildcard only covers single-level subdomains and does not apply to wrong.host.badssl.com. This confirms that the failure is caused by a hostname mismatch.

---

### Step 3 — Validate the Chain

**Command used:**

```
openssl verify mismatch_cert.pem
```

**Result:**

`error 20 at 0 depth lookup: unable to get local issuer certificate`  
`error mismatch_cert.pem: verification failed`


**What you found:**
The openssl verify command failed because the R13 intermediate CA certificate was not available in the local system trust store. This does not indicate a problem with the server’s configuration.

The connection output from Step 1 showed that the server presents a complete certificate chain (leaf → R13 intermediate → ISRG Root X1). The verification failure reflects a limitation of the local environment, not an issue with the chain itself.

The key finding is that the certificate chain is structurally valid and correctly configured on the server. This confirms that the failure is not related to chain construction. The certificate is valid, the signatures are correct, and the root CA is trusted. The issue is a hostname mismatch — the certificate is being used for a hostname that is not included in its SAN.

---

### Step 4 — Check Revocation and Trust

**Command used:**

```
openssl x509 -in mismatch_cert.pem -noout -text | grep -A2 "OCSP"

```

**What you found:**

No OCSP URL was found in the certificate. The grep command returned no output, which indicates that the Authority Information Access extension does not include an OCSP responder URI.

Revocation is not relevant in this case. The failure is caused by a hostname mismatch between the requested hostname (wrong.host.badssl.com) and the names listed in the certificate’s SAN (*.badssl.com, badssl.com).

Even if the certificate were confirmed as valid and not revoked, it still could not be used for this hostname because it was not issued for it. The correct fix is to issue a certificate that includes the required hostname in the SAN.

---

## Reflection

This lab highlights the difference between trust and identity in TLS. A certificate can be valid and trusted but still unusable if it does not match the requested hostname. It also reinforces the importance of the SAN field and the limitations of wildcard certificates. Careful inspection of the SAN is essential when diagnosing certificate-related issues.

---

*CVI PKI Career Pathway — Foundations Phase*
