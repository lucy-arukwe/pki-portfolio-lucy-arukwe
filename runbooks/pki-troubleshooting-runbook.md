----
# PKI Troubleshooting Runbook

## Title: **Certificate Verification Failure Due to Missing Intermediate CA (Broken Chain)**

## Problem Statement

A TLS connection to `incomplete-chain.badssl.com` failed with a certificate verification error. 
The certificate was valid, but the client could not establish trust because the server did not provide a 
complete certificate chain.

## Environment

**Tools:** OpenSSL (`s_client`, `x509`, `verify`), `curl`  
**Components:** TLS leaf certificate, intermediate CA, root CA  
**Context:** Server presented only the leaf certificate after renewal  

## Symptoms

- `unable to verify the first certificate`  
- `unable to get local issuer certificate`  
- Incomplete chain shown in `s_client` output  
- Browser shows untrusted connection  

## Diagnostic Steps

### 1. Retrieve certificate chain

```bash
openssl s_client -connect incomplete-chain.badssl.com:443 -showcerts
```

**Result:**
Only the leaf certificate is presented.

### 2. Inspect certificate
`openssl x509 -in leaf_cert.pem -text -noout`

**Result:**
Certificate is valid, hostname matches, issuer is intermediate CA.

### 3. Verify certificate
`openssl verify leaf_cert.pem`

**Result:**
`unable to verify the first certificate`

### 4. Retrieve intermediate
`openssl x509 -in leaf_cert.pem -noout -text | grep -A3 "Authority Information Access"`

`curl -s "[CA_ISSUERS_URL]" -o intermediate.der`

`openssl x509 -inform DER -in intermediate.der -out issuer_cert.pem`

### 5. Re-verify with intermediate
`openssl verify -untrusted issuer_cert.pem leaf_cert.pem`

**Result:**
`OK`

### 6. Check trust and revocation
`openssl s_client -connect incomplete-chain.badssl.com:443 | grep "Verify return code"`

**Result:**
Root trusted, no revocation errors.

**Conclusion:**
Issue is a missing intermediate CA, not a trust or revocation problem.

**Resolution**

Installed the missing intermediate CA and configured the server to present the full chain.

`openssl verify -untrusted issuer_cert.pem leaf_cert.pem`

**Result:** `OK`

**Root Cause**

- Server misconfiguration. Intermediate CA not included after certificate renewal.

**Prevention**

- Always deploy the full certificate chain and validate using:

  `openssl s_client -connect [host]:443 -showcerts`

**Expected result:**

`Depth 0 = leaf
 Depth 1 = intermediate`
 
 **Lab Reference**

`labs/week-06/submissions/broken-chain/lab-02-broken-chain.md`

CVI PKI Career Pathway — Phase 1 Foundations
