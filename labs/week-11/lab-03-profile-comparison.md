# Lab 03: Compare Certificate Profiles Side by Side

**Student Name:**  
**Date Completed:**  
**Phase:** 2 | **Week:** 11  
**Submission Path:** `labs/week-11/lab-03-profile-comparison.md`

---

## Overview

Lab 03 is a synthesis exercise. You have issued three certificates across Weeks 10 and 11:

1. **TLS certificate** — issued from CVI-WebServer in Week 10, Lab 02
2. **Service account certificate** — issued from CVI-ServiceAccount in Week 11, Lab 01
3. **Code signing certificate** — issued from CVI-CodeSigning in Week 11, Lab 02

In this lab, you inspect all three certificates using certutil and document their settings in a structured comparison table. Then you write an analytical explanation of why the differences exist.

> **Prerequisite:** All three certificates must be present in their respective stores on PKI-SRV01. If the Week 10 TLS certificate is no longer available, contact the instructor before proceeding.

---

## Pre-Lab — Locate All Three Certificates

**Step 1 — Confirm the Week 10 TLS certificate is in the store**

```powershell
# List all certificates in the personal store
certutil -store My
```

**Week 10 TLS certificate present:**
- [ ] Yes — Thumbprint: ________________
- [ ] No — contact instructor before proceeding

**Step 2 — Record all three thumbprints**

| Certificate | Template | Thumbprint |
|-------------|----------|------------|
| TLS (Week 10, Lab 02) | CVI-WebServer | |
| Service Account (Lab 01) | CVI-ServiceAccount | |
| Code Signing (Lab 02) | CVI-CodeSigning | |

---

## Part A — Inspect All Three Certificates

Run `certutil -store My "<thumbprint>"` for each certificate. Paste the full output for each.

### Certificate 1 — TLS (CVI-WebServer)

```powershell
certutil -store My "<TLS-thumbprint>"
```

**Full certutil output:**

```
(paste output here)
```

---

### Certificate 2 — Service Account (CVI-ServiceAccount)

```powershell
certutil -store My "<ServiceAccount-thumbprint>"
```

Or, if stored in the service account store:

```powershell
certutil -store -service svc.autoenroll My "<ServiceAccount-thumbprint>"
```

**Full certutil output:**

```
(paste output here)
```

---

### Certificate 3 — Code Signing (CVI-CodeSigning)

```powershell
certutil -store My "<CodeSigning-thumbprint>"
```

**Full certutil output:**

```
(paste output here)
```

---

### Comparison Table

Complete the following table using the certutil outputs above.

| Field | TLS Certificate | Service Account Certificate | Code Signing Certificate |
|-------|----------------|----------------------------|--------------------------|
| Template Name | CVI-WebServer | CVI-ServiceAccount | CVI-CodeSigning |
| Subject | | | |
| Subject Source | Supplied in request | Build from AD | Build from AD |
| Issuer | | | |
| Key Usage | | | |
| EKU | | | |
| EKU OID(s) | | | |
| Validity Period | | | |
| Serial Number | | | |
| Thumbprint | | | |
| Request ID | | | |

---

## Part B — Written Analysis

This section is the substance of Lab 03. Write in prose paragraphs — not bullet points.

**1 — Key Usage**

Each of the three certificates has different Key Usage settings. Explain *why* each certificate has the Key Usage it has. For each, trace the setting back to the cryptographic operation the use case requires.

```
(your written analysis — 2–3 paragraphs)
```

---

**2 — Extended Key Usage (EKU)**

Each certificate has a different EKU. Explain why each certificate has the EKU it has. For each, identify the relying party application that enforces that EKU and what it would do if the EKU were absent or wrong.

```
(your written analysis — 2–3 paragraphs)
```

---

**3 — Subject Name Source**

The TLS certificate uses "Supplied in request" for its subject. The service account and code signing certificates use "Build from Active Directory." Explain why the subject name source is different for the TLS certificate — what is it about the TLS use case that makes AD-supplied subject names impractical?

```
(your written analysis — 1–2 paragraphs)
```

---

**4 — Security Question**

Imagine a single certificate that combined all three EKUs: Server Authentication, Client Authentication, and Code Signing. What security risk would this create? Be specific — what could an attacker do with a compromised private key from this combined certificate that they could not do with any one of the three individual certificates?

```
(your written analysis — 1–2 paragraphs)
```

---

## Reflection

**Which of the three certificates would you consider most critical to revoke quickly if the private key were compromised — and why?**

```
(your answer here — consider the blast radius of each)
```

**What would you add to this comparison if you were presenting it to a security team evaluating the PKI environment?**

```
(your answer here)
```

---

## Submission Checklist

- [ ] Pre-lab: All three certificate thumbprints recorded
- [ ] Part A: certutil output for all three certificates pasted in code blocks
- [ ] Part A: Comparison table fully completed — no blank cells
- [ ] Part B: Key Usage analysis written in prose — explains *why*, not just *what*
- [ ] Part B: EKU analysis written in prose — identifies the relying party for each
- [ ] Part B: Subject Name source difference explained with reasoning
- [ ] Part B: Security question answered — specific risk identified for combined-EKU certificate
- [ ] Reflection completed
- [ ] File saved as `lab-03-profile-comparison.md`
- [ ] File committed to portfolio repo under `labs/week-11/`
- [ ] All three Week 11 labs committed with a single meaningful commit message
- [ ] Request IDs for all three certificates recorded (TLS from Week 10, Service Account, Code Signing) — needed for Week 12
