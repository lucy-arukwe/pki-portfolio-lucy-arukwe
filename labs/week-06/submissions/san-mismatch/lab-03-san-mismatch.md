# Lab [##] — [Lab Title]

**Week 6 · PKI Incident Diagnosis & Troubleshooting**
**CVI PKI Career Pathway — Phase 1 Foundations**

---

## Incident Summary

**Target system:** [system name and/or simulation target]
**Diagnosed by:** [your name]
**Date of diagnosis:** [date]

---

### What failed

[One sentence: what exactly caused the TLS failure]

---

### Evidence

- [Key field or value from the certificate — e.g., Not After date, Issuer CN, SAN entries]
- [Supporting command output or observation]
- [Any additional evidence]

---

### Why it failed

[2–3 sentences: the technical explanation of the failure. Connect it to what you learned in the
relevant Week 5 or Week 6 lesson. Don't just describe what happened — explain why it caused a
TLS error.]

---

### Chain status

[Was the certificate chain structurally intact? Were there any chain-related issues separate from
the primary failure?]

---

### Remediation path

[Step-by-step: what needs to happen to restore the failing system? Be specific. Walk through
the process rather than summarizing it in one line.]

---

### Prevention

[One concrete thing the organization could do differently to prevent this failure type from
recurring]

---

## Diagnostic Steps

Document each step of the PKI Diagnostic Framework as you worked through it.

### Step 1 — Retrieve

**Command used:**

```
[paste command here]
```

**What you observed:**

[What the output told you — connection errors, certificate retrieved, etc.]

---

### Step 2 — Parse

**Command used:**

```
[paste command here]
```

**Key fields from the certificate:**

| Field | Value |
|---|---|
| Subject CN | |
| Issuer | |
| Not Before | |
| Not After | |
| SAN entries | |

**What you found:**

[What the parsed certificate told you about the failure]

---

### Step 3 — Validate the Chain

**Command used:**

```
[paste command here]
```

**Result:**

[Chain valid / chain broken — and what the error said]

**What you found:**

[What this step confirmed or ruled out]

---

### Step 4 — Check Revocation and Trust

**Command used:**

```
[paste command here]
```

**What you found:**

[OCSP URL present or absent, revocation status if checked, any trust store issues]

---

## Reflection

[2–3 sentences: What did this lab reinforce or clarify for you? Was there a step where
you had to slow down and think carefully?]

---

*CVI PKI Career Pathway — Foundations Phase*
