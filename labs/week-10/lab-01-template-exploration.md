# Lab 01: Explore and Duplicate a Certificate Template

**Student Name:**  Lucy Arukwe
**Date Completed:**  13 May, 2026
**Phase:** 2 | **Week:** 10  
**Submission Path:** `labs/week-10/lab-01-template-exploration.md`

---

## Pre-Lab Verification

Run the following on PKI-SRV01 before starting. Do not proceed until all checks pass.

```powershell
# Check 1 — CA service running
Get-Service -Name CertSvc

# Check 2 — CA responding
certutil -ping

# Check 3 — Issuing CA cert in enterprise store
certutil -store -enterprise CA
```

**All checks passed:**
- [x] Yes
- [ ] No — describe the issue and how you resolved it:

```
(describe here)
```

---

## Part A — Explore Three Built-in Templates

Open the Certificate Templates console: **Run → certtmpl.msc**

### Template 1: User

| Field                    | Value        |
|--------------------------|--------------|
| Template Display Name    | User         |
| Template Name (internal) | User         |
| Minimum Supported CA     | Windows 2000 |
| Validity Period          | 1 years      |
| Renewal Period           | 6 Weeks      |

**General tab — notes:**

```
The User template is designed for end users within Active Directory. 
The certificate can be used for user authentication, secure email, and Encrypting File System (EFS). 
The template is integrated with Active Directory and automatically builds identity information from the user account.
The option "Do not automatically reenroll if a duplicate certificate exists in Active Directory" is enabled to prevent unnecessary certificate issuance. 
```

**Request Handling tab — Purpose:**

```
- Signature and Encryption
- The template supports multiple cryptographic service providers including Microsoft Base Cryptographic Provider v1.0 and
allows the private key to be exported.
```

**Subject Name tab — Subject name format:**

```
- Built from information in Active Directory
- The template is configured to pull the subject name from the user's AD account,
  with the "Include e-mail name in subject name" option enabled. Type of subject is set to "User."
```

**Extensions tab — Key Usage:**

```
- Digital signature
- Allow key exchange only with key encryption
- Critical extension
```

**Extensions tab — Application Policies (EKU):**

```
- Encrypting File System
- Secure Email
- Client Authentication
```

---

### Template 2: Computer

| Field                    | Value    |
|--------------------------|----------|
| Template Display Name    | Computer |
| Template Name (internal) | Machine  |
| Validity Period          | 1 years  |
| Renewal Period           | 6 weeks  |

**Request Handling tab — Purpose:**

```
- Signature and Encryption
- The template uses the Microsoft RSA SChannel Cryptographic Provider
  and does not allow the private key to be exported.
```

**Subject Name tab:**

```
- Built from information in Active Directory
- The certificate subject is populated using the computer account in AD
- Type of subject is set to "Computer or other device."
```

**Extensions tab — Key Usage:**

```
- Digital Signature
- Allow key exchange only with key encryption
- Critical extension
```

**Extensions tab — Application Policies (EKU):**

```
- Client Authentication
- Server Authentication
```

---

### Template 3: Web Server

| Field                    | Value     |
|--------------------------|-----------|
| Template Display Name    | WebServer |
| Template Name (internal) | WebServer |
| Validity Period          | 2 years   |
| Renewal Period           | 6 weeks   |

**Request Handling tab — Purpose:**

```
Signature and Encryption
```

**Subject Name tab:**

```
- Supplied in the request
- Requires the certificate requester to provide the subject name in the certificate request
```

**Extensions tab — Key Usage:**

```
- Digital Signature
- Allow key exchange only with key encryption
- Critical extension
```

**Extensions tab — Application Policies (EKU):**

```
Server Authentication
```

---

### Template Comparison

In your own words — what is the most significant difference between the User, Computer, and Web Server templates?

```
The User, Computer, and Web Server templates are designed for different identities and trust purposes within the PKI environment. 
The User template is intended for individual users and automatically pulls identity information from Active Directory. 
The Computer template is designed for domain-joined systems and also builds the subject information from Active Directory. 
The Web Server template is different because it allows the subject name to be supplied in the request, which is necessary for 
web servers that may host different DNS names or external services not directly tied to an AD object.
```

Why does the Web Server template use "Supplied in the request" for the subject name rather than building it from Active Directory?

```
The Web Server template uses "Supplied in the request" because web servers often require specific DNS names such as public website addresses or application names.
These names may not exactly match the Active Directory computer object. Allowing the requester to define the subject name provides flexibility for TLS certificates
used by websites, load balancers, reverse proxies, and other web-based services.
```

---

## Part B — Duplicate the Web Server Template

**Steps performed:**

1. Right-clicked the **Web Server** template → **Duplicate Template**
2. Selected compatibility settings:

   - Certification Authority: ________________
   - Certificate Recipient: ________________

3. Opened the properties of the new duplicate template.

**General tab — changes made:**

| Setting | Original Value | New Value |
|---------|---------------|-----------|
| Template display name | Web Server | CVI-WebServer |
| Template name | WebServer | CVI-WebServer |
| Validity period | | |
| Renewal period | | |

**Rationale for validity period chosen:**

```
(why did you choose this period?)
```

**Subject Name tab — changes made:**

| Setting | Change Made | Rationale |
|---------|-------------|-----------|
| | | |

**Security tab — permissions confirmed:**

| Group / Account | Enroll | Autoenroll |
|-----------------|--------|------------|
| Domain Computers | | |
| Authenticated Users | | |

**Template saved:**
- [ ] Yes — template visible in certtmpl.msc

---

## Part C — Inspect the Duplicate Template with certutil

Run the following command on PKI-SRV01:

```powershell
certutil -template CVI-WebServer
```

**Full output:**

```
(paste output here)
```

**From the certutil output — record the following:**

| Field | Value from certutil Output |
|-------|---------------------------|
| Template Name | |
| Template OID | |
| Schema Version | |
| Key Usage | |
| Enhanced Key Usage (EKU) | |
| Validity Period | |
| Subject Name flags | |

---

## Reflection

**Why does AD CS require you to duplicate a built-in template rather than modifying it directly?**

```
(your answer here)
```

**One setting in the template you found unexpected or would want to explore further:**

```
(your observation here)
```

---

## Submission Checklist

- [ ] Pre-lab verification completed and outputs recorded
- [ ] Part A: All three templates explored with tab-level observations
- [ ] Part A: Comparison question answered in own words
- [ ] Part B: Duplicate template created as CVI-WebServer
- [ ] Part B: All changes documented with rationale
- [ ] Part C: certutil -template output pasted and key fields extracted
- [ ] Reflection section completed
- [ ] File saved as `lab-01-template-exploration.md`
- [ ] File committed to portfolio repo under `labs/week-10/`
