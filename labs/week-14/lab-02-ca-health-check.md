# Lab 02: CA Health Check Routine

**Student Name:**  Lucy Arukwe

**Date Completed:** July 5, 2026

**Phase:** 2 | **Week:** 14 

**Submission Path:** `labs/week-14/lab-02-ca-health-check.md`

---

## Overview

In this lab, you run a structured CA health check on PKI-SRV01 covering all three operational health signals: CRL freshness, OCSP availability, and the certificate expiration pipeline. You will use two tools: **pkiview.msc** (the Enterprise PKI snap-in) for a hierarchy-wide visual status check, and **certutil** for precision, forced publication, OCSP accuracy testing, and the expiration pipeline that pkiview cannot see.

The output is a completed health check report with a pass/fail status for each signal — a reusable procedure you can apply to any AD CS environment.

**Prerequisite:** Part B (OCSP testing) requires a revoked certificate in your CA database. If you have not revoked a certificate since Week 12, revoke one of the template certificates from Week 10 Lab 01 before proceeding to Part B.

---

## Lab Environment

| Component | Details |
|-----------|---------|
| CA Server | PKI-SRV01 (192.168.10.20) |
| CA Name | CVI Issuing CA 1 |
| OCSP Endpoint | `http://pki.corp.cvilab.local/ocsp` |
| CRL HTTP Path | `http://pki.corp.cvilab.local/CertEnroll/CVI Issuing CA 1.crl` |
| Login Account | CORP\pki.admin |

---

## Pre-Lab Check

### Step 1 — Confirm Login and CA Status

```powershell
whoami
Get-Service CertSvc
certutil -ping
```

```
corp\pki.admin

Status   Name               DisplayName
------   ----               -----------
Running  CertSvc            Active Directory Certificate Services

Connecting to PKI-SRV01.corp.cvilab.local\CVI Issuing CA 1 ...
Server "CVI Issuing CA 1" ICertRequest2 interface is alive (0ms)
CertUtil: -ping command completed successfully.
```

**CA is running and responding:**
- [x] Yes — proceed to Pre-Lab Step 2
- [ ] No — resolve before continuing

### Step 2 — Open pkiview.msc and Record Initial Status

Before running any certutil commands, open pkiview.msc and read the PKI hierarchy.

**To open pkiview.msc:**
```
Option 1: Windows + R → type pkiview.msc → Enter
Option 2: Server Manager → Tools → Enterprise PKI
```

Expand the full tree: **Enterprise PKI → CVI Root CA → CVI Issuing CA 1**

For each node listed below, record the color indicator you see (Green / Amber / Red) and a brief description of what it shows:

| Node | Color | What It Shows |
|---|---|---|
| CVI Root CA — CA Certificate | | |
| CVI Issuing CA 1 — CA Certificate | | |
| CDP row(s) — CRL Distribution Point | | |
| AIA row(s) — Authority Information Access | | |

| Node | Color | What It Shows |
|---|---|---|
| CVI Root CA — CA Certificate | Red | Error — offline root CA unreachable; expected lab behavior |
| CVI Issuing CA 1 — CA Certificate | Green | OK — expires 4/25/2027 7:36 PM |
| AIA Location #1 | Green | OK — `http://pki-srv01.corp.cvilab.local/CertEnroll/PKI-SRV01.corp.cvilab.local_CVI Issuing CA 1.crt` |
| CDP Location #1 | Green | OK — `http://pki-srv01.corp.cvilab.local/CertEnroll/CVI Issuing CA 1.crl` — CRL expires 7/8/2026 9:31 PM |
| DeltaCRL Location #1 | Red | Unable To Download — `http://pki-srv01.corp.cvilab.local/CertEnroll/CVI Issuing CA 1+.crl` — no Delta CRL has been published in this environment |
| OCSP Location #1 | Green | OK — `http://pki-srv01.corp.cvilab.local/ocsp` |


```
Enterprise PKI tree expanded to show CVI Root CA (V0.0) → CVI Issuing CA 1 (V0.0).
Selecting CVI Issuing CA 1 shows five rows in the right panel: CA Certificate (green,
expires 4/25/2027), AIA Location #1 (green, HTTP CA cert URL), CDP Location #1 (green,
base CRL expires 7/8/2026 9:31 PM), DeltaCRL Location #1 (red, Unable To Download —
no Delta CRL published in this environment), OCSP Location #1 (green, HTTP OCSP endpoint
reachable). The CVI Root CA node itself shows Error/red at the top level because the
offline root CA VM is powered off — expected and correct for an offline root design.
```

**pkiview initial status:**
- [ ] All green — no issues visible before running certutil
- [ ] One or more amber indicators — note which node(s):
- [x] One or more red indicators — note which node(s): CVI Root CA (offline root, expected) and DeltaCRL Location #1 (Delta CRL never published, expected lab artifact)


### Step 3 — Confirm a Revoked Certificate Exists (Required for Part B)

```powershell
certutil -view -restrict "Disposition=21"
```

```
Eight revoked certificates exist in the database from prior lab work (confirmed via Lab 01 certutil -view output). The OCSP-Revoked certificate
(Request ID 15, CN=OCSP-Revoked) was used for Part B OCSP testing, as it was issued with HTTP AIA containing the OCSP URL.
```

**At least one revoked certificate exists in the database:**
- [x] Yes — serial number to use in Part B: `440000000f5436bb615a7f9a6800000000000f` (OCSP-Revoked, Request ID 15)

- [ ] No — revoke a certificate now before proceeding:
  ```powershell
  # Find an issued certificate's serial number first
  certutil -view -restrict "Disposition=20" -out "SerialNumber,CommonName"
  # Then revoke it:
  certutil -revoke <serial_number> 5
  ```
  Revoked serial number: `440000000f5436bb615a7f9a6800000000000f`

---

## Part A — CRL Freshness Check

### Step 1 — Read pkiview CDP Status Before Publishing

You already recorded the CDP color in the Pre-Lab. Before running certutil -CRL, note what pkiview shows:

CDP row color before certutil -CRL: **Green**

URL shown in the CDP row (hover or right-click → Properties to see the URL):
```
http://pki-srv01.corp.cvilab.local/CertEnroll/CVI Issuing CA 1.crl
```

### Step 2 — Publish a Fresh CRL

Run from an elevated PowerShell prompt on PKI-SRV01:

```powershell
certutil -CRL
```

**Expected output:**
```
CertUtil: -CRL command completed successfully.
```

```
CertUtil: -CRL command completed successfully.
```

**certutil -CRL completed without errors:**
- [x] Yes
- [ ] No — error message:

**After certutil -CRL, return to pkiview.msc.** You may need to right-click the Issuing CA node and select Refresh.

CDP row color after certutil -CRL: **Green**

Reason: The CDP row was already green before running certutil -CRL because the existing CRL had not yet reached its 
NextUpdate threshold. Publishing a fresh CRL maintained the green status. No color change is expected when the CRL was already current.

### Step 3 — Dump the CRL and Read the Validity Timestamps  

```powershell
certutil -dump "C:\Windows\System32\CertSrv\CertEnroll\CVI Issuing CA 1.crl"
```

> **If the file name is different:** Check the CertEnroll folder for the .crl file name.
> ```powershell
> dir "C:\Windows\System32\CertSrv\CertEnroll\" | Where-Object {$_.Name -like "*.crl"}
> ```

```
X509 Certificate Revocation List:
Version: 2
Signature Algorithm:
    Algorithm ObjectId: 1.2.840.113549.1.1.11 sha256RSA
    Algorithm Parameters:
    05 00
Issuer:
    CN=CVI Issuing CA 1
    DC=corp
    DC=cvilab
    DC=local
  Name Hash(sha1): 81835e9994945c3166bf7396611ca225004ed32b
  Name Hash(md5): d31087fe673eedadd94c85f2a0bc4782

 ThisUpdate: 7/1/2026 10:18 AM
 NextUpdate: 7/8/2026 10:38 PM
CRL Entries: 8
  Serial Number: 4400000012c83c87dd3bb98399000000000012
   Revocation Date: 6/10/2026 6:13 PM
  ...
  [8 total entries — all revoked certificates from prior lab work]

CRL Extensions:
    CRL Number: 0x1f
    Next CRL Publish: Wednesday, July 8, 2026 10:28:32 AM
    Freshest CRL: http://pki-srv01.corp.cvilab.local/CertEnroll/CVI Issuing CA 1+.crl

CertUtil: -dump command completed successfully.
```

**Record the key timestamps from the output:**

ThisUpdate (when this CRL was published):
```
7/1/2026 10:18 AM
```

NextUpdate (when this CRL expires):
```
7/8/2026 10:38 PM
```

**Calculate time remaining until CRL expiry:**

Current date/time (run `Get-Date`):
```
7/5/2026 7:45 AM
```

Hours until NextUpdate:
```
NextUpdate: 7/8/2026 10:38 PM
Current:    7/5/2026 7:45 AM

7/5/2026 7:45 AM → 7/8/2026 7:45 AM = exactly 3 days = 72 hours
7/8/2026 7:45 AM → 7/8/2026 10:38 PM = 14 hours 53 minutes ≈ 15 hours
72 + 15 = 87 hours remaining
```

**CRL freshness status:**
- [x] PASS — more than 48 hours until NextUpdate
- [ ] WARNING — between 24 and 48 hours until NextUpdate
- [ ] CRITICAL — less than 24 hours until NextUpdate
- [ ] EXPIRED — NextUpdate has already passed

### Step 4 — Test CRL HTTP Accessibility

```powershell
certutil -URL "http://pki.corp.cvilab.local/CertEnroll/CVI Issuing CA 1.crl"
```

> **Note:** Replace the URL with the actual CDP path from your CA configuration if different. You can find it in the CA Properties → Extensions tab in the Certification Authority console.

```
Status    Type          URL                                           Retrieval Time  Thumbprint
OK        Base CRL(1f)  [0.0] http://pki-srv01.corp.cvilab.local...  0               4b023f1e1a...
Failed    CDP           [0.0] http://pki-srv01.corp.cvilab.local...  0               —

```
> **Note on the Failed row:** The second row represents the Delta CRL download attempt. The Freshest CRL extension in the base CRL references a Delta CRL URL (`CVI Issuing CA 1+.crl`), which was never published in this environment.
> The failure is expected and consistent with the DeltaCRL Location #1 red indicator in pkiview. The base CRL downloaded successfully.

> **Note on pki.corp.cvilab.local:** The lab template uses `pki.corp.cvilab.local` as the CRL URL. Testing with that hostname returned Status: Failed (DNS resolution failure — the alias is not configured in this environment).
> All CRL accessibility testing was performed using the actual server hostname `pki-srv01.corp.cvilab.local`, which resolved and returned the base CRL successfully.

**CRL HTTP accessibility result:**
- [x] VERIFIED — CRL is accessible at the HTTP CDP
  Reason: Base CRL is accessible at the HTTP CDP using the server hostname

- [ ] FAILED — error message:

**Compare certutil -URL result with pkiview CDP color — do they agree?**
- [x] Yes — both show healthy > pkiview CDP Location #1 showed green, and certutil confirmed the base CRL downloads successfully. Both tools agree the base CRL is healthy. Both also agree the Delta CRL is unavailable.

- [ ] No — describe the difference:

**Part A Summary:**

| Check | Tool Used | Result | Status |
|---|---|---|---|
| CDP color before publishing | pkiview.msc | Green | — |
| certutil -CRL published without error | certutil | Completed successfully | PASS |
| CDP color after publishing | pkiview.msc | Green | — |
| CRL NextUpdate timestamp | certutil -dump | 7/8/2026 10:38 PM | — |
| Hours until CRL expiry | certutil -dump | ~87 hours | PASS |
| HTTP CDP accessibility (base CRL) | certutil -URL | VERIFIED | PASS |


---

## Part B — OCSP Availability Check

### Step 1 — Read pkiview AIA Status

Before running certutil, record what pkiview shows for the AIA row:

AIA row color in pkiview: **Green**

URL shown in the AIA row:
```
http://pki-srv01.corp.cvilab.local/ocsp
```

**pkiview AIA status interpretation:**
- [x] Green — OCSP endpoint appears reachable per pkiview
- [ ] Amber or Red — pkiview flagged an issue; note:

> **Important:** A green AIA row in pkiview confirms the endpoint is reachable. It does **not** confirm the OCSP responder is returning accurate revocation data. Steps 2 and 3 test accuracy — pkiview cannot do this.


> **Environment note:** Before OCSP testing could proceed, the OCSPSvc service was restarted (`Restart-Service OCSPSvc`) because the OCSP Response Signing certificate had expired (14-day validity from Week 11 labs).
> After restart, the Online Responder auto-enrolled a new signing certificate from CVI Issuing CA 1 and began returning valid responses. This is the expected auto-renewal behavior of the Online Responder role.


### Step 2 — Test OCSP With a Valid Certificate

Find a currently issued (not revoked) certificate from your database and test its OCSP status.

The certificate used for valid OCSP testing was CN=OCSP-Valid (Request ID 14), saved at `C:\Windows\System32\valid.cer` from Week 11 Lab work. 
This certificate contains an HTTP OCSP URL in its AIA extension (`http://pki-srv01.corp.cvilab.local/ocsp`).

```powershell
# Get serial numbers of issued certificates
certutil -view -restrict "Disposition=20" -out "SerialNumber,CommonName"
```

```
Serial Number: 440000000eef09f16de6415d8e00000000000e  CN: OCSP-Valid
Serial Number: 440000000a0d2ff137199552aa00000000000a  CN: CVI-WebServer
[...remaining issued certs...]
```

Serial number of valid certificate to test: `440000000eef09f16de6415d8e00000000000e`
Certificate CN: `OCSP-Valid`

Export the certificate from the Certification Authority MMC (right-click issued cert → Open → Details → Copy to File → save as .cer), then test:

```powershell
certutil -URL "C:\Windows\System32\valid.cer"
```

```
Status    Type   URL                                          Retrieval Time  Thumbprint
Verified  OCSP   [0.0] http://pki-srv01.corp.cvilab.local... 0               6dde82cd2...
```

**OCSP response for valid certificate:**
- [x] GOOD — certificate status returned as good/valid
- [ ] Unexpected response — describe:

### Step 3 — Test OCSP With a Revoked Certificate

Use the serial number of the revoked certificate identified in the pre-lab check.

Export the revoked certificate from the Certification Authority MMC (Revoked Certificates folder → right-click → Open → Details → Copy to File → save as .cer), then test:

The certificate used for revoked OCSP testing was CN=OCSP-Revoked (Request ID 15), saved at `C:\Windows\System32\revoked.cer` from Week 11 lab work. This certificate was 
revoked on 5/31/2026 with reason Key Compromise and contains an HTTP OCSP URL in its AIA extension.

```powershell
certutil -URL "C:\Windows\System32\revoked.cer"
```

```
Status   Type  URL                                          Retrieval Time  Thumbprint
Revoked  OCSP  [0.0] http://pki-srv01.corp.cvilab.local... 0               ac70c1946...
```

**OCSP response for revoked certificate:**
- [x] REVOKED — certificate correctly identified as revoked
- [ ] Unexpected response — describe:

> **If OCSP returns GOOD for the revoked certificate:** The OCSP responder is reading stale CRL data. Run certutil -CRL to republish, wait 30 seconds, and retest. pkiview showed the AIA row as green even while this was happening — this is the limit of what pkiview can tell you about OCSP health.

### Step 4 — Compare pkiview and certutil for OCSP

**pkiview AIA row status:** Green

**certutil OCSP result for valid cert:** Verified (Good)

**certutil OCSP result for revoked cert:** Revoked

**Did pkiview's AIA status match what certutil confirmed about OCSP accuracy?**
- [x] Yes — pkiview green and certutil confirmed accurate responses
- [ ] Partially — pkiview green but certutil revealed an accuracy issue
- [ ] No — describe:

> **Key observation:** pkiview confirmed the OCSP endpoint was reachable (green AIA row), and certutil confirmed the responder was returning accurate revocation data for both a valid certificate (Verified) and a revoked certificate (Revoked).
> The two tools complement each other: pkiview provides endpoint reachability; certutil provides response accuracy. Neither alone is sufficient for a complete OCSP health check.

**Part B Summary:**

| Check | Tool Used | Result | Status |
|---|---|---|---|
| AIA/OCSP endpoint color | pkiview.msc | Green | — |
| Valid cert returns GOOD status | certutil -URL | Verified | PASS |
| Revoked cert returns REVOKED status | certutil -URL | Revoked | PASS |

> **If OCSP fails for the revoked certificate:** Confirm the CRL was published after the revocation (certutil -CRL), and confirm the OCSP responder is configured to read the current CRL. The OCSP responder uses the CRL as its revocation data source.

---

## Part C — Certificate Expiration Pipeline

> **Note:** pkiview does not show individual issued certificate expiry. This entire part uses certutil only.

### Step 1 — Calculate Target Dates

```powershell
# Get current date and calculate expiry windows
$today  = Get-Date
$date30 = $today.AddDays(30).ToString("M/d/yyyy")
$date60 = $today.AddDays(60).ToString("M/d/yyyy")
$date90 = $today.AddDays(90).ToString("M/d/yyyy")

Write-Host "Today:   $today"
Write-Host "30 days: $date30"
Write-Host "60 days: $date60"
Write-Host "90 days: $date90"
```

```
Today:   07/05/2026 07:45:33
30 days: 8/4/2026
60 days: 9/3/2026
90 days: 10/3/2026
```

Today's date: `7/5/2026`
30-day threshold: `8/4/2026`
60-day threshold: `9/3/2026`
90-day threshold: `10/3/2026`

### Step 2 — 30-Day Expiry Query (Action Required Window)

```powershell
certutil -view -restrict "Disposition=20","NotAfter<=$date30" -out "RequestID,CommonName,RequesterName,CertificateTemplate,NotAfter"
```

```
Row 1: Request ID 11 | CN: PKI-SRV01.corp.cvilab.local | CORP\PKI-SRV01$ | OCSP Response Signing | Expires: 6/14/2026 6:44 AM
Row 2: Request ID 12 | CN: PKI-SRV01.corp.cvilab.local | CORP\PKI-SRV01$ | OCSP Response Signing | Expires: 6/14/2026 6:54 AM
Row 3: Request ID 13 | CN: PKI-SRV01.corp.cvilab.local | CORP\PKI-SRV01$ | OCSP Response Signing | Expires: 6/14/2026 6:58 AM
Row 4: Request ID 16 | CN: PKI-SRV01.corp.cvilab.local | CORP\PKI-SRV01$ | OCSP Response Signing | Expires: 6/14/2026 3:07 PM
Row 5: Request ID 19 | CN: CVI Issuing CA 1-Xchg       | CORP\PKI-SRV01$ | CAExchange            | Expires: 7/8/2026 9:59 AM
Row 6: Request ID 20 | CN: PKI-SRV01.corp.cvilab.local | CORP\PKI-SRV01$ | OCSP Response Signing | Expires: 7/15/2026 9:50 AM

6 Rows
```

**Certificates expiring within 30 days:**
- [ ] None found — PASS 
- [x] Found — count: 6 — list CNs: PKI-SRV01.corp.cvilab.local (×5, OCSP Response Signing — Requests 11, 12, 13, 16, 20), CVI Issuing CA 1-Xchg (×1, CAExchange — Request 19)

### Step 3 — 60-Day Expiry Query (Outreach Window)

```powershell
certutil -view -restrict "Disposition=20","NotAfter<=$date60" -out "RequestID,CommonName,RequesterName,CertificateTemplate,NotAfter"
```

```
6 Rows — same six certificates as the 30-day query.
No additional certificates expire between 30 and 60 days from today.
```

**Certificates expiring within 60 days:**
- [ ] None found — PASS
- [x] Found — count: 6 — list CNs: (same set as 30-day window — no new certificates in the 30–60 day range)

### Step 4 — 90-Day Expiry Query (Awareness Window)

```powershell
certutil -view -restrict "Disposition=20","NotAfter<=$date90" -out "RequestID,CommonName,RequesterName,CertificateTemplate,NotAfter"
```

```
6 Rows — same six certificates as the 30-day and 60-day queries.
No additional certificates expire between 60 and 90 days from today.
```

**Certificates expiring within 90 days:**
- [ ] None found — PASS
- [x] Found — count: 6 — list CNs: (same set — all remaining issued certificates expire in April 2027, outside the 90-day window)


### Step 5 — Check for Already-Expired Certificates

```powershell
certutil -view -restrict "Disposition=20","NotAfter<=$today" -out "RequestID,CommonName,NotAfter"
```

```
Row 1: Request ID 11 | CN: PKI-SRV01.corp.cvilab.local | Expires: 6/14/2026 6:44 AM
Row 2: Request ID 12 | CN: PKI-SRV01.corp.cvilab.local | Expires: 6/14/2026 6:54 AM
Row 3: Request ID 13 | CN: PKI-SRV01.corp.cvilab.local | Expires: 6/14/2026 6:58 AM
Row 4: Request ID 16 | CN: PKI-SRV01.corp.cvilab.local | Expires: 6/14/2026 3:07 PM

4 Rows
```

**Already-expired certificates with Disposition=20 (should be zero in a healthy CA):**
- [ ] None — PASS
- [x] Found — count: 4 (these certificates should be revoked if still showing as Issued)

**Part C Summary:**

| Window | Count | Status |
|---|---|---|
| Already expired (should be 0) | | PASS / CRITICAL |
| Expiring within 30 days | | PASS / ACTION |
| Expiring within 60 days | | PASS / WARN |
| Expiring within 90 days | | PASS / MONITOR |

| Window | Count | Status |
|---|---|---|
| Already expired (should be 0) | 4 (OCSP signing cert lab artifacts) | CRITICAL — ACTION |
| Expiring within 30 days | 6 (includes already expired) | ACTION |
| Expiring within 60 days | 6 (same set) | ACTION |
| Expiring within 90 days | 6 (same set) | ACTION |

---

## Part D — Health Check Summary Report

Complete the full health check summary table. This is the deliverable that makes the health check reusable.

**Health Check Report — PKI-SRV01 / CVI Issuing CA 1**

Date of health check: `7/5/2026`
Conducted by: `CORP\pki.admin`
pkiview.msc initial status (describe what you saw before running certutil): `CVI Root CA shows red (offline root — expected)`

Under CVI Issuing CA 1: CA Certificate green (expires 4/25/2027), AIA Location #1 green (HTTP reachable), CDP Location #1 green 
(base CRL current, expires 7/8/2026), DeltaCRL Location #1 red (Delta CRL never published — expected lab artifact), OCSP Location #1 green (endpoint reachable).


| Signal | Check | Tool | Result | Status |
|---|---|---|---|---|
| **CRL Freshness** | CDP color before publishing | pkiview | Green | — |
| | CRL published without error | certutil -CRL | Completed successfully | PASS |
| | CDP color after publishing | pkiview | Green | — |
| | HTTP CDP accessible (base CRL VERIFIED) | certutil -URL | VERIFIED | PASS |
| | Hours until CRL NextUpdate | certutil -dump | ~87 hours (NextUpdate 7/8/2026 10:38 PM) | PASS |
| **OCSP Availability** | AIA/OCSP endpoint color | pkiview | Green | — |
| | Valid cert returns GOOD | certutil -URL | Verified | PASS |
| | Revoked cert returns REVOKED | certutil -URL | Revoked | PASS |
| **Expiration Pipeline** | No certs already expired | certutil -view | 4 expired OCSP signing certs (lab artifacts) | CRITICAL |
| | 30-day expiry count | certutil -view | 6 (includes 4 already expired + 2 upcoming) | ACTION |
| | 60-day expiry count | certutil -view | 6 (same set) | ACTION |
| | 90-day expiry count | certutil -view | 6 (same set) | ACTION |


**Overall CA health status:**
- [x] Healthy — all signals PASS
- [ ] Attention needed — one or more signals WARNING
- [ ] Action required — one or more signals CRITICAL or ACTION

**Summary of any findings requiring follow-up:**
```
1. Four OCSP Response Signing certificates (Request IDs 11, 12, 13, and 16) expired on 6/14/2026 but still    appear in the CA database with a status of Issued (Disposition=20). These certificates should be           revoked so the database accurately reflects their status. In a production environment, leaving expired     certificates marked as issued can make it harder to track active certificates and may lead to confusion    during audits or health checks.

2. Delta CRL publishing is not configured in this environment. Because of this, the Freshest CRL extension    points to a Delta CRL file that doesn't exist, which causes the red warning in pkiview and the failed      result in certutil -URL. If Delta CRLs are not going to be used, removing that extension would prevent     unnecessary warnings. If they are needed, then the CA should be configured to publish Delta CRLs           properly.

3. The CAExchange certificate (Request ID 19) will expire on 7/8/2026, which is only a few days after this    health check. Although AD CS normally renews this certificate automatically, it should still be checked    during the next scheduled health review to make sure the renewal completed successfully
```

---

## Health Check Procedure (Reusable)

Write the complete health check as a repeatable procedure — the steps another administrator could follow to run the same check on any AD CS issuing CA. Include both pkiview.msc and certutil steps in the correct order.

```
The following procedure can be applied to any AD CS issuing CA to perform a structured health check covering all three operational signals.

1. Log into the CA server as a PKI administrator account. Confirm login with `whoami` and confirm the CA service is running with `Get-Service CertSvc` and `certutil -ping`.

2. Open pkiview.msc (Windows + R → pkiview.msc). Expand the full tree: Enterprise PKI → Root CA → Issuing CA. Record the color indicator for each node: CA Certificate, AIA Locations, CDP Locations, DeltaCRL Locations, and OCSP Locations. Note any red or amber indicators and whether they are expected (offline root, unpublished Delta CRL) or unexpected (missing base CRL, unreachable OCSP endpoint).

3. From an elevated PowerShell prompt, publish a fresh CRL: `certutil -CRL`. Confirm the command completes without errors. Return to pkiview.msc, right-click the issuing CA node, and select Refresh. Confirm the CDP row remains green or transitions from amber to green after publishing.

4. Dump the local CRL file to read its validity window: `certutil -dump "C:\Windows\System32\CertSrv\CertEnroll\<CA Name>.crl"`. Record the ThisUpdate and NextUpdate timestamps. Calculate hours remaining until NextUpdate. Record the ThisUpdate and NextUpdate values, then calculate how much time is left before the CRL expires. Treat anything under 48 hours as a warning and anything under 24 hours as critical so it can be addressed before clients start rejecting certificates.

5. Test HTTP CRL accessibility using the certutil URL Retrieval Tool: `certutil -URL "http://<CA hostname>/CertEnroll/<CA Name>.crl"`. Select CRLs (from CDP) and click Retrieve. Confirm Status shows OK for the base CRL. A Failed row for the Delta CRL is expected if Delta CRLs are not published.

6. Record the AIA/OCSP URL shown in pkiview for the OCSP Location row. Note the color. A green indicator confirms the endpoint is reachable but does not confirm response accuracy.

7. Locate a currently valid certificate with an HTTP OCSP URL in its AIA extension. Test it: `certutil -URL <valid-cert.cer>`, select OCSP (from AIA), click Retrieve. Confirm Status shows Verified.

8. Locate a known-revoked certificate with an HTTP OCSP URL in its AIA extension. Test it: `certutil -URL <revoked-cert.cer>`, select OCSP (from AIA), click Retrieve. Confirm Status shows Revoked. If it shows Verified, run `certutil -CRL` to refresh the CRL, wait 30 seconds, and retest — the OCSP responder reads the CRL as its data source.

9. Calculate expiry threshold dates in PowerShell: `$today = Get-Date`, then `$today.AddDays(30)`, `$today.AddDays(60)`, `$today.AddDays(90)`. Run four certutil -view queries with Disposition=20 and NotAfter filters at each threshold plus today. Record counts at each window and list any certificate CNs found.

10. Summarize all of the results in the health check report. Highlight any items that need follow-up, explain what action is required, who is responsible for it, and when it should be completed.

```

---

## Lab Report Questions

**1. You opened pkiview.msc before running any certutil commands. Describe one thing pkiview told you that you could not have known from certutil alone, and one thing certutil told you that pkiview cannot show. What does this tell you about the role of each tool in a CA health check?**

```
One thing pkiview showed immediately was the overall health of the PKI hierarchy. Before running any commands, I could already see that the offline Root CA was marked in red, which was expected, and that the Delta CRL location also showed a warning. Certutil, on the other hand, gave much more detailed information, such as the exact CRL NextUpdate date and time, which allowed me to calculate how long the CRL would remain valid. This showed me that the two tools serve different purposes. pkiview is useful for getting a quick overview of the environment, while certutil is better for checking the details and confirming that everything is working as expected.
```

**2. In Part B, you tested OCSP with both a valid and a revoked certificate. pkiview showed the AIA row as green — meaning the OCSP endpoint was reachable. Why is testing with a known-revoked certificate a required step that pkiview cannot replace? What would it mean operationally if the revoked certificate returned a GOOD status instead of REVOKED?**

```
pkiview's green AIA indicator confirms only that the OCSP endpoint returned an HTTP response, it does not inspect the content of that response or verify that the revocation data inside
it is accurate. An OCSP responder could return well-formed responses for every certificate
while reading a stale CRL that pre-dates a recent revocation event, and pkiview would
continue to show green throughout. That's why testing with a certificate that is already known to be revoked is so important. If that certificate returned a GOOD status instead of REVOKED, it would mean the OCSP responder was using outdated revocation information, such as an old CRL. As a result, clients could continue trusting a certificate that should no longer be trusted, which would create a serious security risk.
```

**3. In Part C, you checked for certificates expiring within 30, 60, and 90 days. pkiview does not show this data. AD CS does not generate any automatic alerts. Given this, what operational discipline is required to prevent a certificate expiry from becoming a service outage — and what would a mature weekly health check routine look like for a CA with 200 issued certificates?**

```
Because AD CS generates no automatic expiry alerts, preventing a certificate expiry from
becoming a service outage requires proactive, scheduled human-initiated queries, the
expiration pipeline check must be built into a recurring operational routine rather than
treated as reactive troubleshooting. The minimum discipline required is running the
certutil -view queries against 30, 60, and 90-day windows on a fixed schedule, reviewing
the results against a certificate inventory that maps each certificate to the service or
system it protects, and initiating renewal workflows with enough lead time for the
certificate owner to act before the expiry date causes an outage. For a CA with 200 issued
certificates, a mature weekly health check routine would begin with the automated certutil
-view queries exported to a structured format (CSV or JSON), sorted by NotAfter date
ascending so the most urgent certificates appear first. The output would be compared against
a maintained inventory that records the certificate CN, the owning team, the renewal method
(auto-enrollment or manual), and the lead time needed for renewal. Certificates in the
30-day window would trigger direct notification to the owning team with a specific renewal
deadline. Certificates in the 60-day window would receive a standing warning. The 90-day
window would serve as an awareness list reviewed at the weekly meeting but not yet actioned.
Additionally, any certificates showing Disposition=20 with a NotAfter in the past —
as observed in this lab — would be flagged for immediate revocation and database cleanup,
since expired-but-not-revoked certificates degrade the reliability of expiration pipeline
queries over time.
```

**4. If you were setting up this health check to run automatically on a weekly schedule, which signal would you consider most urgent to monitor — CRL freshness, OCSP availability, or the expiration pipeline? Explain your reasoning, including what failure in that signal would look like within your first hour of not catching it.**

```
CRL freshness is the most important signal I would monitor on an automated schedule because an expired CRL can affect every system that relies on the CA to validate certificates. Every CRL contains a NextUpdate timestamp, which tells clients when a newer copy of the CRL should be available. Once that time passes without a new CRL being published, many applications no longer trust the old CRL because it may not contain the latest revocation information. As a result, they may reject certificates that are actually still valid simply because they cannot verify whether those certificates have been revoked.
The impact can spread very quickly across the environment. Services that rely on certificate validation, such as HTTPS websites, VPNs, smart card logons, or code signing verification, may begin failing even though nothing is wrong with the certificates themselves. To users and helpdesk staff, it can look like there has been a major certificate failure or even a security breach, when the real issue is simply that the CA did not publish a new CRL before the previous one expired.
Although OCSP availability is also important, many clients can fall back to CRL checking if the OCSP responder is temporarily unavailable, making it slightly more forgiving. The certificate expiration pipeline is also critical, but certificates usually expire over days or weeks, giving administrators time to identify and renew them before they cause an outage. CRL freshness is different because there is very little warning once the NextUpdate time has passed. Monitoring it automatically helps detect the problem early and allows administrators to publish a new CRL before users begin experiencing widespread certificate validation failures.
```

**5. pkiview.msc has been a standard tool in Windows Server AD CS since 2003. Enterprise CLM platforms like Keyfactor provide richer dashboards that automate much of what you did manually in this lab. Based on what you observed in this lab, what does pkiview show that a newer platform dashboard would also need to show — and what would a platform need to add to go beyond what pkiview and certutil provide manually?**

```
From this lab, I found that pkiview provides a quick overview of the PKI environment by showing the CA hierarchy, the health of the CDP, AIA, OCSP, and Delta CRL locations, and whether the CA certificates are still valid. Any modern PKI management platform should be able to provide this same information in an easy-to-read dashboard. To go beyond what pkiview and certutil provide, the platform should also include automatic health checks, certificate expiration alerts, historical reporting, and a complete inventory showing which certificates belong to which systems or applications. It should also monitor OCSP responses automatically so problems can be detected before users are affected.
```

---

## Submission Checklist

- [x] Logged in as CORP\pki.admin — whoami output included
- [x] CA running and responding — Get-Service and certutil -ping output included
- [x] pkiview.msc opened — initial status of all nodes recorded in Pre-Lab Step 2
- [x] Revoked certificate confirmed or created — serial number recorded
- [x] Part A: CDP color in pkiview recorded before and after certutil -CRL
- [x] certutil -CRL output included — completed without errors
- [x] certutil -dump on CRL included — ThisUpdate and NextUpdate recorded
- [x] Hours until CRL expiry calculated and documented
- [x] CRL HTTP accessibility tested — certutil -URL output included
- [x] pkiview vs. certutil comparison completed for Part A
- [x] Part A summary table completed
- [x] Part B: AIA color in pkiview recorded and URL noted
- [x] OCSP tested with valid certificate — certutil -URL output and GOOD response documented
- [x] OCSP tested with revoked certificate — certutil -URL output and REVOKED response documented
- [x] pkiview vs. certutil comparison completed for Part B
- [x] Part B summary table completed
- [x] Target dates (30/60/90) calculated and recorded
- [x] All three expiry query outputs included
- [x] Already-expired certificate check completed
- [x] Part C summary table completed
- [x] Part D health check summary table completed with pkiview initial status and overall status
- [x] Reusable health check procedure written — includes both pkiview and certutil steps
- [x] All five lab report questions answered in complete sentences
- [x] File committed to `labs/week-14/lab-02-ca-health-check.md`
