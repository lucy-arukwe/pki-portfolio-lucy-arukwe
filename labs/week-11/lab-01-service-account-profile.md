# Lab 01: Build a Certificate Profile for a Service Account

**Student Name:**  
**Date Completed:**  
**Phase:** 2 | **Week:** 11  
**Submission Path:** `labs/week-11/lab-01-service-account-profile.md`

---

## Pre-Lab Verification

Run the following on PKI-SRV01 before starting. Do not proceed until all checks pass.

```powershell
# Check 1 — CA service running
Get-Service -Name CertSvc

# Check 2 — CA responding
certutil -ping

# Check 3 — Confirm svc.autoenroll account exists in AD
Get-ADUser -Identity svc.autoenroll -Properties UserPrincipalName | Select-Object Name, UserPrincipalName
```

**All checks passed:**
- [ ] Yes
- [ ] No — describe the issue and how you resolved it:

```
(describe here)
```

---

## Part A — Design the CVI-ServiceAccount Template

Open the Certificate Templates console: **Run → certtmpl.msc**

### Template Duplication

**Source template duplicated:** ________________ (recommended: User or Computer — document your choice)

**Reason for choosing this source template:**

```
(your explanation here)
```

**Compatibility settings selected:**
- Certification Authority: ________________
- Certificate Recipient: ________________

### Design Decisions

Document every setting with a reason. This is the design record for the template.

**1 — Key Usage**

| Key Usage | Included? | Reason |
|-----------|-----------|--------|
| Digital Signature | | |
| Key Encipherment | | |
| Data Encipherment | | |
| Non-Repudiation | | |

**Explanation of Key Usage decisions:**

```
(in your own words — why did you include what you included, and exclude what you excluded?)
```

**2 — Extended Key Usage (EKU)**

| EKU | Included? | OID | Reason |
|-----|-----------|-----|--------|
| Client Authentication | | 1.3.6.1.5.5.7.3.2 | |
| Server Authentication | | 1.3.6.1.5.5.7.3.1 | |
| Code Signing | | 1.3.6.1.5.5.7.3.3 | |
| Secure Email | | 1.3.6.1.5.5.7.3.4 | |

**Explanation of EKU decisions:**

```
(in your own words — what does this certificate need to do, and why does that determine the EKU?)
```

**3 — Subject Name**

| Setting | Value Selected | Reason |
|---------|---------------|--------|
| Subject name format | Build from AD / Supply in request | |
| Include this information in the subject name | | |

**Explanation of Subject Name decision:**

```
(why does this certificate use "Build from AD" rather than "Supply in request"?)
```

**4 — Validity Period**

| Setting | Value | Reason |
|---------|-------|--------|
| Validity period | | |
| Renewal period | | |

**Explanation of validity period decision:**

```
(why did you choose this period? how does it relate to the CA maximum and the service account lifecycle?)
```

**5 — Security Tab — Enrollment Permissions**

| Group / Account | Read | Enroll | Autoenroll | Reason |
|-----------------|------|--------|------------|--------|
| Authenticated Users | | | | |
| CORP\svc.autoenroll | | | | |
| Domain Computers | | | | |

**Explanation of enrollment permission decisions:**

```
(why are permissions set this way? what risk does restricting Enroll to svc.autoenroll only prevent?)
```

**General tab — Template names:**

| Field | Value |
|-------|-------|
| Template display name | CVI Service Account |
| Template name (internal) | CVI-ServiceAccount |
| Schema version | |

**Template saved and visible in certtmpl.msc:**
- [ ] Yes

---

## Part B — Publish the Template and Issue the Certificate

### Publish the Template

**Steps performed on PKI-SRV01 as CORP\pki.admin:**

1. Opened **certsrv.msc**
2. Expanded **CVI Issuing CA 1** → right-clicked **Certificate Templates** → **New → Certificate Template to Issue**
3. Selected **CVI-ServiceAccount** from the list
4. Clicked **OK**

**CVI-ServiceAccount visible in Certificate Templates node:**
- [ ] Yes
- [ ] No — describe what happened:

```
(describe here)
```

---

### Request the Certificate for svc.autoenroll

**Steps performed:**

1. Opened **mmc.exe** → **File → Add/Remove Snap-in**
2. Added **Certificates** snap-in
3. Selected: **Service account** → **Local computer**
4. Entered service account: ________________
5. Navigated to **Personal → Certificates**
6. Right-clicked → **All Tasks → Request New Certificate**

**Enrollment wizard — enrollment policy selected:**

```
(Active Directory Enrollment Policy or other?)
```

**Templates visible in the wizard:**

```
(list all templates visible)
```

**CVI-ServiceAccount visible in wizard:**
- [ ] Yes
- [ ] No — troubleshooting steps taken:

**Certificate request submitted:**
- [ ] Yes — issued immediately
- [ ] Yes — pending manager approval (describe resolution):
- [ ] No — error encountered:

```
(paste error or describe outcome)
```

---

## Part C — Verify the Issued Certificate

### Via certutil

Run the following to inspect the service account certificate store:

```powershell
certutil -store -service svc.autoenroll My
```

Or, if the certificate was issued to the personal store under the current user context:

```powershell
certutil -store My
```

**Full certutil output:**

```
(paste output here)
```

**From the certutil output — record the following:**

| Field | Value |
|-------|-------|
| Subject | |
| Issuer | |
| Serial Number | |
| Key Usage | |
| Enhanced Key Usage (EKU) | |
| Validity: Not Before | |
| Validity: Not After | |
| Thumbprint | |

---

### In certsrv.msc — Issued Certificates Node

Navigate to **certsrv.msc → CVI Issuing CA 1 → Issued Certificates**.

**Certificate visible in Issued Certificates node:**
- [ ] Yes

**Record from the Issued Certificates node:**

| Column | Value |
|--------|-------|
| Request ID | |
| Requester Name | |
| Certificate Template | |
| Issued Common Name | |
| Certificate Expiration Date | |

> **Save this Request ID.** You will use it in Week 12 if you revoke this certificate, and in Lab 03 for the comparison exercise.

---

## Part D — Written Explanation

**What makes a service account certificate different from a user certificate? Address the following:**

1. Key difference in EKU — what does a user certificate include that the service account certificate should not, and why?
2. Subject Name source — both use "Build from AD," but what is different about the identity being represented?
3. Enrollment — who requests a user certificate vs. who requests a service account certificate, and why does this matter?

```
(your written explanation here — aim for 2–3 paragraphs, plain prose)
```

**What are the operational risks of relying on password authentication for service accounts instead of certificate-based authentication?**

```
(your answer here — identify at least two specific risks)
```

---

## Reflection

**One thing about the CVI-ServiceAccount template design that was a non-obvious decision:**

```
(your observation here)
```

**What would you change about this template if this were a production environment rather than a lab?**

```
(your answer here — think about approval workflow, validity period, or monitoring)
```

---

## Submission Checklist

- [ ] Pre-lab verification completed and outputs recorded
- [ ] Part A: All five template design decisions documented with rationale
- [ ] Part A: Template created as CVI-ServiceAccount and visible in certtmpl.msc
- [ ] Part B: Template published to CVI Issuing CA 1
- [ ] Part B: Certificate issued to svc.autoenroll — enrollment steps documented
- [ ] Part C: certutil output pasted and key fields extracted into table
- [ ] Part C: Request ID recorded from certsrv.msc Issued Certificates node
- [ ] Part D: Written explanation completed in prose
- [ ] Reflection section completed
- [ ] File saved as `lab-01-service-account-profile.md`
- [ ] File committed to portfolio repo under `labs/week-11/`
