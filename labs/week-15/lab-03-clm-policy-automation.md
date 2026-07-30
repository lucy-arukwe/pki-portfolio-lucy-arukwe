# Lab 03: Policy Zones, Violations & Automated Remediation

**Student Name:** Lucy Arukwe

**Date Completed:** July 29, 2026

**Phase:** 2 | **Week:** 15

**Submission Path:** `labs/week-15/lab-03-clm-policy-automation.md`

---

## Overview

This lab has two connected parts, using the same fleet discovered in Lab 02. In **Part A**, the Policy tab is explored, reading the fleet's zone-by-zone pass and fail rates and explaining specific violation types in plain language. In **Part B**, Automated Remediation is run and documented in detail: which certificates got auto-fixed, which need manual review, and why, grounded in the trust-decision boundary from Lesson 3.

---

## Lab Environment

| Component | Details |
|-----------|---------|
| Access Path | Lab portal → `/select-track` → CLM Simulator |
| Tabs Used This Lab | Policy, Automation |
| Zones | Web Servers, Service Accounts, Domain Infrastructure, Network/Remote Access, Unmanaged/Legacy, Other |

---

## Part A — Policy Zones & Violations

### Step 1-2 — Return to the CLM Simulator, Open the Policy Tab

Signed back in and returned to the same fleet from Lab 02. Opened the Policy tab from the tab bar.

### Step 3 — Locate the Zone Groupings

All zone groupings were visible except Domain Infrastructure, which showed zero certificates. Adding up certificates across the visible zones (Web Servers: 2, Service Accounts: 3, Network/Remote Access: 1, Unmanaged/Legacy: 5, Other: 4) accounts for all 15 certificates in the fleet, confirming Domain Infrastructure is genuinely empty for this random draw rather than hidden or broken.

A banner on the page confirmed: "9 of 15 certificates have at least one policy violation across all zones."

### Step 4 — Zone Pass/Fail Rates

| Zone | Certificates in Zone | Pass | Fail | Pass Rate |
|---|---|---|---|---|
| Web Servers | 2 | 0 | 2 | 0% |
| Service Accounts | 3 | 2 | 1 | 66.7% |
| Domain Infrastructure | 0 | — | — | N/A (no certificates in zone) |
| Network/Remote Access | 1 | 1 | 0 | 100% |
| Unmanaged/Legacy | 5 | 0 | 5 | 0% |
| Other | 4 | 3 | 1 | 75% |

### Step 5 — Lowest-Passing Zone

Web Servers and Unmanaged/Legacy are tied at a 0% pass rate, but they fail for very different reasons. Web Servers fails narrowly: both certificates (`www.cvilab.local`, `portal.cvilab.local`) fail for a single reason, approaching expiry (19d and 22d), with everything else compliant. Unmanaged/Legacy fails on stacked violations across all 5 certificates (weak key, self-signed, missing owner), with one certificate adding an expiry violation on top.

Lowest-passing zone (by severity): **Unmanaged/Legacy**

**Failures in this zone:** Spread across several violation types, consistently stacked together (weak key + untrusted issuer + missing owner, present in all 5 failing certificates).

### Step 6 — Two Violation Types Explained

**Violation type 1: Approaching expiry**

`www.cvilab.local` fails policy for expiring in 19 days, nothing else. This is the kind of failure that shouldn't require any judgment call to fix. It is already on a trusted CA with an adequate key. The only reason it is flagged is that a renewal simply has not happened yet. If this one slips past its expiry, it likely breaks HTTPS for whatever site or service depends on that hostname. The urgency here is purely a clock problem, not a trust problem.

**Violation type 2: Untrusted issuer (self-signed)**

`old-wiki.corp.cvilab.local` fails policy partly because it is self-signed rather than issued by the Corp Issuing CA. A self-signed certificate means there is no chain of trust back to anything the organization actually controls or vets. Anyone connecting to this host has to manually trust the certificate or get a security warning, and there is no CA to revoke it through if it is ever compromised. Combined with its other violations, it represents a certificate operating entirely outside the organization's actual PKI, which is exactly the pattern the Unmanaged/Legacy zone rule is designed to catch and force back under a real CA.

**Part A Summary:**

| Check | Result |
|---|---|
| All six zones reviewed | Yes (Domain Infrastructure confirmed empty, not missing) |
| Zone pass/fail rates documented | Yes |
| Lowest-passing zone identified with failure pattern | Yes |
| Two violation types explained in own words | Yes |

---

## Part B — Automated Remediation

### Step 1 — Open the Automation Tab

Opened the Automation tab from the tab bar, working against the same fleet.

### Step 2 — Predictions (Before Running)

Based on the trust-decision boundary: any certificate whose only violation is expiry or missing owner tag, issued by a trusted CA, should be auto-fixable via re-enrollment. Any certificate that is self-signed or has a weak key requires a human, since neither issue can be resolved by simply reissuing through the same broken chain of trust.

Predicted auto-fix: `jenkins.corp.cvilab.local`, `svc-report-generator`, `www.cvilab.local`, `portal.cvilab.local`, all Corp Issuing CA-issued, violations limited to expiry and/or missing owner tag.

Predicted manual review: `kiosk-app03`, `iot-gateway-07`, `old-fileserver.corp.cvilab.local`, `iot-sensor-14`, `old-wiki.corp.cvilab.local`, all self-signed with weak keys, regardless of expiry status.

### Step 3 — Actual Results

Certificates auto-fixed: **4**
Certificates needing manual review: **5**

Terminal log confirmed the mechanism for each:
- `jenkins.corp.cvilab.local`: renewed via Corp Issuing CA enrollment agent, new expiry 2027
- `svc-report-generator`: renewed via Corp Issuing CA enrollment agent, new expiry 2027, and owner auto-tagged from a CMDB lookup
- `www.cvilab.local`: renewed via Corp Issuing CA enrollment agent, new expiry 2027
- `portal.cvilab.local`: renewed via Corp Issuing CA enrollment agent, new expiry 2027

### Step 4 — Predictions vs. Actual Results

**Fully matched.** The predicted 4/5 split matched exactly, both in count and in which specific certificates landed in each group.

### Step 5 — Manual-Review Trust Decisions

| Certificate | Violation(s) | Specific Trust Decision Required |
|---|---|---|
| kiosk-app03 | Weak key (1024-bit), untrusted issuer (self-signed), missing owner tag | Whether this device can be pulled into the managed PKI at all, or must be replaced, since a self-signed cert on unknown hardware cannot simply be re-issued without first establishing what the device is and whether it should be trusted |
| iot-gateway-07 | Weak key (1024-bit), untrusted issuer (self-signed), missing owner tag | Same as above: a human must decide whether this IoT device is legitimate infrastructure or a rogue/forgotten asset before any reissuance path makes sense |
| old-fileserver.corp.cvilab.local | Weak key (1024-bit), untrusted issuer (self-signed), missing owner tag | Whether the fileserver is still in active use. If it is a decommission candidate, replacing its certificate is wasted effort. If it is still serving files, someone must accept responsibility for migrating it to the Corp CA |
| iot-sensor-14 | Weak key (1024-bit), untrusted issuer (self-signed), missing owner tag | Same device-legitimacy judgment as the other IoT/kiosk entries; automation cannot distinguish a sanctioned sensor from an unmanaged one |
| old-wiki.corp.cvilab.local | Expires in 30d, weak key (1024-bit), untrusted issuer (self-signed), missing owner tag | Highest-stakes decision in the fleet: whether this legacy wiki is still a live service worth re-issuing under the Corp CA before it expires, or whether it should be decommissioned instead of remediated at all |

### Step 6 — Ownership Status on Manual-Review Items

All five manual-review certificates share the same gap: none have an owner tag. This means even after a human makes the trust decision, there is no clear person or team to route the resulting ticket to. The manual-review queue itself has no default assignee, which is an additional, compounding gap on top of the trust decision itself.

**Part B Summary:**

| Check | Result |
|---|---|
| Predictions recorded before running remediation | Yes |
| Automated Remediation run — counts recorded | Yes |
| Manual-review trust decisions documented per certificate | Yes |
| Ownership status checked for manual-review certificates | Yes |

---

## Lab Report Questions

**1. Compare the lowest-passing zone (Part A) to the highest-risk-score certificates from Lab 02. Do they point to the same certificates, or different ones? What does this tell you about the difference between "risk score" and "zone blast radius" as prioritization tools?**

The lowest-passing zone by severity, Unmanaged/Legacy, points to the exact same five certificates that scored highest in Lab 02's Discovery scan (`old-wiki.corp.cvilab.local`, `kiosk-app03`, `iot-gateway-07`, `old-fileserver.corp.cvilab.local`, `iot-sensor-14`). In this fleet the two views agree, but they are not measuring the same thing. Risk score ranks individual certificates by how dangerous each one is on its own. Zone pass/fail rate measures how an entire category of infrastructure is doing against the standard expected for that category. A zone could have a low pass rate built from many certificates each with a small, low-risk violation, which would not surface at all in a top-5 risk list. Zone view answers "which part of the organization is falling behind," while risk score answers "which individual asset is most dangerous right now." Both were needed to see the full picture, even though in this case they converged on the same certificates.

**2. For one of your manual-review certificates, explain specifically why automating its fix would require a "trust decision" a machine can't safely make, referencing the specific violation and what a human would actually have to weigh.**

`old-wiki.corp.cvilab.local` needs manual review because it is self-signed, weak-keyed, unowned, and expiring in 30 days all at once. A machine can renew a certificate through an existing trusted CA relationship, but it cannot decide whether a self-signed, unowned legacy wiki server is still a service worth keeping alive. Automating past this point would mean either blindly re-issuing trust for a system nobody has claimed, or blindly decommissioning something that might still be in use. That judgment, whether the underlying service is still legitimate and worth investing renewed trust in, requires a human who knows the business context, not just the certificate data.

**3. If your fleet had a certificate that was both self-signed AND missing an owner tag, walk through what happens to it in this lab: how would it score in Discovery, which zone would it land in, and how would Automation handle it?**

In Discovery, this certificate would score at least 70 (self-signed at +35, missing owner at +10, plus whatever expiry penalty applies), landing it in the Critical or High band before automation ever touches it. In the Policy tab, it would land in the Unmanaged/Legacy zone, since that zone's naming pattern is defined specifically by the self-signed-plus-unowned signature, and it would fail that zone's enforced rule outright, since the rule requires migration to the Corp CA with a modern key. In Automation, it would be routed straight to manual review rather than auto-fixed, since both of its defining violations, self-signed and missing owner, are the two conditions the CLM Simulator marks as flags that cannot be resolved through automated re-enrollment alone.

**4. If your organization's policy required 100% automation with no manual-review category at all, what specifically would have to change about how trust decisions get made, and why is that a bad idea?**

The automation would have to start making trust calls on its own, like re-issuing a self-signed certificate without anyone checking if the device behind it is even real. That's a bad idea because it means trusting things nobody actually verified. Manual review exists so a person confirms something is legitimate before it gets trusted. Skipping that just hides the risk instead of removing it.

**5. Now that you've seen both policy zones and automated remediation acting on the same fleet, explain how these two tabs work together, and what would be missing if a CLM platform only had one of them, not both.**

Policy identifies which certificates are out of compliance and why, grouped by the operational context of their zone, while Automation acts on those violations by fixing what is safe to fix and escalating what is not. Without Policy, Automation would have no organized view of why a certificate needs attention beyond a raw violation list, making it harder to see patterns like an entire zone failing systematically. Without Automation, Policy would just be a list of problems with nobody separating the easy fixes from the ones that need a person.

---

## Submission Checklist

- [x] All six policy zones reviewed and pass/fail rates documented
- [x] Lowest-passing zone identified with its failure pattern
- [x] Two violation types explained in own words, referencing specific certificates
- [x] Predictions recorded before running Automated Remediation
- [x] Automated Remediation run — auto-fixed and manual-review counts recorded
- [x] Predictions compared to actual results, mismatches explained
- [x] Specific trust decision documented for every manual-review certificate
- [x] Ownership status checked for all manual-review certificates
- [x] All five lab report questions answered in complete sentences
- [x] File committed to `labs/week-15/lab-03-clm-policy-automation.md`
