# Lab 04 — Full Diagnostic Scenario: Incident Report

**Week 6 · PKI Incident Diagnosis & Troubleshooting**
**CVI PKI Career Pathway — Phase 1 Foundations**

---

## PKI Incident Report

**System:** [system name]
**Reported:** [date and time of incident]
**Author:** [your name]
**Status:** [Diagnosis complete — pending remediation / Resolved]

---

### Executive Summary

[2–3 sentences: what failed, who was affected, and what needs to happen to fix it.
Write this so a non-technical manager can understand it without PKI background.]

---

### Technical Findings

#### Finding 1 — [Descriptive title, e.g., "Missing Root CA in Clinical Subnet Trust Stores"]

**Type:** [Certificate / Chain / Trust Store / Revocation / Configuration]
**Severity:** [Critical / High / Medium / Low]

**Detail:**
[What you found and why it matters technically]

**Evidence:**
[Commands run or scenario information that confirms this finding]

---

#### Finding 2 — [Descriptive title]

**Type:** [Certificate / Chain / Trust Store / Revocation / Configuration]
**Severity:** [Critical / High / Medium / Low]

**Detail:**
[What you found and why it matters technically]

**Evidence:**
[Commands run or scenario information that confirms this finding]

---

#### Finding 3 — [Descriptive title, if applicable]

**Type:** [Certificate / Chain / Trust Store / Revocation / Configuration]
**Severity:** [Critical / High / Medium / Low]

**Detail:**
[What you found and why it matters technically]

**Evidence:**
[Commands run or scenario information that confirms this finding]

---

### Diagnostic Steps

Document how you worked through the 4-step framework for this scenario.

#### Step 1 — Retrieve

[What you would do to retrieve the certificate from the failing system.
What command would you use? What output would you expect?]

---

#### Step 2 — Parse

[What fields you would check and what the scenario tells you about each.
What does the certificate confirm or rule out?]

---

#### Step 3 — Validate the Chain

[What chain validation would show. What is the likely result and why?
What does this step confirm about where the failure is located?]

---

#### Step 4 — Check Revocation and Trust

[What revocation check would show. Is there a revocation obligation from the scenario?
What does this step confirm or surface as a secondary concern?]

---

### Failures in Diagnostic Order

List each failure or contributing factor in the order a PKI engineer should address them:

1. **[Primary failure]**
   - Type: [Certificate / Chain / Trust Store / Revocation / Configuration]
   - Evidence: [what in the scenario supports this]

2. **[Contributing factor or secondary issue]**
   - Type:
   - Evidence:

3. **[Additional issue, if applicable]**
   - Type:
   - Evidence:

---

### Root Cause

[One paragraph: what is the underlying reason the incident occurred? Go beyond the technical
failure — explain the process or operational gap that allowed it to happen.]

---

### Remediation Steps

List each action in the order it should be executed:

1. [Immediate — what restores access right now]
2. [Short-term — what fully resolves the incident within 24–48 hours]
3. [Secondary — cleanup, revocation, or follow-up actions]

---

### Prevention Recommendations

[2–3 concrete recommendations to prevent recurrence. Think about: certificate deployment
verification, Group Policy or MDM scope, post-renewal validation checklists, monitoring.]

---

### Lessons Learned

[One paragraph written as if debriefing your team. What did this incident reveal about the
organization's PKI operations? What should change going forward?]

---

## Reflection

[2–3 sentences: Which part of the multi-failure investigation was hardest to reason through?
Was there a point where you wanted to skip a framework step? What would you do differently
in a real production incident?]

---

*CVI PKI Career Pathway — Foundations Phase*
