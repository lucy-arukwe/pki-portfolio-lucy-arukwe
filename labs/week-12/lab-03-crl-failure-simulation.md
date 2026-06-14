# Lab 03: Simulate a CRL Publication Failure *(Stretch)*

**Student Name:** Lucy Arukwe 
**Date Completed:**  June 10, 2026
**Phase:** 2 | **Week:** 12  
**Submission Path:** `labs/week-12/lab-03-crl-failure-simulation.md`

---

> **Stretch Lab:** Lab 03 is optional but strongly recommended. It is the highest-value evidence artifact of Week 12. The incident report format you produce here — failure, detection, resolution — is the same format used for real PKI infrastructure incidents throughout your career.

---

## Prerequisites

Complete Labs 01 and 02 before starting Lab 03.

- Lab 01 complete: a revoked certificate exists in the CA
- Lab 02 complete: the Online Responder is configured and operational

---

## Pre-Lab Verification

If you can log into PKI-SRV01 as CORP\pki.admin, you are communicating with DC01 and the environment is ready. Do not proceed until both service checks pass.

### Step 1 — Confirm Services Are Running

1. Open an elevated PowerShell prompt (right-click → **Run as Administrator**)
2. Run both checks:

```powershell
# Check 1 — CA service
Get-Service -Name CertSvc

# Check 2 — OCSP service
Get-Service -Name OCSPSvc
```

Expected: both show `Status = Running`. If either is stopped, troubleshoot before continuing — Lab 03 requires both services.

**Both services running:**
- [x] Yes
- [ ] No — describe the issue and how you resolved it:

```
(describe here)
```

### Step 2 — Record the Current CRL Publication Settings

> **Why record these first?** You are about to change these registry values to create a simulated failure. If you do not record the originals before making changes, you will not be able to restore the environment accurately. This step is mandatory.

1. Run the following four commands and record each value in the table below:

```powershell
certutil -getreg CA\CRLPeriod
certutil -getreg CA\CRLPeriodUnits
certutil -getreg CA\CRLDeltaPeriod
certutil -getreg CA\CRLDeltaPeriodUnits
```

**Current CRL publication settings — record before making any changes:**

| Setting             | Current Value | Lab Default (for reference) |
|---------------------|---------------|-----------------------------|
| CRLPeriod           | Weeks         | Weeks                       |
| CRLPeriodUnits      | 1             | 1                           |
| CRLDeltaPeriod      | Days          | Days                        |
| CRLDeltaPeriodUnits | 1             | 1                           |

> **If your values differ from the Lab Default column:** That is fine — record what your environment actually shows. Use those values (not the reference values) when you restore in Part D.

### Step 3 — Record the Current CRL NextUpdate

1. Run the following command to capture the current CRL expiry time before the failure:

```powershell
certutil -dump "C:\Windows\System32\CertSrv\CertEnroll\CVI Issuing CA 1.crl" | findstr "NextUpdate\|ThisUpdate"
```
The `findstr "NextUpdate\|ThisUpdate"` command returned no output in this environment. The full `certutil -dump` command was used instead to retrieve the timestamps.

```powershell
certutil -dump "C:\Windows\System32\CertSrv\CertEnroll\CVI Issuing CA 1.crl"
```

**CRL timestamps before the failure:**

```
ThisUpdate (current CRL was published at): ThisUpdate: 6/10/2026 6:40 PM


NextUpdate (current CRL expires at): NextUpdate: 6/18/2026 7:00 AM
```

> Save this NextUpdate time. You will compare it to the CRL state during and after the failure in Parts B and D.

---

## Part A — Create the CRL Publication Failure

You will simulate a CRL publication failure by setting the publication interval to an extreme value — 99 years. This tells the CA it does not need to publish a new CRL for 99 years. The existing CRL remains valid until its NextUpdate timestamp passes, at which point relying parties can no longer obtain a fresh CRL from the normal schedule.

> **What this simulates:** A misconfigured CA where the CRL schedule was altered during a maintenance window and not restored. The CA continues running, certificates continue to be issued, but the CRL propagation mechanism has broken silently.

### Step 1 — Extend the CRL Period to 99 Years

1. In your elevated PowerShell prompt, run all four registry changes in sequence:

```powershell
certutil -setreg CA\CRLPeriodUnits 99
certutil -setreg CA\CRLPeriod Years
certutil -setreg CA\CRLDeltaPeriodUnits 99
certutil -setreg CA\CRLDeltaPeriod Years
```

Expected output for each command:
```
CertUtil: -setreg command completed successfully.
```

**All four registry commands completed successfully:**
- [x] Yes
- [ ] No — paste the error:

```
(paste any error output here)
```

### Step 2 — Restart the CA Service to Apply the Changes

1. Registry changes to CRL settings require a CertSvc restart to take effect:

```powershell
net stop CertSvc
net start CertSvc
```

2. Confirm the service restarted cleanly:

```powershell
Get-Service -Name CertSvc
```

Expected: `Status = Running`

```
Status   Name               DisplayName
------   ----               -----------
Running  CertSvc            Active Directory Certificate Services
```

**CertSvc status after restart:**
- [x] Running
- [ ] Stopped or error — describe:

### Step 3 — Verify the New CRL Period Is Set

1. Confirm the registry now shows the extended values:

```powershell
certutil -getreg CA\CRLPeriod
certutil -getreg CA\CRLPeriodUnits
```

Expected output:
```
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\CertSvc\Configuration\<CA-Name>:
  CRLPeriod REG_SZ = Years

HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\CertSvc\Configuration\<CA-Name>:
  CRLPeriodUnits REG_DWORD = 0x63 (99)
```

```
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\CertSvc\Configuration\CVI Issuing CA 1\CRLPeriod:
  CRLPeriod REG_SZ = Years

HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\CertSvc\Configuration\CVI Issuing CA 1\CRLPeriodUnits:
  CRLPeriodUnits REG_DWORD = 63 (99)
```

### Step 4 — Record the CRL State After the Change

1. Dump the current CRL to confirm it still exists on disk and record its NextUpdate:

```powershell
certutil -dump "C:\Windows\System32\CertSrv\CertEnroll\CVI Issuing CA 1.crl" | findstr "NextUpdate\|ThisUpdate"
```

```
ThisUpdate (CRL was last published at): ThisUpdate: 6/10/2026 6:40 PM

NextUpdate (CRL will expire at — this is when the failure becomes observable): NextUpdate: 6/18/2026 7:00 AM
```

> **Important timing note:** The failure is not instantaneous. The existing CRL remains valid until the NextUpdate time you recorded in the Pre-Lab. Until that time passes, relying parties can still download and use the current CRL. The observable failure — where certificate verification begins to degrade — occurs when the CRL's NextUpdate passes with no new CRL published. In your lab session you may observe the failure immediately (if the CRL is already close to expiry) or you may need to wait or advance the clock. Check with your instructor if the NextUpdate is more than a few hours in the future.

---

## Part B — Observe the Failure

> **When to run Part B:** If the CRL's NextUpdate has passed, you will observe the full failure mode. If the CRL is still valid, run through each step and document whatever state you can observe — the OCSP responder state in particular may change before the full CRL expiry.

### Step 1 — Attempt Certificate Verification

1. Export a valid, non-revoked certificate to a `.cer` file if you do not already have one on disk. You can use the TLS certificate from Week 10 or any non-revoked cert from Week 11. Then run:

```powershell
certutil -verify valid-cert.cer
```

2. Note the outcome. Two possible results depending on your CRL state:

| CRL State | Expected certutil -verify Outcome |
|---|---|
| CRL still valid (NextUpdate not yet passed) | Verification succeeds — relying party uses cached or freshly downloaded CRL |
| CRL expired (NextUpdate passed, no new CRL published) | `CRYPT_E_REVOCATION_OFFLINE (0x80092013)` — cannot obtain fresh CRL |

```
ChainContext.dwInfoStatus = CERT_TRUST_HAS_PREFERRED_ISSUER (0x100)
ChainContext.dwRevocationFreshnessTime: 46 Days, 35 Minutes, 24 Seconds
CRL 19:
    ThisUpdate: 6/10/2026 6:40 PM
    NextUpdate: 6/18/2026 7:00 AM
Leaf certificate revocation check passed
```

**Verification outcome:**
- [x] Succeeded — CRL is still within its validity window
- [ ] Failed with CRYPT_E_REVOCATION_OFFLINE — CRL has expired
- [ ] Other — describe:

### Step 2 — Test the OCSP Responder

1. Run the URL retrieval test against the same certificate:

```powershell
certutil -URL valid-cert.cer
```

2. In the URL Retrieval Tool window, select **OCSP** from the dropdown and click **Retrieve**
3. Also select **CRL** and click **Retrieve** to compare both revocation methods

**OCSP response after CRL schedule is broken:**
- [x] Good — OCSP responder has not yet detected the stale CRL
- [ ] Unknown — OCSP responder has detected the stale CRL and cannot confirm status
- [ ] Connection error

**CRL retrieval result:**
- [x] Retrieved successfully — CRL is still within its validity window
- [ ] Expired or unavailable

```
OCSP (from AIA):
Status: Verified
Type: OCSP
URL: [0.0] http://pki-srv01.corp.cvilab.local/ocsp
The OCSP responder returned a valid response for the certificate. Because the existing
CRL had not yet expired, the Online Responder was still able to serve accurate responses
based on its cached CRL data.

CRLs from CDP:
Status: Verified — Base CRL retrieved successfully
URL: [0.0] http://pki-srv01.corp.cvilab.local/CertEnroll/CVI Issuing CA 1.crl
Status: Failed — Delta CRL returned 404
URL: [0.0.0] http://pki-srv01.corp.cvilab.local/CertEnroll/CVI Issuing CA 1+.crl
The base CRL was still downloadable and within its validity window. 
```

> **Why OCSP may still return Good temporarily:** The Online Responder caches the CRL it reads from the CA. If the CRL has not expired yet, the Online Responder continues serving responses based on the still-valid CRL. Once the CRL passes its NextUpdate, the OCSP responder will no longer be able to validate its CRL data and will return Unknown status. This dependency — OCSP accuracy depends entirely on CRL freshness — is a core concept of Week 12.

### Step 3 — Check Event Viewer for CA Events

1. Query the Application event log for CA-related events:

```powershell
Get-EventLog -LogName Application -Source "CertSvc" -Newest 30 |
  Where-Object { $_.EventID -in 46, 47 } |
  Format-List TimeGenerated, EventID, Message
```

**Event ID reference:**

| Event ID | Source | Meaning | When It Fires |
|---|---|---|---|
| 46 | CertSvc | CRL published successfully | Each time a new CRL is written to disk |
| 47 | CertSvc | CRL publication failed | Only when the CA attempts publication and fails — NOT when the CRL simply expires between scheduled intervals |

> **If you see no Event 47:** This is expected and is itself a finding. With CRLPeriodUnits set to 99 Years, the CA believes its next scheduled publication is 99 years away. It never attempts to publish — so it never generates a failure event. The absence of Event 47 does not mean the system is healthy. In a production environment, this is why monitoring must check the CRL's NextUpdate value directly, not just watch for error events.

**Events found:**

| Event ID | Timestamp            | Description     |
|----------|----------------------|-----------------|
| 26       | 6/10/2026 8:00:32 PM | CertSvc started |
| 38       | 6/10/2026 8:00:18 PM | CertSvc stopped |

```
No Event 46 (CRL published) and no Event 47 (CRL publication failed) were observed.

With CRLPeriodUnits set to 99 Years, the CA calculates its next scheduled publication as
approximately 99 years in the future. Because the CA never attempts to publish, it never
encounters a failure and therefore never generates Event 47. The absence of Event 47 does
not indicate the system is healthy — it indicates the CA is not even trying to publish.
This is a critical monitoring gap.
```

---

## Part C — Diagnose the Failure

Document your diagnostic process as if you are responding to an operational incident — not as if you already know the cause. The goal is to demonstrate that you can trace from symptom to root cause using the tools available.

### Step 1 — Identify What a Relying Party Would Experience

1. Based on your observations in Part B, describe the symptom a relying party would encounter:

```
During the silent phase of the failure, while the existing CRL was still within its validity
window, a relying party would experience no visible impact. Certificate verification would
succeed, OCSP responses would return Good, and no errors would be logged. The failure is
completely invisible until June 18, 2026 at 7:00 AM when the CRL's NextUpdate passes.

After that point, a relying party attempting to verify a certificate would receive
CRYPT_E_REVOCATION_OFFLINE (0x80092013) if configured for hard-fail, or would silently
accept the certificate if configured for soft-fail. In either case, the relying party has
no way to confirm current revocation status because no fresh CRL exists.
```

### Step 2 — Identify the Hard-Fail or Soft-Fail Behavior

1. Review your certutil -verify output from Part B Step 1 and determine which behavior your environment exhibited:

**Hard-fail vs. soft-fail reference:**

| Behavior | certutil -verify Output | What It Means |
|---|---|---|
| Hard-fail | `CRYPT_E_REVOCATION_OFFLINE (0x80092013)` — verification fails | The application refuses the certificate when revocation status cannot be confirmed. More secure, but all certificate operations fail when the CRL is unavailable. |
| Soft-fail | Verification completes successfully — no revocation error | The application accepts the certificate when revocation cannot be checked. Less disruptive, but a revoked certificate remains trusted if the CRL is stale or offline. |

**Your environment's revocation fail behavior:**
- [ ] Hard-fail — verification failed with a revocation error
- [x] Soft-fail — verification succeeded despite the stale or expired CRL
- [ ] Could not determine — describe what you observed:

**Explain how you determined this:**

```
The certutil -verify command completed successfully and showed "Leaf certificate revocation check passed" with no revocation errors. This showed that the certificate was still being trusted even though the CRL publication schedule had been changed.

The reason verification still worked was because the existing CRL had not yet expired. The relying party was able to use that CRL to check the certificate status, so no errors were reported. This indicated soft fail behavior because certificate validation continued to succeed even though the CA would not publish another CRL under the current configuration.
```

### Step 3 — State the Root Cause

1. Identify the specific root cause of the failure using the evidence you collected:

```
The root cause of the issue was a change to four CA registry settings. CRLPeriodUnits was changed from 1 to 99 and CRLPeriod was changed from Weeks to Years. The same changes were made to the Delta CRL settings.

After CertSvc was restarted, the CA calculated its next CRL publication far into the future and stopped publishing new CRLs on its normal schedule. The CRL that already existed remained valid until its NextUpdate time. Once that time passed, no new CRL would be available for relying parties to download, and the OCSP responder would no longer have current revocation information to work with.
```

### Step 4 — Describe the Production Detection Gap

1. Explain what in this scenario would have alerted you to the failure in a production environment — and what would not:

```
Event ID 47 (CRL publication failed) would not have appeared in this scenario because the CA never attempted to publish a new CRL. Event 47 is only generated when a CRL publication is attempted and fails. Since the publication schedule was changed to 99 years, the CA believed it did not need to publish another CRL for a very long time, so no failure event was created.

This means that monitoring only for Event ID 47 would not have detected the problem. A better approach would be to monitor the CRL's NextUpdate value and create an alert before the CRL expires. Monitoring the CRL publication settings would also help identify unexpected changes before they affect certificate validation.
```

---

## Part D — Restore Normal Operation

> **Do not skip any step in this part.** Leaving the CRL schedule misconfigured will break Week 13's lab environment. Restore all four registry values, restart CertSvc, force a fresh CRL, and verify before closing.

### Step 1 — Restore the Original CRL Registry Settings

1. Run all four restore commands, using the values you recorded in the Pre-Lab. If your environment was at the lab defaults, the commands below are correct. If your original values differed, substitute them:

```powershell
certutil -setreg CA\CRLPeriodUnits 1
certutil -setreg CA\CRLPeriod Weeks
certutil -setreg CA\CRLDeltaPeriodUnits 1
certutil -setreg CA\CRLDeltaPeriod Days
```

Expected output for each:
```
CertUtil: -setreg command completed successfully.
```

```
All four values restored:
- CRLPeriodUnits: 99 → 1
- CRLPeriod: Years → Weeks
- CRLDeltaPeriodUnits: 99 → 1
- CRLDeltaPeriod: Years → Days
```

**All four registry values restored:**
- [x] Yes
- [ ] No — describe which values differ and why:

### Step 2 — Restart the CA Service

1. Restart CertSvc to apply the restored registry values:

```powershell
net stop CertSvc
net start CertSvc
```

2. Confirm the service is running:

```powershell
Get-Service -Name CertSvc
```

```
The Active Directory Certificate Services service is starting.
The Active Directory Certificate Services service was started successfully.

Status   Name               DisplayName
------   ----               -----------
Running  CertSvc            Active Directory Certificate Services
```

### Step 3 — Force an Immediate CRL Publication

1. Publish a new CRL immediately — do not wait for the schedule:

```powershell
certutil -CRL
```

Expected output:
```
CRL published successfully.
CertUtil: -CRL command completed successfully.
```

```
CertUtil: -CRL command completed successfully.
```

**CRL published successfully:**
- [x] Yes
- [ ] No — describe the error:

### Step 4 — Verify the New CRL Has a Future NextUpdate

1. Inspect the newly published CRL:

```powershell
certutil -dump "C:\Windows\System32\CertSrv\CertEnroll\CVI Issuing CA 1.crl" | findstr "NextUpdate\|ThisUpdate"
```

```
New ThisUpdate (CRL published at): 6/10/2026 8:06 PM


New NextUpdate (CRL expires at — confirm this is approximately 1 week from now):  6/18/2026 8:26 AM
```

**NextUpdate is in the future:**
- [x] Yes
- [ ] No — describe:

### Step 5 — Confirm Certificate Verification Succeeds

1. Re-run verification on the valid certificate from Part B:

```powershell
certutil -verify valid-cert.cer
```

Expected: verification completes without a revocation error.

```
ChainContext.dwInfoStatus = CERT_TRUST_HAS_PREFERRED_ISSUER (0x100)
CRL 19: ThisUpdate 6/10/2026 6:40 PM / NextUpdate 6/18/2026 7:00 AM
Delta CRL 1a: ThisUpdate 6/10/2026 8:06 PM / NextUpdate 6/12/2026 8:26 AM
Leaf certificate revocation check passed
CertUtil: -verify command completed successfully.
```

### Step 6 — Confirm OCSP Responder Returns Correct Status

1. Re-run the URL retrieval test:

```powershell
certutil -URL valid-cert.cer
```

Select **OCSP** → click **Retrieve**. Expected: Good for valid certificate.

```
Status: Verified
Type: OCSP
URL: [0.0] http://pki-srv01.corp.cvilab.local/ocsp
```

**Restoration verified:**
- [x] certutil -verify succeeds
- [x] OCSP returns Good for valid certificate

---

## Part E — Structured Incident Report

Write your incident report using the standard format: **Failure → Detection → Resolution**.

This is the primary deliverable for Lab 03. Write it as you would for a real PKI operations incident — in past tense, in complete prose sentences, with timestamps where available. Do not use bullet points in any section. The incident report is graded on clarity and technical accuracy, not length.

---

### Incident Report: CRL Publication Failure Simulation

**Incident Date:** June 10, 2026 
**Environment:** corp.cvilab.local  
**Incident Type:** CRL publication schedule failure — simulated  

---

#### Section 1: Failure

*Describe what happened, what caused it, and when the failure began. Include the specific registry values that were changed and the mechanism by which those changes led to CRL expiry.*

```
On June 10, 2026, a simulated CRL publication failure was introduced on PKI-SRV01 by modifying four CA registry values using certutil. The CRLPeriodUnits value was changed from 1 to 99 and CRLPeriod was changed from Weeks to Years. The same change was applied to the Delta CRL settings. CertSvc was restarted at approximately 8:00 PM to apply the changes. From that point forward, the CA calculated its next scheduled CRL publication as 99 years in the future and stopped generating new CRLs entirely.

The failure entered a silent phase immediately after the restart. The existing CRL on disk, published earlier that day at 6:40 PM with a NextUpdate of June 18, 2026 at 7:00 AM, remained valid and continued to be served to relying parties. Certificate verification continued to succeed and the OCSP responder continued to return Good responses because both relied on the still-valid cached CRL. No error events were generated because the CA never attempted a publication, it simply believed none was due for 99 years.

The full impact of the failure would have become observable on June 18, 2026 at 7:00 AM when the CRL's NextUpdate passed with no new CRL published.
```

**Supporting evidence — CRL inspection at time of failure:**

```
ThisUpdate: 6/10/2026 6:40 PM
NextUpdate: 6/18/2026 7:00 AM
CRL Number: 19
```

**Supporting evidence — Event Log entries:**

| Event ID | Source  | Timestamp            | Description     |
|----------|---------|----------------------|-----------------|
| 26       | CertSvc | 6/10/2026 8:00:32 PM | CertSvc started |
| 38       | CertSvc | 6/10/2026 8:00:18 PM | CertSvc stopped |

> If no Event 47 was observed, record that explicitly and explain why in the context of how the failure was configured.

No Event 46 (CRL published successfully) and no Event 47 (CRL publication failed) were observed at any point during the failure window. This is because the CA never attempted a CRL publication after the schedule was extended to 99 years — it had nothing to fail at, so no failure event was ever raised.

---

#### Section 2: Detection

*Describe how the failure was identified — what tool or check revealed it, and whether the detection was proactive or reactive.*

```
The failure was detected through direct inspection of the CA registry settings using certutil. After observing that certificate verification was still succeeding and OCSP responses were still returning Good, the CA registry was queried to check whether the CRL publication schedule was still configured correctly. The query revealed that CRLPeriod had been changed to Years and CRLPeriodUnits was set to 99, confirming the root cause.

The detection was reactive in the context of this lab — the misconfiguration was known in advance. In a production environment, the failure would likely go undetected until the CRL expired on June 18, at which point relying parties would begin experiencing certificate verification failures with no prior warning from the event log.
```

**Diagnostic commands used (in order):**

```powershell
certutil -verify valid-cert.cer
certutil -URL valid-cert.cer
Get-WinEvent -LogName Application -MaxEvents 30 | Where-Object { $_.ProviderName -like "*cert*" }
certutil -dump "C:\Windows\System32\CertSrv\CertEnroll\CVI Issuing CA 1.crl"
certutil -getreg CA\CRLPeriod
certutil -getreg CA\CRLPeriodUnits

```

**Failure behavior observed:**

```
Soft-fail behavior was observed throughout the silent phase. certutil -verify returned
"Leaf certificate revocation check passed" with no error, even though the CA would
never publish a new CRL under the misconfigured schedule. The environment accepted
certificates without complaint because the existing CRL had not yet expired.
```

---

#### Section 3: Resolution

*Describe the steps taken to restore normal operation, in order. Be specific about each command and why it was necessary.*

```
Resolution was carried out on June 10, 2026 beginning at approximately 8:00 PM. All four CA registry values were restored to their original settings using certutil: CRLPeriodUnits was set back to 1, CRLPeriod was set back to Weeks, CRLDeltaPeriodUnits was set back to 1, and CRLDeltaPeriod was set back to Days. CertSvc was then stopped and restarted to apply the restored values.

After confirming CertSvc was running, certutil -CRL was run to force an immediate CRL publication. The first attempt returned RPC_S_SERVER_UNAVAILABLE because the service had not fully initialised; waiting approximately 30 seconds and rerunning the command succeeded. The newly published CRL showed a ThisUpdate of 6/10/2026 8:06 PM and a NextUpdate of 6/18/2026 8:26 AM, with CRL Number 1a confirming it was a fresh publication. Certificate verification was re-run using certutil -verify and completed successfully. The OCSP responder was confirmed to be returning Verified status via certutil -URL.
```

**Post-restoration certutil -verify output:**

```
ChainContext.dwInfoStatus = CERT_TRUST_HAS_PREFERRED_ISSUER (0x100)
Delta CRL 1a: ThisUpdate 6/10/2026 8:06 PM / NextUpdate 6/12/2026 8:26 AM
Leaf certificate revocation check passed
CertUtil: -verify command completed successfully.
```

**Time from failure identification to restored CRL:**

```
Time from failure identification to restored CRL:** Approximately 10 minutes.
```

---

## Part F — Reflection Questions

Answer all questions in your own words. Write in complete sentences.

**1. In a production environment, how would you know the CRL had become stale before a relying party noticed? What monitoring or alerting would you put in place — and why is watching only for Event ID 47 not sufficient?**

```
Event ID 47 only fires when the CA attempts a CRL publication and fails. In this scenario,
the CA never attempted to publish at all because the schedule was set to 99 years — so no
Event 47 was ever generated. A monitoring system that only watched for Event 47 would see
nothing wrong right up until the moment the CRL expired and relying parties started failing.

In a production environment, I would monitor the CRL NextUpdate value and create an alert when the CRL is getting close to expiring. I would also monitor the CRL publication settings so that unexpected changes are detected quickly.

Watching only for Event ID 47 is not enough because that event only appears when a CRL publication attempt fails. In this lab, the CA never attempted another publication, so there was no Event 47 even though the configuration was broken. A separate check should also monitor the
CA registry values for CRLPeriod and CRLPeriodUnits and alert on any unexpected change.
Together these two checks would catch both a broken schedule and a CRL approaching expiry,
giving the operations team time to respond before any relying party was affected.
```

**2. You observed whether your environment was hard-fail or soft-fail. What is the security implication of soft-fail behavior specifically for a key compromise scenario — where an attacker holds a revoked certificate and the stale CRL still shows it as good?**

```
In this lab the environment exhibited soft-fail behavior, certificate verification succeeded
even after the CRL publication schedule was broken. In a key compromise scenario, this means
an attacker holding a certificate that has been revoked due to key compromise could continue
to use that certificate without any relying party detecting a problem, for as long as the
stale CRL remains within its validity window.

The CA administrator may have already revoked the certificate and published a new CRL the
moment the compromise was discovered. But if the CRL publication schedule then broke, or
if a relying party was using a cached copy of the old CRL, the attacker's certificate
would still appear as Good. In a soft-fail environment there is no safety net: the system
assumes the certificate is valid when it cannot confirm otherwise, which is the worst
possible outcome in a compromise scenario where speed of revocation is critical.
```

**3. After the CRL expired, the OCSP responder's behavior changed. Explain the dependency between CRL publication and OCSP responder accuracy — and what this means for environments that rely solely on OCSP without monitoring the underlying CRL.**

```
The Online Responder does not maintain its own independent database of revoked certificates.
It reads the CA's published CRL and uses that as the source of truth for every OCSP response
it issues. When the CRL is fresh, the OCSP responder accurately reflects the current
revocation state. When the CRL becomes stale or expires, the OCSP responder loses its
data source and can no longer confirm revocation status, at that point it returns Unknown
rather than Good or Revoked.

This means OCSP is only as accurate as the CRL it reads from. An environment that deploys
OCSP and stops monitoring the underlying CRL is building a false sense of security. The
OCSP endpoint may appear healthy and reachable while its responses are based on outdated
data. Proper PKI operations require monitoring both the OCSP service availability and the
freshness of the CRL it depends on.
```

---

## Submission Checklist

- [x] Logged in as CORP\pki.admin (not a local account)
- [x] Both services verified running: CertSvc and OCSPSvc
- [x] All four original CRL settings recorded before making changes
- [x] CRL NextUpdate recorded before failure
- [x] All four registry changes applied — CRLPeriodUnits 99, CRLPeriod Years, CRLDeltaPeriodUnits 99, CRLDeltaPeriod Days
- [x] CertSvc restarted after registry changes
- [x] certutil -verify output documented (failure or stale CRL state)
- [x] certutil -URL output documented (OCSP behavior after CRL schedule broken)
- [x] Event Viewer queried — Event 47 presence or absence documented with explanation
- [x] Hard-fail vs. soft-fail behavior identified and explained
- [x] Root cause stated in specific technical terms
- [x] All four registry values restored to original values
- [x] CertSvc restarted after restoration
- [x] certutil -CRL confirms "CRL published successfully"
- [x] New NextUpdate confirmed to be in the future
- [x] certutil -verify confirms successful verification after restoration
- [x] OCSP returns correct status after restoration
- [x] Incident report completed in prose (Failure / Detection / Resolution)
- [x] All three reflection questions answered in complete sentences
- [x] Lab file committed to `labs/week-12/lab-03-crl-failure-simulation.md`
- [x] All three reflection questions answered
- [x] Lab report committed to `labs/week-12/lab-03-crl-failure-simulation.md`
