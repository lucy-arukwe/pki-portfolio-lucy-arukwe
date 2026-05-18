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
The option "Do not automatically reenroll if a duplicate certificate exists in Active Directory" is enabled to prevent
unnecessary certificate issuance. 
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
The Web Server template uses "Supplied in the request" because web servers often require
specific DNS names such as public website addresses or application names.
These names may not exactly match the Active Directory computer object.
Allowing the requester to define the subject name provides flexibility for TLS certificates
used by websites, load balancers, reverse proxies, and other web-based services.
```

---

## Part B — Duplicate the Web Server Template

**Steps performed:**

1. Right-clicked the **Web Server** template → **Duplicate Template**
2. Selected compatibility settings:

   - Certification Authority: Windows Server 2012 R2
   - Certificate Recipient: Windows 7 / Server 2008 R2

3. Opened the properties of the new duplicate template.

**General tab — changes made:**

| Setting               | Original Value | New Value     |
|-----------------------|----------------|---------------|
| Template display name | Web Server     | CVI-WebServer |
| Template name         | WebServer      | CVI-WebServer |
| Validity period       | 2 years        | 1 year        |
| Renewal period        | 6 weeks        | 6 weeks       |

**Rationale for validity period chosen:**

```
A one-year validity period was chosen because it gives the certificate a shorter trusted lifetime while still being practical for administration. If a certificate or its private key is ever compromised, a shorter validity period reduces how long that certificate can remain trusted before it naturally expires. It also encourages regular renewal, which helps administrators review certificate usage, confirm ownership, and replace certificates before they become outdated.

This also follows the PKI validity hierarchy. The end-entity certificate should not outlive the issuing CA, and the template validity period should stay within the limits enforced by the CA. Since the CVI Issuing CA has a longer validity period, a one-year web server certificate is appropriate for this lab. It supports better security while still giving enough time for normal certificate management and renewal.
```

**Subject Name tab — changes made:**

| Setting     | Change Made | Rationale |
|-------------|-------------|-----------|
|Subject Name | Supply in the request confirmed | Allows administrators to specify DNS names and server identities manually during enrollment  |

**Security tab — permissions confirmed:**

| Group / Account     | Enroll    | Autoenroll |
|---------------------|-----------|------------|
| Domain Computers    | Yes       | No         |
| Authenticated Users | Read only | No         |

**Template saved:**
- [x] Yes — template visible in certtmpl.msc

---

## Part C — Inspect the Duplicate Template with certutil

Run the following command on PKI-SRV01:

```powershell
certutil -template CVI-WebServer
```

**Full output:**

```
 *certutil -template`* command had limited output.
The detailed template information was retrieved using:

certutil -v -template CVI-WebServer

This displayed the template properties, EKU settings, key usage, subject name flags, validity period, permissions, and template OID.
```

**From the certutil output — record the following:**

| Field                              | Value from certutil Output                                                       |
|------------------------------------|----------------------------------------------------------------------------------|
| Template Name                      | CVI-WebServer                                                                    |
| Template OID                       |1.3.6.1.4.1.311.21.8.15886664.4298044.8996776.14853544.7902291.169.6737681.317150 |
| Schema Version                     | 2                                                                                |
| Key Usage                          | Digital Signature, Key Encipherment                                              |
| Enhanced Key Usage (EKU)           | Server Authentication                                                            |
| Validity Period                    | 1 years                                                                          |
| Subject Name flags                 | CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT                                                |
```
The certutil output confirmed that the CVI-WebServer template was configured correctly for TLS web server certificates. The EKU was set to Server Authentication, ensuring the certificate is trusted specifically for TLS web server authentication purposes. The key usage allowed Digital Signature and Key Encipherment, and the subject name flag confirmed that the subject must be supplied in the request rather than automatically pulled from Active Directory.
```
---

## Reflection

**Why does AD CS require you to duplicate a built-in template rather than modifying it directly?**

```
AD CS prevents administrators from directly modifying built-in templates to protect the
stability and consistency of the PKI environment. Built-in templates are considered default baseline configurations that
Microsoft expects to remain unchanged. Duplicating a template creates a customizable copy while preserving
the original template for compatibility and recovery purposes.
```

**One setting in the template you found unexpected or would want to explore further:**

```
One setting I found interesting was how the compatibility settings affect the available template options. When the template
was set to Windows Server 2012 R2 and Windows 7 / Server 2008 R2 compatibility, several settings became grayed out or unavailable.
It showed how certificate templates need to balance newer security features with support for older systems.
```

---

## Submission Checklist

- [x] Pre-lab verification completed and outputs recorded
- [x] Part A: All three templates explored with tab-level observations
- [x] Part A: Comparison question answered in own words
- [x] Part B: Duplicate template created as CVI-WebServer
- [x] Part B: All changes documented with rationale
- [ ] Part C: certutil -template output pasted and key fields extracted
- [x] Reflection section completed
- [x] File saved as `lab-01-template-exploration.md`
- [x] File committed to portfolio repo under `labs/week-10/`
