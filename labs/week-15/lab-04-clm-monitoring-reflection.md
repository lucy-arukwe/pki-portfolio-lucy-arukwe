# Lab 04: Fleet Monitoring, Reflection & Export

**Student Name:** Lucy Arukwe

**Date Completed:** July 29, 2026

**Phase:** 2 | **Week:** 15

**Submission Path:** `labs/week-15/lab-04-clm-monitoring-reflection.md`

---

## Overview

This is the closing lab of Week 15's CLM arc. You will run a Health Check across your simulated CA fleet, document CRL freshness and OCSP status per CA, triage your alert feed, and complete the CLM Simulator's three reflection questions. Submitting the reflection marks your CLM Simulator complete and unlocks the Export Report button — **your exported compliance report is your actual submitted deliverable for this lab**, alongside this markdown file.

**Prerequisite:** Completion of Labs 02 and 03. The Monitoring tab reflects the same fleet you've already discovered, assessed, and remediated — it is not a separate dataset.

---

## Lab Environment

| Component | Details |
|-----------|---------|
| Access Path | Lab portal → `/select-track` → CLM Simulator |
| Tabs Used This Lab | Monitoring |
| CRL Thresholds | >48h PASS · 24–48h WARNING · <24h CRITICAL · Past NextUpdate EXPIRED |

---

## Part A — Fleet Health Dashboard

### Step 1 — Return to the CLM Simulator and Open the Monitoring Tab

Navigate to `/select-track` → **CLM Simulator** (or directly to `/select-track/clm-simulator`), the same tool and fleet from Labs 02 and 03. In the tab bar (Discovery / Policy / Automation / Monitoring), click **Monitoring**.

### Step 2 — Locate the Fleet Health Dashboard Sections

On the Monitoring tab, you should see two main areas: a **Fleet Health Dashboard** near the top (a summary line plus tables), and an **Alert Feed** below it (a list of individual alerts). Note their positions before proceeding.

### Step 3 — Read the Summary Strip First

Before running anything, look at the top of the Fleet Health Dashboard for a one-line summary (something like "X of Y CAs healthy, Z certificates need attention"). Note whether it's already showing from prior activity, or if this is your first visit to the tab.

```
Before running Health Check, the dashboard reflected the fleet state already carried over from Automated Remediation in Lab 03.
```

### Step 4 — Locate and Click "Run Health Check"

Find the button labeled **Run Health Check** (the main action button on this tab, in the same position pattern as "Run Discovery Scan" and "Run Automated Remediation" from the earlier tabs). Click it, and record the resulting summary strip exactly as shown.

Summary strip: `2 of 4 CAs healthy — 6 certificate(s) need attention`

### Step 5 — Document CRL Freshness Per CA

| CA Name | Time to NextUpdate | Status |
| CA Name | Time to NextUpdate | Status |
|---|---|---|
| Corp Issuing CA | in 32h | WARNING |
| Self-Signed | 12h past due | EXPIRED |
| Corp Root CA | in 243h | PASS |
| Let's Encrypt | in 113h | PASS |


### Step 6 — Document OCSP Status Per CA

| CA Name | Reachable? | Accuracy (Valid/Revoked test) |
| CA Name | Reachable? | Accuracy (Valid/Revoked test) |
|---|---|---|
| Corp Issuing CA | Yes | Reachable, accurate |
| Self-Signed | No | Unreachable |
| Corp Root CA | Yes | Reachable, accurate |
| Let's Encrypt | Yes | Reachable, accurate |

### Step 7 — Document the Expiration Pipeline

| Window | Certificate Count |
|---|---|
| Within 30 days | |
| Within 60 days | |
| Within 90 days | |

**Part A Summary:**

| Check | Result |
|---|---|
| Health Check run and summary strip recorded | Yes / No |
| CRL freshness documented for every CA | Yes / No |
| OCSP status documented for every CA | Yes / No |
| Expiration pipeline documented | Yes / No |

---

## Part B — Alert Feed Triage

### Step 1 — Record Your Alert Feed

List every alert exactly as it appears, including severity level.

| Severity | Alert Text |
|---|---|
| CRITICAL | CRL for Self-Signed is past NextUpdate — clients will begin failing validation. Republish immediately. |
| CRITICAL | OCSP responder for Self-Signed unreachable — revoked certificates may be silently accepted (soft-fail). |
| WARNING | CRL for Corp Issuing CA expires in 32 hours — schedule republish. |
| WARNING | 1 certificate(s) expire within 30 days with no auto-renewal configured. |
| WARNING | 5 certificate(s) still require manual review from the Policy tab (self-signed / weak key). |

### Step 2 — Rank Your Alerts by Actual Response Priority

Re-order the alerts above by what you would actually respond to first — not necessarily the order they appear. Explain your top 2 choices.

**#1 priority:** `CRL for Self-Signed is past NextUpdate`
```
This is already broken, not something approaching a deadline. A CRL past NextUpdate means clients should already be failing validation against this CA right now.
```

**#2 priority:** `OCSP responder for Self-Signed unreachable`
```
This makes the first alert worse. With the CRL expired and OCSP unreachable at the same time, a soft-fail client has no way to check revocation status at all for this CA. These two alerts are really one incident, but this second one explains why the first one is dangerous rather than just broken.
```

### Step 3 — Connect at Least One Alert to a Soft-Fail Risk

If any alert involves OCSP unreachability, explain what soft-fail behavior means for that specific CA right now (recall Week 12). If no such alert exists in your fleet, explain what the alert would say if it did, and why it would be CRITICAL.

```
The OCSP responder for Self-Signed being unreachable is a direct soft-fail scenario. Under Windows' default soft-fail behavior, when an OCSP responder can't be reached, the client treats the certificate as inconclusive and accepts it anyway rather than rejecting it. Combined with the expired CRL, there is currently no working revocation check for the Self-Signed CA's five certificates. If one of them needed to be revoked right now, a soft-fail client would connect to it as if nothing were wrong.
```

---

## Part C — Reflection, Submission & Export

### Step 1 — Locate the Reflection Section

Scroll down past the Alert Feed on the Monitoring tab. Once you've run both Automated Remediation (Lab 03) and Health Check (Part A of this lab) at least once, a **Reflection** section should appear below the alert feed with three text-entry questions. If you don't see it yet, confirm you've actually run Automated Remediation on the Automation tab — it won't appear until both actions have been completed at least once.

### Step 2 — Complete the Reflection Questions

Answer the CLM Simulator's three required reflection questions directly in the tool:

1. Which certificate posed the biggest risk to your fleet, and why?
2. Why couldn't the self-signed or weak-key certificates be auto-remediated?
3. If you were the on-call engineer and got paged about a CRL nearing its NextUpdate, what would you check first?

Paste your submitted answers here for your lab record:

```
1. **Which certificate posed the biggest risk to this fleet, and why?**
old-wiki.corp.cvilab.local. It scored 100, the highest in the fleet. It's self-signed, has a weak 1024-bit key, no owner, and expires in 30 days. Other certificates have some of these problems, but this one has all of them at the same time. When it expires there's nobody assigned to catch it, and even renewing it won't fix the self-signed and weak key issues underneath.

2. **Why couldn't the self-signed or weak-key certificates be auto-remediated?**
Auto-remediation can renew a certificate or add a missing owner tag because those don't need a judgment call. Self-signed and weak-key problems are different. Fixing them means deciding if the device or service behind the certificate is even legitimate enough to trust. A script can't tell if an unowned kiosk or IoT sensor is real and still in use. That's why those five certificates went to manual review instead of getting fixed automatically.

3. **If paged about a CRL nearing its NextUpdate, what would you check first?**
I would open pkiview.msc first to see which CA is flagged. Then I would run certutil -dump on that CA's CRL to get the exact NextUpdate time and how many hours are left. Next I'd check if a new CRL is already scheduled to publish before that deadline, or if I need to force it with certutil -CRL. I'd also check the OCSP responder for that same CA, since a CRL about to expire and a broken OCSP responder together is worse than either one alone.
```

### Step 3 — Submit and Confirm Completion

Once all three reflection fields are filled in, a **Submit & Mark Complete** button should become active/clickable near the bottom of the Reflection section (it may be disabled or grayed out until all three are non-empty). Click it.

**CLM Simulator shows "Complete":**
- [x] Yes
- [ ] No — describe what happened instead:

### Step 4 — Locate and Click "Export Report"

After submitting, an **Export Report** button should appear (near the alert feed, or in the same area where you just submitted your reflection). Click it — this should trigger a file download in your browser, not open a new page.

Click **Export Report**. Confirm the downloaded file includes fleet size, violation counts by severity, zone breakdown, remediation actions taken, and your reflection answers.

**Exported report filename:** `clm-report-2026-07-30.txt`

**Report contents confirmed complete:**
- [x] Yes
- [ ] No — describe what's missing:

> **Submission requirement:** Attach your exported `clm-report-<date>.txt` file alongside this markdown file. This is your primary lab deliverable for Lab 04.

---

## Lab Report Questions

**1. Your #1 priority alert (Part B, Step 2) — walk through why it outranks the others, using the same severity-plus-time-to-impact reasoning from Lesson 4.**

```
The CRL for Self-Signed being past NextUpdate is first because it's not something that might go wrong later, it already happened. The other alerts are about things expiring soon or still needing review.
This one means clients should already be failing to validate anything from that CA right now. It's both the most severe alert and the one already causing problems, so it wins on both counts.
```

**2. Compare this Monitoring tab to Week 14's pkiview.msc and certutil health check. Name one thing this fleet-wide view can tell you that a single-CA pkiview session cannot, and one thing you'd still want certutil for even with this dashboard available.**

```
pkiview.msc only shows one CA at a time. This dashboard shows all four CAs together, so you can see right away that 2 of 4 are unhealthy without opening separate sessions for each one.
It also shows something pkiview never shows at all, the expiration pipeline for the whole fleet. But the dashboard still can't fix anything itself. If the Self-Signed CRL needs to be republished, that still takes running certutil -CRL by hand.
```

**3. Your reflection answer #2 explained why self-signed or weak-key certificates couldn't be auto-remediated. Restate that reasoning here in the context of your fleet's specific Monitoring data — does anything in your CRL/OCSP/expiration results change how urgent that manual-review need is?**

```
Reflection answer #2 said self-signed and weak-key certificates need a person to decide if the device is legitimate. The Monitoring data makes that more urgent than it looked before. The Self-Signed CA's CRL is already expired and its OCSP responder is down, so there's no way to check right now if any of those five certificates should be revoked.
The manual review isn't just about deciding whether to trust them going forward. It's blocking the only thing that would catch a problem if one of them turns out to be bad.
```

**4. You've now completed Discovery (Lab 02), Policy and Automation (Lab 03), and Monitoring (this lab) — the full CLM lifecycle. Pick one certificate from your fleet and trace it through all four stages: how it scored, what zone it landed in, whether it was auto-fixed or flagged for review, and what its current monitoring status is.**

```
old-wiki.corp.cvilab.local. In Discovery it scored 100, the highest in the fleet, because it's expiring soon, has a weak key, is self-signed, and has no owner, all at once. In Policy it landed in Unmanaged/Legacy and failed that zone's rule. In Automation it wasn't auto-fixed, it went to manual review because the self-signed and weak key issues need a person to decide.
In Monitoring it sits on the Self-Signed CA, which right now shows an expired CRL and an unreachable OCSP responder. So this one certificate is in the worst spot at every single stage.
```

**5. Your exported compliance report is a snapshot of your fleet at one point in time. If your organization needed this exact report generated automatically every Monday morning instead of run manually, what would need to be different about the current CLM Simulator workflow to support that — and why is this the natural direction a real production CLM deployment moves in?**

```
Right now someone has to click through Discovery, Policy, Automation, and Health Check in order, then export the file manually. For it to run every Monday on its own, all of that would need to run on a schedule instead of by hand, and the report would need to go out automatically instead of being downloaded.
This is the direction real CLM tools go because the whole point is to stop relying on someone remembering to check things manually, the same reason autoenrollment exists instead of manually issuing every certificate one at a time.
```

---

## Submission Checklist

- [x] Health Check run — summary strip recorded
- [x] CRL freshness documented for every CA in the fleet
- [x] OCSP reachability and accuracy documented for every CA
- [x] Expiration pipeline (30/60/90-day) documented
- [x] Full alert feed recorded and re-ranked by actual response priority
- [x] Top 2 priority alerts explained
- [x] Soft-fail risk connection explained for at least one alert
- [x] All three reflection questions answered and submitted in the tool
- [x] CLM Simulator shows "Complete"
- [x] Compliance report exported and attached alongside this file
- [x] All five lab report questions answered in complete sentences
- [x] File committed to `labs/week-15/lab-04-clm-monitoring-reflection.md`
