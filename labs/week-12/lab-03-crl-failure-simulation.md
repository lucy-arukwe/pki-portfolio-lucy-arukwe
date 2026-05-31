# Lab 03: Simulate a CRL Publication Failure *(Stretch)*

**Student Name:**  
**Date Completed:**  
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
- [ ] Yes
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

| Setting | Current Value | Lab Default (for reference) |
|---|---|---|
| CRLPeriod | | Weeks |
| CRLPeriodUnits | | 1 |
| CRLDeltaPeriod | | Days |
| CRLDeltaPeriodUnits | | 1 |

> **If your values differ from the Lab Default column:** That is fine — record what your environment actually shows. Use those values (not the reference values) when you restore in Part D.

### Step 3 — Record the Current CRL NextUpdate

1. Run the following command to capture the current CRL expiry time before the failure:

```powershell
certutil -dump "C:\Windows\System32\CertSrv\CertEnroll\CVI Issuing CA 1.crl" | findstr "NextUpdate\|ThisUpdate"
```

**CRL timestamps before the failure:**

```
ThisUpdate (current CRL was published at):

NextUpdate (current CRL expires at):
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
- [ ] Yes
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
(paste net stop / net start output here)
```

**CertSvc status after restart:**
- [ ] Running
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
(paste certutil -getreg output confirming Years / 99)
```

### Step 4 — Record the CRL State After the Change

1. Dump the current CRL to confirm it still exists on disk and record its NextUpdate:

```powershell
certutil -dump "C:\Windows\System32\CertSrv\CertEnroll\CVI Issuing CA 1.crl" | findstr "NextUpdate\|ThisUpdate"
```

```
ThisUpdate (CRL was last published at):

NextUpdate (CRL will expire at — this is when the failure becomes observable):
```

> **Important timing note:** The failure is not instantaneous. The existing CRL remains valid until the NextUpdate time you recorded in the Pre-Lab. Until that time passes, relying parties can still download and use the current CRL. The observable failure — where certificate verification begins to degrade — occurs when the CRL's NextUpdate passes with no new CRL published. In your lab session you may observe the failure immediately (if the CRL is already close to expiry) or you may need to wait or advance the clock. Check with your instructor if the NextUpdate is more than a few hours in the future.

---

## Part B — Observe the Failure

> **When to run Part B:** If the CRL's NextUpdate has passed, you will observe the full failure mode. If the CRL is still valid, run through each step and document whatever state you can observe — the OCSP responder state in particular may change before the full CRL expiry.

### Step 1 — Attempt Certificate Verification

1. Export a valid, non-revoked certificate to a `.cer` file if you do not already have one on disk. You can use the TLS certificate from Week 10 or any non-revoked cert from Week 11. Then run:

```powershell
certutil -verify <valid-certificate.cer>
```

2. Note the outcome. Two possible results depending on your CRL state:

| CRL State | Expected certutil -verify Outcome |
|---|---|
| CRL still valid (NextUpdate not yet passed) | Verification succeeds — relying party uses cached or freshly downloaded CRL |
| CRL expired (NextUpdate passed, no new CRL published) | `CRYPT_E_REVOCATION_OFFLINE (0x80092013)` — cannot obtain fresh CRL |

```
(paste certutil -verify output here)
```

**Verification outcome:**
- [ ] Succeeded — CRL is still within its validity window
- [ ] Failed with CRYPT_E_REVOCATION_OFFLINE — CRL has expired
- [ ] Other — describe:

### Step 2 — Test the OCSP Responder

1. Run the URL retrieval test against the same certificate:

```powershell
certutil -URL <valid-certificate.cer>
```

2. In the URL Retrieval Tool window, select **OCSP** from the dropdown and click **Retrieve**
3. Also select **CRL** and click **Retrieve** to compare both revocation methods

**OCSP response after CRL schedule is broken:**
- [ ] Good — OCSP responder has not yet detected the stale CRL
- [ ] Unknown — OCSP responder has detected the stale CRL and cannot confirm status
- [ ] Connection error

**CRL retrieval result:**
- [ ] Retrieved successfully — CRL is still within its validity window
- [ ] Expired or unavailable

```
(describe or paste certutil -URL output — note what each method returns)
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

| Event ID | Timestamp | Description |
|---|---|---|
| | | |
| | | |

```
(paste relevant Event Viewer output, or note "No Event 47 observed — CA has not attempted publication")
```

---

## Part C — Diagnose the Failure

Document your diagnostic process as if you are responding to an operational incident — not as if you already know the cause. The goal is to demonstrate that you can trace from symptom to root cause using the tools available.

### Step 1 — Identify What a Relying Party Would Experience

1. Based on your observations in Part B, describe the symptom a relying party would encounter:

```
(e.g., certificate verification failure with specific error, silent acceptance despite expired CRL — describe what you observed and what a production user or system would see)
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
- [ ] Soft-fail — verification succeeded despite the stale or expired CRL
- [ ] Could not determine — describe what you observed:

**Explain how you determined this:**

```
(reference the specific certutil -verify output and error code that led to your conclusion)
```

### Step 3 — State the Root Cause

1. Identify the specific root cause of the failure using the evidence you collected:

```
(state the root cause precisely: what registry setting was changed, why that caused the CRL to become stale, and why that affected certificate verification or OCSP responses)
```

### Step 4 — Describe the Production Detection Gap

1. Explain what in this scenario would have alerted you to the failure in a production environment — and what would not:

```
(consider: would Event 47 have fired? Would monitoring based only on error events have caught this? What proactive check would have detected the stale CRL before a relying party noticed?)
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
(paste output of all four restore commands)
```

**All four registry values restored:**
- [ ] Yes
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
(paste net stop / net start output)
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
(paste certutil -CRL output)
```

**CRL published successfully:**
- [ ] Yes
- [ ] No — describe the error:

### Step 4 — Verify the New CRL Has a Future NextUpdate

1. Inspect the newly published CRL:

```powershell
certutil -dump "C:\Windows\System32\CertSrv\CertEnroll\CVI Issuing CA 1.crl" | findstr "NextUpdate\|ThisUpdate"
```

```
New ThisUpdate (CRL published at):

New NextUpdate (CRL expires at — confirm this is approximately 1 week from now):
```

**NextUpdate is in the future:**
- [ ] Yes
- [ ] No — describe:

### Step 5 — Confirm Certificate Verification Succeeds

1. Re-run verification on the valid certificate from Part B:

```powershell
certutil -verify <valid-certificate.cer>
```

Expected: verification completes without a revocation error.

```
(paste certutil -verify output — confirm verification succeeds)
```

### Step 6 — Confirm OCSP Responder Returns Correct Status

1. Re-run the URL retrieval test:

```powershell
certutil -URL <valid-certificate.cer>
```

Select **OCSP** → click **Retrieve**. Expected: Good for valid certificate.

```
(paste or describe certutil -URL output — confirm OCSP returns Good for valid cert)
```

**Restoration verified:**
- [ ] certutil -verify succeeds
- [ ] OCSP returns Good for valid certificate

---

## Part E — Structured Incident Report

Write your incident report using the standard format: **Failure → Detection → Resolution**.

This is the primary deliverable for Lab 03. Write it as you would for a real PKI operations incident — in past tense, in complete prose sentences, with timestamps where available. Do not use bullet points in any section. The incident report is graded on clarity and technical accuracy, not length.

---

### Incident Report: CRL Publication Failure Simulation

**Incident Date:**  
**Environment:** corp.cvilab.local  
**Incident Type:** CRL publication schedule failure — simulated  

---

#### Section 1: Failure

*Describe what happened, what caused it, and when the failure began. Include the specific registry values that were changed and the mechanism by which those changes led to CRL expiry.*

```
(write your description here in past tense — be specific about the root cause and the timeline)
```

**Supporting evidence — CRL inspection at time of failure:**

```
(paste certutil -dump output showing the stale or expired CRL's NextUpdate)
```

**Supporting evidence — Event Log entries:**

| Event ID | Source | Timestamp | Description |
|---|---|---|---|
| | | | |

> If no Event 47 was observed, record that explicitly and explain why in the context of how the failure was configured.

---

#### Section 2: Detection

*Describe how the failure was identified — what tool or check revealed it, and whether the detection was proactive or reactive.*

```
(write your description here — what command or observation confirmed the failure? was it certutil -verify, certutil -URL, Event Viewer, or direct CRL inspection?)
```

**Diagnostic commands used (in order):**

```powershell
(list each command you ran to confirm the failure)
```

**Failure behavior observed:**

```
(describe whether you observed hard-fail or soft-fail behavior, and what that looked like in the tool output)
```

---

#### Section 3: Resolution

*Describe the steps taken to restore normal operation, in order. Be specific about each command and why it was necessary.*

```
(write your resolution narrative in past tense — include each registry restore, the service restart, the forced CRL publication, and the verification steps)
```

**Post-restoration certutil -verify output:**

```
(paste the output confirming verification succeeded after restoration)
```

**Time from failure identification to restored CRL:**

```
(estimate based on your lab session)
```

---

## Part F — Reflection Questions

Answer all questions in your own words. Write in complete sentences.

**1. In a production environment, how would you know the CRL had become stale before a relying party noticed? What monitoring or alerting would you put in place — and why is watching only for Event ID 47 not sufficient?**

```
(your answer here)
```

**2. You observed whether your environment was hard-fail or soft-fail. What is the security implication of soft-fail behavior specifically for a key compromise scenario — where an attacker holds a revoked certificate and the stale CRL still shows it as good?**

```
(your answer here)
```

**3. After the CRL expired, the OCSP responder's behavior changed. Explain the dependency between CRL publication and OCSP responder accuracy — and what this means for environments that rely solely on OCSP without monitoring the underlying CRL.**

```
(your answer here)
```

---

## Submission Checklist

- [ ] Logged in as CORP\pki.admin (not a local account)
- [ ] Both services verified running: CertSvc and OCSPSvc
- [ ] All four original CRL settings recorded before making changes
- [ ] CRL NextUpdate recorded before failure
- [ ] All four registry changes applied — CRLPeriodUnits 99, CRLPeriod Years, CRLDeltaPeriodUnits 99, CRLDeltaPeriod Days
- [ ] CertSvc restarted after registry changes
- [ ] certutil -verify output documented (failure or stale CRL state)
- [ ] certutil -URL output documented (OCSP behavior after CRL schedule broken)
- [ ] Event Viewer queried — Event 47 presence or absence documented with explanation
- [ ] Hard-fail vs. soft-fail behavior identified and explained
- [ ] Root cause stated in specific technical terms
- [ ] All four registry values restored to original values
- [ ] CertSvc restarted after restoration
- [ ] certutil -CRL confirms "CRL published successfully"
- [ ] New NextUpdate confirmed to be in the future
- [ ] certutil -verify confirms successful verification after restoration
- [ ] OCSP returns correct status after restoration
- [ ] Incident report completed in prose (Failure / Detection / Resolution)
- [ ] All three reflection questions answered in complete sentences
- [ ] Lab file committed to `labs/week-12/lab-03-crl-failure-simulation.md`
- [ ] All three reflection questions answered
- [ ] Lab report committed to `labs/week-12/lab-03-crl-failure-simulation.md`
