# Lab 02: Certificate Discovery & Risk Scoring

**Student Name:** Lucy Arukwe

**Date Completed:** July 29, 2026

**Phase:** 2 | **Week:** 15

**Submission Path:** `labs/week-15/lab-02-clm-discovery-risk.md`

---

## Overview

In this lab, the first Discovery scan is run in the CLM Simulator, a browser-based tool in the lab portal requiring no VM. The simulator generates a personalized certificate fleet, seeded from a larger pool, so findings are specific to this account rather than a copy of a classmate's.

A Discovery Scan is run, the computed risk scores are read, and the fleet's highest-risk certificates are documented along with the specific violations driving each score. This lab is the foundation for Lab 03 (Policy Zones and Automation) and Lab 04 (Monitoring), all three of which use this same fleet.

---

## Lab Environment

| Component | Details |
|-----------|---------|
| Access Path | Lab portal → `/select-track` → CLM Simulator card |
| Tool | CLM Simulator (browser-based, no VM) |
| Fleet | 15 certificates, personalized per student, seeded from a 50-certificate pool |
| Tabs Used This Lab | Discovery |

---

## Pre-Lab Check — Orientation: Finding and Opening the CLM Simulator

### Step 1 — Sign In to the Lab Portal

Signed in with student account. Portal access required resolving a vendor sign-in error (`server_error - failed to sign in with vendor`) before access was granted.

### Step 2 — Navigate to Track Selection

Navigated to `/select-track`. Both the **VM Buildout** and **CLM Simulator** cards were visible.

**Track selection screen loaded with both cards visible:** Yes

### Step 3 — Confirm CLM Simulator Access and Open It

CLM Simulator card was clickable, not locked. Opened cleanly on first click.

**CLM Simulator accessible and opened:** Yes

### Step 4 — Orient Yourself to the Interface

Confirmed on the CLM Simulator page:
- Page header: "Certificate Lifecycle Management Simulator"
- Four tabs visible: Discovery, Policy, Automation, Monitoring
- Discovery was the active/highlighted tab
- Progress tracker showing "0 / 4 steps" before scanning

---

## Part A — Run the First Discovery Scan

### Step 1 — Confirm on the Discovery Tab

Discovery was already the highlighted tab on load.

### Step 2 — Locate and Click "Run Discovery Scan"

Clicked **Run Discovery Scan**. The scan completed instantly with no loading state, and the button label changed to "Scan Complete." The progress tracker updated to 1/4 steps.

### Step 3 — Read the Resulting Table

A table of 15 certificates populated the page immediately upon scan completion.

### Step 4 — Record Fleet Size and Column Structure

Total certificates discovered: **15**

Columns shown: Risk, Common Name / Host, Issuer, Expires, Key, SANs, Owner

### Step 5 — Locate the Risk Score Column and Confirm the Sort Order

The Risk column displays a colored badge (Critical/High/Medium/Low) alongside the numeric score.

**The Discovery table is sorted by risk score, descending, by default:** Yes — confirmed. The tool's own caption states: "Found 15 certificates. Sorted by risk score so the fleet's biggest problems surface first."

---

## Part B — Document the Highest-Risk Certificates

### Step 1 — Top 5 Highest-Risk Certificates

| Rank | Subject / Hostname | Issuer | Risk Score | Badge | Violations Contributing |
|---|---|---|---|---|---|
| 1 | old-wiki.corp.cvilab.local | Self-Signed | 100 | Critical | Expiring 8–30 days (+20), weak key 1024-bit RSA (+35), self-signed (+35), missing owner (+10) |
| 2 | kiosk-app03 | Self-Signed | 80 | Critical | Weak key 1024-bit RSA (+35), self-signed (+35), missing owner (+10) |
| 3 | iot-gateway-07 | Self-Signed | 80 | Critical | Weak key 1024-bit RSA (+35), self-signed (+35), missing owner (+10) |
| 4 | old-fileserver.corp.cvilab.local | Self-Signed | 80 | Critical | Weak key 1024-bit RSA (+35), self-signed (+35), missing owner (+10) |
| 5 | iot-sensor-14 | Self-Signed | 80 | Critical | Weak key 1024-bit RSA (+35), self-signed (+35), missing owner (+10) |

### Step 2 — Verify the Score Math on the #1 Certificate

`old-wiki.corp.cvilab.local` expires 2026-08-10, which is 12 days from the current date (July 29, 2026), placing it in the 8–30 day bucket.

```
Expiring 8-30 days:   +20
Weak key (1024-bit):  +35
Self-signed:          +35
Missing owner:        +10
-------------------------
Total:                100 (Critical)
```

**The calculation matches the score shown in the tool:** Yes

**Note on a separate observation:** `jenkins.corp.cvilab.local` displays a risk score of 40 (High), but its listed expiry date (2026-07-14) is already in the past relative to today's date (2026-07-29). This suggests the tool applies the same scoring bucket to already-expired certificates as it does to certificates expiring within 7 days, rather than maintaining a distinct "expired" category. This is recorded as observed tool behavior rather than corrected or smoothed over.

### Step 3 — Count Certificates by Risk Band

| Band | Score Range | Count in Fleet |
|---|---|---|
| Critical | 70–100 | 5 |
| High | 40–69 | 1 |
| Medium | 15–39 | 3 |
| Low | 0–14 | 6 |

---

## Part C — Analysis

### Step 1 — Identify Stacking Violations

`old-wiki.corp.cvilab.local` (Rank 1, score 100) is the clearest example of stacking violations in this fleet, with four violations compounding at once: expiring in 8-30 days (+20), a weak 1024-bit RSA key (+35), a self-signed/untrusted issuer (+35), and a missing owner (+10).

None of these violations would be especially urgent on their own. A 1024-bit key on a certificate with a healthy multi-year runway and a known owner is a lower-priority fix. Stacked together, however, they describe a certificate that is simultaneously weak, untrusted, unowned, and close to expiring. The score is not simply adding penalties in isolation. It is flagging a compounding operational blind spot: several independent failure modes converging on one asset that nobody appears to be watching.

### Step 2 — Preview: What Zone Does Each Top-5 Certificate Belong To?

| Rank | Certificate | Predicted Zone |
|---|---|---|
| 1 | old-wiki.corp.cvilab.local | Unmanaged/Legacy |
| 2 | kiosk-app03 | Unmanaged/Legacy |
| 3 | iot-gateway-07 | Unmanaged/Legacy |
| 4 | old-fileserver.corp.cvilab.local | Unmanaged/Legacy |
| 5 | iot-sensor-14 | Unmanaged/Legacy |

All five top-risk certificates land in the same predicted zone. Every one is self-signed with no owner tag, which is the exact pattern the naming guide points to for Unmanaged/Legacy, regardless of what the hostname itself might otherwise suggest. "old-wiki" and "old-fileserver" hint at retired or neglected assets, while the IoT devices and kiosk suggest hardware that was likely never onboarded into a managed enrollment process at all.

---

## Lab Report Questions

**1. The #1 highest-risk certificate, walk through its exact risk score calculation and explain, in plain language, why it needs attention before the other four top-5 certificates.**

`old-wiki.corp.cvilab.local` scores 100: expiring within 8 to 30 days (+20), a 1024-bit RSA key (+35), a self-signed issuer (+35), and no owner tag (+10). The total is capped at 100, Critical. What separates this certificate from the other four Critical-tier entries is the expiration component. `kiosk-app03`, `iot-gateway-07`, `old-fileserver.corp.cvilab.local`, and `iot-sensor-14` all share the same weak-key, self-signed, and missing-owner violations, but none of them are actively counting down to expiry. Once `old-wiki` expires, whatever service depends on it stops trusting the connection or fails outright, and there is no owner on record to notice or respond. The other four have time. This one does not.

**2. Compare the fleet's risk band counts to a classmate's, if accessible, or reason about it hypothetically. Why would these counts differ between students even though everyone is using the same simulator?**

The lab description confirms each student's fleet is personalized, seeded from a shared 50-certificate pool but generated individually per account. Different students draw a different subset of that pool, so the mix of expiring, weak-key, self-signed, and unowned certificates varies by draw. Two students could both have 15 certificates and still land on entirely different risk band distributions, since the underlying violations attached to each certificate are randomized per account rather than fixed.

**3. Autoenrollment, configured by hand in Lab 01, has no concept of a risk score at all. Explain specifically what autoenrollment would have to do differently to produce something like this Discovery view, and why that's outside its actual job.**

Autoenrollment just issues and renews certificates based on the template. It doesn't look at anything after that. To do what Discovery does, it would have to check every certificate's key strength, issuer, expiry, and owner, then rank them by risk. That's a monitoring job, not an enrollment job. Autoenrollment's job ends once the certificate is issued.

**4. Pick a certificate with a missing owner tag. Explain, using Lesson 1's reasoning, why this violation only adds 10 points to the risk score even though Lesson 2 describes missing ownership as one of the most operationally dangerous, quietest problems in a certificate fleet. Is the point weighting inconsistent with that framing, or does it make sense?**

Take `svc-report-generator` (Medium, score 30, missing owner) as an example. A missing owner tag is low severity on its own. The certificate can still be technically valid, correctly issued, and functioning. The danger isn't in the certificate's current state. It's in what happens when something goes wrong. A missing owner doesn't cause failure, 
but it guarantees that when failure does happen, there's no clear path to a fix. That's a different kind of risk than a weak key or an untrusted issuer, which are active weaknesses right now. The scoring isn't inconsistent. That's why the highest-risk certificates in this fleet are the ones where missing ownership stacks on top of things that are already broken.

**5. If designing this Discovery tool for a real organization, what is one additional column or data point to include that isn't currently shown, and why would it help with prioritization?**

A renewal history column. Right now the table only shows today's risk. It doesn't show if a certificate has been renewed several times with the same problems still there. Knowing old-wiki has been renewed three times still self-signed and still unowned tells you it's a process problem, not just one bad certificate.

---

## Submission Checklist

- [x] CLM Simulator access confirmed
- [x] Discovery Scan run — fleet size and column structure recorded
- [x] Default sort order (risk score, descending) confirmed
- [x] Top 5 highest-risk certificates documented with exact scores and violations
- [x] Risk score calculation shown and verified for the #1 certificate
- [x] Risk band counts completed for the full fleet
- [x] Stacking-violation example identified and explained
- [x] Zone predictions recorded for all top 5 certificates
- [x] All five lab report questions answered in complete sentences
- [x] File committed to `labs/week-15/lab-02-clm-discovery-risk.md`
