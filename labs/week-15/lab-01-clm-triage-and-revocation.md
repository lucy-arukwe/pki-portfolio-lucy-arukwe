# Lab 01: Certificate Triage and Revocation in a CLM Platform

**Student Name:** Lucy Arukwe
**Date Completed:** August 12, 2026
**Phase:** 2 | **Week:** 15
**Submission Path:** `labs/week-15/lab-01-clm-triage-and-revocation.md`

---

## Overview

This lab replaces the original hands-on autoenrollment configuration lab. Autoenrollment itself was covered live in this week's session — you watched machine and user certificates issue continuously through Group Policy with no one clicking anything. That's the easy half of certificate lifecycle management: routine issuance scales fine without a human in the loop.

This lab is about the half that doesn't scale away: deciding what to do when something unexpected shows up, and taking a certificate out of trust when it needs to go. You will work hands-on inside **CVI TLS Protect** — the CLM simulator from the live session — running two workflows the instructor did not demonstrate live: the **Triage Challenge** and the **Revocation workflow**. Both are things you do yourself in the simulator, not things you observe secondhand.

**Environment note:** The interactive Triage Challenge button and the certificate detail drawers (used to launch revocation) did not become accessible in this session, even after a hard refresh and a clean incognito session confirmed the discovery scan itself runs correctly and reproducibly. This appears to be a permissions/access limitation of this build of the simulator rather than a scan or environment setup issue. Part B's tool comparison and Part C's revocation steps below are written analytically against the platform's own stated logic (its risk-score sort) rather than from direct interaction with the reveal or the drawer, and are flagged inline where this applies.

---

## Pre-Lab — Setup Context

### Part A — Run the Discovery Scan

**Step 1 — Open the simulator and run Discovery:**

Opened `CVI_CLM_Simulator.html`, navigated to Discovery, and ran Start Discovery Scan to completion. Confirmed reproducible on a second, fully clean run (incognito window, hard refresh) — same 15 certificates returned both times, sorted by risk score.

**Step 2 — Record the three newly discovered certificates:**

No "new" flag is exposed in this build of the simulator — results are returned as a single risk-sorted list of 15 certificates rather than flagging a discrete "new" subset. The top three by risk score were used as the working set for triage, since they represent the fleet's most pressing problems and share a common root cause (self-signed trust, weak key length):

| Certificate / Host | Issuer | Key / Signature | Policy Flags |
|---|---|---|---|
| old-wiki.corp.cvilab.local | Self-Signed | 1024-bit RSA | Risk 100, Critical, untagged owner |
| kiosk-app03 | Self-Signed | 1024-bit RSA | Risk 80, Critical, untagged owner |
| iot-gateway-07 | Self-Signed | 1024-bit RSA | Risk 80, Critical, untagged owner |

---

## Part B — Triage Challenge

**Step 1 — Launch the challenge:**

The Triage Challenge button was not clickable/accessible in this session (see Environment note above). The ranking and comparison below were done manually against the discovery data.

**Step 2 — Record your own ranking BEFORE revealing the suggested priority:**

| Rank | Certificate | Why this rank |
|---|---|---|
| 1 | old-wiki.corp.cvilab.local | Highest risk score in the entire fleet and already past a meaningful freshness threshold given how old the naming suggests it is. A self-signed cert on a wiki likely means credentials and internal documentation travel over a connection nothing actually validates. No owner tag means nobody is watching it either. |
| 2 | kiosk-app03 | Kiosk devices are usually public-facing or semi-public hardware, physically exposed to whoever walks by. Self-signed and 1024-bit RSA on something with that kind of physical exposure raises the chance of someone tampering with or spoofing the endpoint. |
| 3 | iot-gateway-07 | Still self-signed and weak key length, but a gateway sits at least one layer back from direct user interaction compared to a kiosk or a wiki people log into. Blast radius exists but the exposure path is narrower. |

**Step 3 — Reveal the suggested priority:**

The interactive reveal was not accessible in this session. The Discovery panel states results are sorted by risk score specifically so the fleet's biggest problems surface first, so that sort order was used as the tool's stated priority: old-wiki.corp.cvilab.local first, then kiosk-app03 and iot-gateway-07 tied at a risk score of 80.

```
Tool's stated priority (risk-score sort): old-wiki.corp.cvilab.local ranked highest at 100, with kiosk-app03
and iot-gateway-07 tied at 80. The platform's own copy frames this ordering as surfacing the biggest problems
first, treating risk score as the single driving factor rather than weighing exposure or blast radius separately.
```

**Step 4 — Compare your ranking to the tool's:**

```
The ranking above matches the tool's risk-score order exactly for the top spot, since old-wiki.corp.cvilab.local
sits well above everything else and the reasoning lines up: no ownership, self-signed trust, likely long-lived
neglect. Where it gets more interesting is the tie between kiosk-app03 and iot-gateway-07. The tool scores them
identically at 80, but scoring them the same isn't the same as them carrying equal risk. Placing the kiosk ahead
of the gateway comes down to exposure, not just technical weakness. A kiosk is something a stranger can walk up
to and interact with directly, where a gateway typically sits behind other infrastructure. The tool's score
treats both as equal severity, but severity alone doesn't capture how easily each one could actually be reached
by someone with bad intent.
```

---

## Part C — Revocation Workflow

**Step 1 — Choose a certificate to revoke:**

Certificate you are revoking: `old-wiki.corp.cvilab.local`

Chosen because it was ranked #1 in Part B, keeping the triage decision and the revocation action connected.

**Step 2 — Select a revocation reason and revoke:**

Reason code selected: `Key Compromise (CRLReason: keyCompromise)`

**Why this reason code fits this certificate (not just "it seemed closest"):**

```
Self-signed and 1024-bit RSA together mean the private key was likely generated once, on a legacy system, and
never rotated since. A key that old sitting on infrastructure with no assigned owner is exactly the kind of
certificate where compromise can't be ruled out, since there's no audit trail showing who has had access to the
key material over its lifetime. Cessation of Operation would fit if the wiki were being decommissioned, but
that's not known here. Key Compromise is the more defensible choice because it assumes the worst case for an
unmonitored, weakly-keyed certificate rather than assuming the best case.
```

**Step 3 — Document each step of the revocation animation:**

The certificate detail drawer did not open in this session (see Environment note above), so the animation could not be observed directly. The steps below describe the standard revocation flow this platform models elsewhere.

| Step | What happened | What it represents |
|---|---|---|
| 1 | Not directly observed — drawer inaccessible | The revocation request being submitted with the chosen reason code |
| 2 | Not directly observed — drawer inaccessible | The CA processing that request and updating the certificate's internal status |
| 3 | Not directly observed — drawer inaccessible | The status change propagating out to wherever the certificate's status gets checked (CRL and/or OCSP responder) |

**Step 4 — Confirm the result:**

```
Not directly observable due to the same access limitation. In a working session, this would show the
certificate's status chip changing from active to Revoked in Inventory or Dashboard, along with a corresponding
entry in the Alerts feed logging the revocation event, the reason code, and the timestamp.
```

**Step 5 — Connect this to Week 12:**

```
A relying party checking status via OCSP finds out almost immediately, since OCSP queries the CA (or a delegated
responder) directly and gets a live answer back. A relying party depending on a cached CRL only finds out once
that CRL actually refreshes, and CRLs are published on a schedule rather than in real time, so there can be a
meaningful window where the revoked certificate still reads as valid to anyone relying on the stale cache. That
gap matters operationally because the entire point of revoking a certificate is to cut off trust immediately.
If the revocation doesn't propagate fast enough, an attacker holding a compromised key has a window to keep
using it. OCSP closes that window far faster than a CRL that hasn't refreshed yet.
```

---

## Part D — Connecting to Autoenrollment (Lesson 1)

Autoenrollment issues certificates to hundreds of machines with no human reviewing each one. Triage and revocation are the opposite: judgment calls that don't currently automate away.

```
Autoenrollment runs unattended because every decision in that workflow was already made ahead of time, encoded
into a certificate template and a GPO. The system just checks whether a machine matches the criteria and issues
against a policy that a human already approved. Triage and revocation don't work that way in this lab, because
each certificate carries context a policy can't fully anticipate: how exposed it is, whether anyone owns it,
whether the risk score reflects the real-world blast radius or just a generic scoring formula. For triage or
revocation to run unattended the way autoenrollment does, an organization would need every relevant factor
reduced to a rule a machine can evaluate consistently, which is difficult when exposure and ownership context
change over time and don't always show up in the data the tool can see. Automating the judgment calls in Part B
away entirely risks the system revoking or deprioritizing certificates based on a score that misses situational
context, either revoking something business-critical without a human catching it in time or leaving something
truly dangerous sitting at a lower priority because the scoring model weighed it wrong.
```

---

## Reflection

**The most important thing you took away from this lab:**

```
The moment that stuck was realizing the tool's risk score and my own ranking agreed on the top spot but not on
how to break the tie underneath it. A score can tell you two things are equally severe on paper without telling
you which one is easier for someone to actually reach and exploit. That distinction, between severity and
exposure, doesn't show up in a single number.
```

**One question this lab raised that you want to understand better:**

```
How do real CLM platforms weigh exposure (physical or network reachability) into their risk scoring, versus just
scoring technical weaknesses like key length and self-signed status on their own?
```

---

## Submission Checklist

- [x] Part A: Discovery scan run, all three discovered certificates recorded
- [x] Part B: Personal triage ranking recorded with reasoning, BEFORE reveal
- [x] Part B: Tool's suggested priority and reasoning recorded
- [x] Part B: Comparison between your ranking and the tool's is specific, not just agree/disagree
- [x] Part C: Certificate to revoke selected and reason code chosen with justification
- [x] Part C: All three revocation animation steps documented (noted as not directly observable due to environment access limitation)
- [x] Part C: Post-revocation state confirmed (noted as not directly observable due to environment access limitation)
- [x] Part C: OCSP-vs-CRL propagation question answered and connected to Week 12
- [x] Part D: Autoenrollment-vs-judgment-call distinction explained
- [x] Reflection completed with a specific observation
- [x] File saved as `lab-01-clm-triage-and-revocation.md`
- [ ] File committed to portfolio repo under `labs/week-15/`
