# Lab 02: Issue Your First Certificate from a Custom Template

**Student Name:**  
**Date Completed:**  
**Phase:** 2 | **Week:** 10  
**Submission Path:** `labs/week-10/lab-02-first-issuance.md`

---

## Pre-Lab Verification

Run on PKI-SRV01 before starting.

```powershell
Get-Service -Name CertSvc
certutil -ping
```

**CertSvc status:** ________________  
**CA responding (certutil -ping):**
- [ ] Yes
- [ ] No — action taken:

**CVI-WebServer template visible in certtmpl.msc (from Lab 01):**
- [ ] Yes
- [ ] No — complete Lab 01 before proceeding

---

## Part A — Publish the Template to the CA

The CVI-WebServer template exists in Active Directory but is not yet published to CVI Issuing CA 1. Publishing it makes it available for certificate requests.

**Steps performed on PKI-SRV01:**

1. Opened **certsrv.msc**
2. Expanded **CVI Issuing CA 1** → right-clicked **Certificate Templates** → **New → Certificate Template to Issue**
3. Selected **CVI-WebServer** from the list
4. Clicked **OK**

**CVI-WebServer template now visible under Certificate Templates node:**
- [ ] Yes
- [ ] No — describe what happened:

```
(describe here)
```

**Screenshot or description of the Certificate Templates node showing CVI-WebServer:**

```
(describe what you see in certsrv.msc)
```

---

## Part B — Request the Certificate via MMC

**Steps performed on PKI-SRV01, logged in as CORP\pki.admin:**

1. Opened **mmc.exe** → **File → Add/Remove Snap-in**
2. Added **Certificates** snap-in
3. Selected: ________________ (Computer account / My user account / Service account)
4. Navigated to **Personal → Certificates**
5. Right-clicked → **All Tasks → Request New Certificate**
6. Proceeded through the Certificate Enrollment wizard

**Certificate Enrollment wizard — enrollment policy selected:**

```
(Active Directory Enrollment Policy or other?)
```

**Templates shown in the wizard:**

```
(list all templates visible)
```

**CVI-WebServer template visible:**
- [ ] Yes
- [ ] No — troubleshooting steps taken:

**Subject name entered (if prompted):**

```
(what subject name did you provide? or was it auto-populated?)
```

**Certificate request submitted:**
- [ ] Yes — certificate issued immediately
- [ ] Yes — certificate pending manager approval
- [ ] No — error encountered:

```
(paste error here if applicable)
```

---

## Part C — Inspect the Issued Certificate

### In the MMC Certificates Snap-in

Navigate to the Personal → Certificates store and double-click the issued certificate.

**General tab:**

| Field | Value |
|-------|-------|
| Issued to | |
| Issued by | |
| Valid from | |
| Valid to | |

**Details tab — record the following fields:**

| Field | Value |
|-------|-------|
| Serial Number | |
| Signature Algorithm | |
| Subject | |
| Key Usage | |
| Enhanced Key Usage | |
| Subject Alternative Name (if present) | |
| Thumbprint | |

---

### Via certutil

Export the certificate thumbprint from the Details tab, then run:

```powershell
certutil -store My "<thumbprint>"
```

Replace `<thumbprint>` with the thumbprint value (no spaces).

**Full certutil output:**

```
(paste output here)
```

---

### In certsrv.msc — Issued Certificates Node

Navigate to **certsrv.msc → CVI Issuing CA 1 → Issued Certificates**.

**Does the certificate appear in the Issued Certificates node?**
- [ ] Yes

**Record from the Issued Certificates node:**

| Column | Value |
|--------|-------|
| Request ID | |
| Requester Name | |
| Certificate Template | |
| Issued Common Name | |
| Certificate Expiration Date | |

---

## Part D — Write-Up: The Issuance Workflow

Describe the full certificate issuance workflow in your own words. Cover:

1. What happened in Active Directory when you published the template
2. What the MMC Certificate Enrollment wizard sent to the CA
3. What the CA evaluated before issuing the certificate
4. Where the issued certificate was placed and why

```
(your write-up here — aim for 1–2 paragraphs, plain language)
```

**One thing about the issuance process that you did not expect or want to understand better:**

```
(your observation here)
```

---

## Submission Checklist

- [ ] Pre-lab verification completed
- [ ] Part A: CVI-WebServer template published to CVI Issuing CA 1
- [ ] Part A: Template visible in certsrv.msc Certificate Templates node — confirmed
- [ ] Part B: Certificate requested via MMC — request submitted
- [ ] Part B: Enrollment wizard observations documented
- [ ] Part C: Certificate details recorded from MMC (General + Details tabs)
- [ ] Part C: certutil -store My output pasted
- [ ] Part C: Certificate confirmed in certsrv.msc Issued Certificates node
- [ ] Part D: Issuance workflow write-up completed in own words
- [ ] File saved as `lab-02-first-issuance.md`
- [ ] File committed to portfolio repo under `labs/week-10/`
