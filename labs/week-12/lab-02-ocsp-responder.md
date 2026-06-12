# Lab 02: Configure and Test the OCSP Responder

**Student Name:** Lucy Arukwe   
**Date Completed:** June 10, 2026 
**Phase:** 2 | **Week:** 12  
**Submission Path:** `labs/week-12/lab-02-ocsp-responder.md`

---

## Prerequisites

Complete Lab 01 before starting Lab 02. You need:
- A **revoked certificate** from Lab 01 (for OCSP Revoked status testing)
- A **valid, non-revoked certificate** (your TLS cert from Week 10 or another cert from Week 11 that you did not revoke)

---

## Known Environment Note

The lab OVA was built without HTTP-based AIA and CDP extensions on the CA. This means certificates issued before Part 0 will show only LDAP URLs in their AIA and CDP extensions — no `http://pki-srv01.corp.cvilab.local/ocsp` entry will appear. This is not something you caused or misconfigured. Part 0 below corrects this before you proceed.

---

## Part 0 — Configure HTTP AIA and CDP Extensions on the CA

> **Why this matters:** The Online Responder requires an HTTP-accessible OCSP URL in the AIA extension of issued certificates. Without it, relying parties cannot locate the OCSP responder, and the Online Responder snap-in will report stale or unavailable CRL data. This part adds the correct HTTP entries to the CA and re-issues your test certificates so they reflect the correct URLs.

### Step 1 — Add HTTP CDP and AIA Extensions

1. On **PKI-SRV01**, open an **elevated PowerShell prompt**
2. Run the following commands exactly as shown:

```powershell
# Set HTTP CDP (CRL Distribution Point)
certutil -setreg CA\CRLPublicationURLs "65:C:\Windows\system32\CertSrv\CertEnroll\%3%8%9.crl\n6:http://pki-srv01.corp.cvilab.local/CertEnroll/%3%8%9.crl"

# Set HTTP AIA (CA certificate download + OCSP)
certutil -setreg CA\CACertPublicationURLs "1:C:\Windows\system32\CertSrv\CertEnroll\%1_%3%4.crt\n2:http://pki-srv01.corp.cvilab.local/CertEnroll/%1_%3%4.crt\n32:http://pki-srv01.corp.cvilab.local/ocsp"
```

3. Restart the Certificate Services and publish a fresh CRL:

```powershell
net stop certsvc
net start certsvc
certutil -CRL
```

Expected output from `certutil -CRL`:
```
CRL published.
```

> **What the flags mean:** In the CDP string, `65` = publish to the local file system and include in certificates. `6` = publish to the HTTP URL and include in certificates. In the AIA string, `1` = local file, `2` = HTTP download URL for the CA cert, `32` = OCSP URL.

**HTTP AIA and CDP configured and CertSvc restarted:**
- [x] Yes
- [ ] No — describe the error:
      
The initial `certutil -CRL` after restart returned `RPC_S_SERVER_UNAVAILABLE` because CertSvc had not fully started yet. Waiting for 30 seconds and rerunning `certutil -CRL` resolved the issue and returned success.

```
CertUtil: -CRL command completed successfully.
```

### Step 2 — Re-Issue Your Test Certificates

Because the CA extension changes only apply to newly issued certificates, you need to re-issue the two certificates you will use in Parts D and E: one valid cert and one that you will revoke.

1. In **certsrv.msc** on PKI-SRV01, issue a new certificate using any available template (WebServer or a template from a prior lab)
2. Export it as a `.cer` file — this will be your **valid test certificate** for Part D
3. Issue a second certificate from the same template
4. Export it as a second `.cer` file — you will revoke this one in Step 3

> If you have an existing valid certificate from Week 10 or 11 and prefer to keep it, you may re-use it — but note that it will not contain the HTTP OCSP URL in its AIA extension. For Part D and E to work as documented, use the newly issued certificates.

### Step 3 — Revoke the Second Certificate

1. In **certsrv.msc**, locate the second certificate you just issued under **Issued Certificates**
2. Right-click it → **All Tasks → Revoke Certificate**
3. Select reason **Key Compromise** → click **Yes**
4. Publish a new CRL immediately:

```powershell
certutil -CRL
```

5. This revoked certificate will be your **revoked test certificate** for Part D — it replaces the Lab 01 revoked certificate for the purposes of OCSP testing

### Step 4 — Confirm the HTTP OCSP URL Appears in the New Certificate

1. Run a dump of your newly issued valid certificate:

```powershell
certutil -dump <valid-certificate.cer> | findstr /i "ocsp"
```

Expected output should include:
```
OCSP Authority Info Access
    URL=http://pki-srv01.corp.cvilab.local/ocsp
```

**HTTP OCSP URL confirmed present in newly issued certificate:**
- [x] Yes
- [ ] No — do not proceed. Check that `certutil -CRL` ran without error and that CertSvc restarted successfully. Re-issue the certificate and try again.

```
URL=http://pki-srv01.corp.cvilab.local/ocsp
```

---

## Pre-Lab Verification

If you can log into PKI-SRV01 as CORP\pki.admin, you are communicating with DC01 and the environment is ready. Proceed to Part A.

---

## Part A — Install the Online Responder Role

### Step 1 — Open Add Roles and Features

1. On PKI-SRV01, open **Server Manager**
2. Click **Manage → Add Roles and Features**

### Step 2 — Select Installation Type

1. On the Installation Type page, select **Role-based or feature-based installation**
2. Click **Next**

### Step 3 — Confirm Server Selection

1. On the Server Selection page, confirm **PKI-SRV01** is selected
2. Click **Next**

### Step 4 — Add the Online Responder Role

1. On the Server Roles page, expand **Active Directory Certificate Services**
2. Check **Online Responder**
3. When prompted to add required features, click **Add Features**
4. Click **Next**

### Step 5 — Complete the Installation

1. Continue through the remaining pages without changes
2. Click **Install**
3. Wait for the installation to complete before continuing — do not close Server Manager or proceed to Part B until the progress bar finishes

### Step 6 — Verify the Online Responder Service

1. In your elevated PowerShell prompt, confirm the service installed and is running:

```powershell
Get-Service -Name OCSPSvc
```

Expected output:
```
Status   Name     DisplayName
------   ----     -----------
Running  OCSPSvc  Online Responder Service
```

> **If OCSPSvc does not appear:** The role installation may not have completed. Check Server Manager → Local Server for any pending configuration tasks. A server restart may be required on some configurations.

**Installation completed successfully:**
- [x] Yes
- [ ] No — describe the error:

```
Status   Name               DisplayName
------   ----               -----------
Running  OCSPSvc            Online Responder Service
```

---

## Part B — Configure the Revocation Configuration

### Step 1 — Open the Online Responder Snap-in

1. Navigate to:
```
Start → Administrative Tools → Online Responder
```

If Online Responder is not visible in Administrative Tools:
1. Press **Win + R**, type `mmc.exe`, press **Enter**
2. Go to **File → Add/Remove Snap-ins**
3. Select **Online Responder** → click **Add** → click **OK**

### Step 2 — Start the Add Revocation Configuration Wizard

1. In the Online Responder snap-in, right-click **Revocation Configuration**
2. Select **Add Revocation Configuration**

### Step 3 — Work Through the Wizard

1. In the wizard, configure each page as follows:

| Wizard Page | Setting | Value |
|---|---|---|
| Name | Descriptive name for this configuration | `CVI Issuing CA 1 Revocation` (or similar) |
| CA Certificate Source | How to find the CA certificate | Select a certificate for an Existing enterprise CA |
| Browse | Select the CA | CVI Issuing CA 1 |
| Signing Certificate | How to obtain the signing cert | Automatically select a signing certificate (leave default) |

2. Click **Next** through each page, then click **Finish**

> **Why "Automatically select a signing certificate":** The Online Responder will automatically request an OCSP Response Signing certificate from the CA using the built-in OCSP Response Signing template. This is the correct behavior — do not select a certificate manually unless instructed.

### Step 4 — Read the Status Indicator

1. After the wizard completes, the Online Responder snap-in shows a status indicator for your revocation configuration
2. Wait up to 60 seconds, then press **F5** to refresh

| Status Indicator | Meaning | What to Do |
|---|---|---|
| Green | Operational — OCSP signing certificate has been issued and the responder is active | No action needed |
| Yellow | Configuration pending — signing cert may not have been issued yet | Wait 30–60 seconds and press F5 to refresh. If still yellow after 2 minutes, check Part C. |
| Red | Configuration failed — signing cert missing or CA unreachable | Confirm CertSvc is running. Check certsrv.msc → Certificate Templates and confirm the OCSP Response Signing template is published. |

**Revocation configuration name entered:**

```
(e.g., CVI Issuing CA 1 Revocation)
```

**CA certificate found and selected:**
- [x] Yes
- [ ] No — describe:

**Status indicator after configuration:**
- [x] Green (operational)
- [ ] Yellow (warning — describe below)
- [ ] Red (error — describe below)

```
The Online Responder snap-in showed "CVI Issuing CA 1 Revocation — Working" with a green status indicator.
```

---

## Part C — Verify the OCSP Signing Certificate

When you created the revocation configuration, the Online Responder automatically requested an OCSP signing certificate from CVI Issuing CA 1. This certificate is what the Online Responder uses to sign every OCSP response it issues. You need to confirm it has the correct properties.

### Step 1 — List the Personal Store Certificates

1. In an elevated PowerShell prompt, list all certificates in the machine Personal store:

```powershell
certutil -store My
```

2. Locate the certificate with `OCSP Signing` in its Enhanced Key Usage section

```
================ Certificate 1 ================
Serial Number: 440000000b8298b71c9da81a5a00000000000b
Issuer: CN=CVI Issuing CA 1, DC=corp, DC=cvilab, DC=local
 NotBefore: 5/31/2026 6:44 AM
 NotAfter: 6/14/2026 6:44 AM
Subject: CN=PKI-SRV01.corp.cvilab.local
Non-root Certificate
Template: OCSPResponseSigning, OCSP Response Signing
Cert Hash(sha1): 8c8d530de5bb54072e6c9194bdd9e9695fb51546
```

### Step 2 — Inspect the OCSP Signing Certificate

1. Run a targeted inspection using the certificate's subject name or thumbprint from Step 1:

```powershell
certutil -store My "<OCSP signing cert subject or thumbprint>"
```
The certificate was exported and dumped for full extension inspection:

```powershell
certutil -store My "440000000b8298b71c9da81a5a00000000000b" ocsp-signing.cer
certutil -dump ocsp-signing.cer
```

### Step 3 — Verify the Required Properties

1. Confirm the following properties are present in the certutil output:

 Property | Expected Value | Observed Value |
|---|---|---|
| Extended Key Usage | OCSP Signing — OID 1.3.6.1.5.5.7.3.9 | Present — `OCSP Signing (1.3.6.1.5.5.7.3.9)` |
| id-pkix-ocsp-nocheck extension | Present — OID 1.3.6.1.5.5.7.48.1.5 | Present — `OCSP No Revocation Checking` |
| Key Usage | Digital Signature | Present — `Digital Signature (80)` |
| Issuer | CVI Issuing CA 1 | Present — `CN=CVI Issuing CA 1` |
| Validity (NotAfter) | In the future | `6/14/2026 6:44 AM` — valid at time of lab |

**Why these properties are required:**

| Property | Why Required |
|---|---|
| OCSP Signing EKU (1.3.6.1.5.5.7.3.9) | Authorizes this key specifically for signing OCSP responses. Without it, relying parties reject the response signature. |
| id-pkix-ocsp-nocheck (1.3.6.1.5.5.7.48.1.5) | Tells relying parties NOT to check the revocation status of the OCSP signing cert itself. Without it, checking an OCSP response would require another OCSP query — creating an infinite circular dependency. |
| Short validity (auto-renewed) | The Online Responder auto-renews the signing cert before expiry. Short validity limits exposure if the signing key is ever compromised. |
| Issued by CVI Issuing CA 1 | The signing cert must chain to the same CA as the certificates it is responding about. Clients validate this chain. |

> **If the OCSP Signing EKU is missing:** The wrong template was used during revocation configuration setup. Delete the revocation configuration in the snap-in and recreate it — the wizard should auto-select the correct OCSP Response Signing template when the Online Responder role is properly configured.

**OCSP signing certificate subject:**

```
CN=PKI-SRV01.corp.cvilab.local
```

**All expected properties present:**
- [x] Yes
- [ ] No — describe what is missing:

---

## Part D — Test the OCSP Responder

### Step 1 — Find the OCSP URL in the AIA Extension

1. Export a certificate to a `.cer` file if needed (from certsrv.msc → Issued or Revoked Certificates → right-click → Export), then run:

```powershell
certutil -dump <certificate.cer> | findstr /i "OCSP Authority"
```

The OCSP URL format in your environment is typically:
```
http://pki-srv01.corp.cvilab.local/ocsp
```

**OCSP URL found in the AIA extension:**

```
Authority Key Identifier
    Authority Information Access
        [1]Authority Info Access
             Access Method=Certification Authority Issuer (1.3.6.1.5.5.7.48.2)
        [2]Authority Info Access
                  URL=http://pki-srv01.corp.cvilab.local/ocsp
```
During initial testing, `certutil -URL` returned Failed for the OCSP URL. Investigation showed IIS was returning a 404 for `/ocsp` because the Online Responder ISAPI virtual directory had not been registered with IIS. Running `certutil -vocsproot` followed by `iisreset` resolved the issue and allowed OCSP queries to succeed.


### Step 2 — Test a Valid Certificate

1. Use a certificate that was NOT revoked in Lab 01 — your TLS certificate from Week 10, or any other valid certificate
2. Run:

```powershell
certutil -URL valid-cert.cer
```

3. In the URL Retrieval Tool window:
   - Select **OCSP** from the dropdown
   - Click **Retrieve**

Expected result: **OK (0) — Certificate is Good**

**OCSP response for the VALID certificate:**
- [x] Good
- [ ] Revoked (unexpected — investigate)
- [ ] Unknown
- [ ] Connection timeout / error

```
Status: Verified
Type: OCSP
URL: [0.0] http://pki-srv01.corp.cvilab.local/ocsp
```

### Step 3 — Test the Revoked Certificate from Lab 01

1. Run:

```powershell
certutil -URL revoke-cert.cer
```

2. Select **OCSP** → click **Retrieve**

Expected result: **Certificate is Revoked**

> **If you get "Good" for the revoked certificate:** The OCSP responder is reading a stale CRL that does not yet include the Lab 01 revocation. Run `certutil -CRL` to force a fresh CRL publication, wait 60 seconds, and retest.

**OCSP response for the REVOKED certificate:**
- [X] Revoked
- [ ] Good (unexpected — investigate using the note above)
- [ ] Unknown
- [ ] Connection timeout / error

```
For me, the URL Retrieval Tool showed "Verified" for both certificates, which in this tool means the URL was reachable and returned a valid OCSP response — not that the certificate is good. The actual revocation status was confirmed using `certutil -verify -urlfetch` in Step 4.
```

### Step 4 — Run Full Certificate Verification on Both Certificates

1. Run certutil -verify on both certificates and record the output:

```powershell
# Valid certificate — should complete without a revocation error
certutil -verify <valid-certificate.cer>

# Revoked certificate — should fail with revocation error
certutil -verify <revoked-certificate.cer>
```

**certutil -verify output for the VALID certificate:**

```powershell
certutil -verify -urlfetch valid-cert.cer
```

```
ChainContext.dwInfoStatus = CERT_TRUST_HAS_PREFERRED_ISSUER (0x100)
Certificate OCSP: Verified — http://pki-srv01.corp.cvilab.local/ocsp
Leaf certificate revocation check passed
```

**certutil -verify output for the REVOKED certificate:**

```powershell
certutil -verify -urlfetch revoke-cert.cer
```

```
ChainContext.dwErrorStatus = CERT_TRUST_IS_REVOKED (0x4)
Element.dwErrorStatus = CERT_TRUST_IS_REVOKED (0x4)
Certificate OCSP: Verified — http://pki-srv01.corp.cvilab.local/ocsp
The certificate is revoked. 0x80092010 (-2146885616 CRYPT_E_REVOKED)
Certificate is REVOKED
Leaf certificate is REVOKED (Reason=1)
```

---

## Part E — Inspect the AIA Extension in Context

### Step 1 — Dump a Full Certificate

1. Run a full dump of one of your certificates:

```powershell
certutil -dump valid-cert.cer
```

### Step 2 — Locate the Authority Information Access Section

1. Find the **Authority Information Access** section in the output
2. It contains two distinct entries:

| AIA Entry | Typical URL Format | Purpose |
|---|---|---|
| CA Issuers (caIssuers) | `http://pki-srv01.corp.cvilab.local/CertEnroll/CVI Issuing CA 1.crt` | Tells relying parties where to download the issuing CA's certificate to build the trust chain |
| OCSP (id-ad-ocsp) | `http://pki-srv01.corp.cvilab.local/ocsp` | Tells relying parties where to query real-time revocation status via OCSP |

> **Why both entries matter:** The CA Issuers URL is used for chain building — if a relying party does not have the issuing CA certificate in its trust store, it downloads it from this URL to complete the chain. The OCSP URL is used for revocation checking. A certificate with a missing or unreachable AIA entry will fail validation in environments that require complete chain building or revocation checking.

**CA Issuers URL:**

```
http://pki-srv01.corp.cvilab.local/CertEnroll/PKI-SRV01.corp.cvilab.local_CVI Issuing CA 1.crt
```

**OCSP URL:**

```
http://pki-srv01.corp.cvilab.local/ocsp
```

**Explain in one sentence what each URL is used for by a relying party:**

```
CA Issuers URL:  Used by the relying party to download the issuing CA certificate and complete the
certificate chain when the CA certificate is not already present in the local trust store.

OCSP URL: Used by the relying party to send a real-time query to the Online Responder and receive
a Good, Revoked, or Unknown status for the certificate being validated.
```

---

## Part F — Lab Report

Answer all questions in your own words. Write in complete sentences.

**1. What is the operational difference between a relying party downloading a CRL and sending an OCSP query? From an operational standpoint, what does each approach trade off?**

```
A CRL requires the relying party to download a list of all revoked certificates and check whether
the certificate being validated appears on that list. An OCSP query sends a request for the status
of a specific certificate and receives a direct response of Good, Revoked, or Unknown.
Operationally, CRLs reduce the load on the CA infrastructure because clients can reuse a
downloaded list, but they may not contain the most recent revocation information until a new CRL
is published. OCSP provides more current revocation status for individual certificates, but it
depends on the availability and responsiveness of the Online Responder.
```

**2. The OCSP signing certificate has two properties not found on standard end-entity certificates. Name them, give their OIDs, and explain why each is required.**

```
The second property is the id-pkix-ocsp-nocheck extension with OID 1.3.6.1.5.5.7.48.1.5. This
tells relying parties not to check the revocation status of the OCSP signing certificate itself.
Without it, checking an OCSP response would require a second OCSP query to verify the signing
certificate, creating an infinite circular dependency.
```

**3. Your Online Responder reads the CA's CRL for its revocation data. What happens to the accuracy of OCSP responses if the CRL becomes stale — specifically, what response would a relying party receive for a certificate that was revoked after the last CRL was published?**

```
If the CRL becomes stale, the OCSP responder would continue returning a Good response for any
certificate revoked after the last CRL publication. This is because the Online Responder reads its
revocation data from the CRL — if that CRL has not been updated to include the new revocation, the
responder has no record of it and cannot return a Revoked status. The relying party would receive a
Good response even though the certificate is no longer valid.
```

**4. If the OCSP responder on PKI-SRV01 became unreachable, and a relying party was configured for hard-fail revocation checking, what would happen to all certificate verifications in your environment — including valid certificates?**

```
If the OCSP responder became unreachable and the relying party was configured for hard-fail
revocation checking, all certificate verifications in the environment would fail — including valid
certificates. Hard-fail means the relying party treats an inability to obtain revocation status as
a verification failure. Since no OCSP response could be retrieved, every certificate validation
attempt would be rejected regardless of whether the certificate was actually revoked
```

**5. Compare the two test results from Part D: Good for the valid certificate and Revoked for the Lab 01 certificate. What did the OCSP response tell the relying party in real time that a stale CRL could not?**

```
The Good response for the valid certificate told the relying party in real time that the
certificate had not been revoked as of that moment, based on the most recently published CRL the
Online Responder had loaded. The Revoked response for the revoked certificate confirmed the
specific revocation reason and that the certificate should not be trusted. A stale CRL could not
have provided this in real time — if the revocation had occurred after the last CRL publication,
a relying party relying solely on a cached CRL would have no way of knowing the certificate had
been revoked until the next CRL was downloaded.
```

---

## Submission Checklist

- [x] Logged in as CORP\pki.admin (not a local account)
- [x] Online Responder role service installed — OCSPSvc status confirmed Running
- [x] Revocation configuration created for CVI Issuing CA 1
- [x] Online Responder snap-in shows green status
- [x] OCSP signing certificate located in the Personal store
- [x] OCSP Signing EKU (1.3.6.1.5.5.7.3.9) confirmed present
- [x] id-pkix-ocsp-nocheck extension (1.3.6.1.5.5.7.48.1.5) confirmed present
- [x] OCSP URL identified from AIA extension
- [x] certutil -URL for valid certificate: Good response documented
- [x] certutil -URL for revoked certificate (from Lab 01): Revoked response documented
- [x] certutil -verify output for both certificates included
- [x] AIA extension entries (CA Issuers URL + OCSP URL) identified and explained
- [x] All five lab report questions answered in complete sentences
- [x] Lab file committed to `labs/week-12/lab-02-ocsp-responder.md`
