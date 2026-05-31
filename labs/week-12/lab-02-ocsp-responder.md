# Lab 02: Configure and Test the OCSP Responder

**Student Name:**  
**Date Completed:**  
**Phase:** 2 | **Week:** 12  
**Submission Path:** `labs/week-12/lab-02-ocsp-responder.md`

---

## Prerequisites

Complete Lab 01 before starting Lab 02. You need:
- A **revoked certificate** from Lab 01 (for OCSP Revoked status testing)
- A **valid, non-revoked certificate** (your TLS cert from Week 10 or another cert from Week 11 that you did not revoke)

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
- [ ] Yes
- [ ] No — describe the error:

```
(paste Get-Service OCSPSvc output here — confirm Status is Running)
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
- [ ] Yes
- [ ] No — describe:

**Status indicator after configuration:**
- [ ] Green (operational)
- [ ] Yellow (warning — describe below)
- [ ] Red (error — describe below)

```
(describe what the snap-in shows — any warnings, errors, or confirmation of green status)
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
(paste the output — identify the OCSP signing certificate entry)
```

### Step 2 — Inspect the OCSP Signing Certificate

1. Run a targeted inspection using the certificate's subject name or thumbprint from Step 1:

```powershell
certutil -store My "<OCSP signing cert subject or thumbprint>"
```

### Step 3 — Verify the Required Properties

1. Confirm the following properties are present in the certutil output:

| Property | Expected Value | Observed Value |
|---|---|---|
| Extended Key Usage | OCSP Signing — OID 1.3.6.1.5.5.7.3.9 | |
| id-pkix-ocsp-nocheck extension | Present — OID 1.3.6.1.5.5.7.48.1.5 | |
| Key Usage | Digital Signature | |
| Issuer | CVI Issuing CA 1 | |
| Validity (NotAfter) | In the future | |

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
(paste the CN or subject from the signing certificate)
```

**All expected properties present:**
- [ ] Yes
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
(paste the OCSP URL)
```

### Step 2 — Test a Valid Certificate

1. Use a certificate that was NOT revoked in Lab 01 — your TLS certificate from Week 10, or any other valid certificate
2. Run:

```powershell
certutil -URL <valid-certificate.cer>
```

3. In the URL Retrieval Tool window:
   - Select **OCSP** from the dropdown
   - Click **Retrieve**

Expected result: **OK (0) — Certificate is Good**

**OCSP response for the VALID certificate:**
- [ ] Good
- [ ] Revoked (unexpected — investigate)
- [ ] Unknown
- [ ] Connection timeout / error

```
(paste or describe the certutil -URL output for the valid certificate)
```

### Step 3 — Test the Revoked Certificate from Lab 01

1. Run:

```powershell
certutil -URL <revoked-certificate.cer>
```

2. Select **OCSP** → click **Retrieve**

Expected result: **Certificate is Revoked**

> **If you get "Good" for the revoked certificate:** The OCSP responder is reading a stale CRL that does not yet include the Lab 01 revocation. Run `certutil -CRL` to force a fresh CRL publication, wait 60 seconds, and retest.

**OCSP response for the REVOKED certificate:**
- [ ] Revoked
- [ ] Good (unexpected — investigate using the note above)
- [ ] Unknown
- [ ] Connection timeout / error

```
(paste or describe the certutil -URL output for the revoked certificate)
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

```
(paste output — confirm verification completes successfully)
```

**certutil -verify output for the REVOKED certificate:**

```
(paste output — confirm CRYPT_E_REVOKED or similar error)
```

---

## Part E — Inspect the AIA Extension in Context

### Step 1 — Dump a Full Certificate

1. Run a full dump of one of your certificates:

```powershell
certutil -dump <certificate.cer>
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
(the URL that points to the issuing CA certificate for chain building)
```

**OCSP URL:**

```
(the URL that points to the Online Responder — confirm it matches what you used in Part D)
```

**Explain in one sentence what each URL is used for by a relying party:**

```
CA Issuers URL:

OCSP URL:
```

---

## Part F — Lab Report

Answer all questions in your own words. Write in complete sentences.

**1. What is the operational difference between a relying party downloading a CRL and sending an OCSP query? From an operational standpoint, what does each approach trade off?**

```
(your answer here)
```

**2. The OCSP signing certificate has two properties not found on standard end-entity certificates. Name them, give their OIDs, and explain why each is required.**

```
(your answer here)
```

**3. Your Online Responder reads the CA's CRL for its revocation data. What happens to the accuracy of OCSP responses if the CRL becomes stale — specifically, what response would a relying party receive for a certificate that was revoked after the last CRL was published?**

```
(your answer here)
```

**4. If the OCSP responder on PKI-SRV01 became unreachable, and a relying party was configured for hard-fail revocation checking, what would happen to all certificate verifications in your environment — including valid certificates?**

```
(your answer here)
```

**5. Compare the two test results from Part D: Good for the valid certificate and Revoked for the Lab 01 certificate. What did the OCSP response tell the relying party in real time that a stale CRL could not?**

```
(your answer here)
```

---

## Submission Checklist

- [ ] Logged in as CORP\pki.admin (not a local account)
- [ ] Online Responder role service installed — OCSPSvc status confirmed Running
- [ ] Revocation configuration created for CVI Issuing CA 1
- [ ] Online Responder snap-in shows green status
- [ ] OCSP signing certificate located in the Personal store
- [ ] OCSP Signing EKU (1.3.6.1.5.5.7.3.9) confirmed present
- [ ] id-pkix-ocsp-nocheck extension (1.3.6.1.5.5.7.48.1.5) confirmed present
- [ ] OCSP URL identified from AIA extension
- [ ] certutil -URL for valid certificate: Good response documented
- [ ] certutil -URL for revoked certificate (from Lab 01): Revoked response documented
- [ ] certutil -verify output for both certificates included
- [ ] AIA extension entries (CA Issuers URL + OCSP URL) identified and explained
- [ ] All five lab report questions answered in complete sentences
- [ ] Lab file committed to `labs/week-12/lab-02-ocsp-responder.md`
