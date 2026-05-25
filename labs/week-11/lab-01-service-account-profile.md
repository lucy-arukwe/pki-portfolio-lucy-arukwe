# Lab 01: Build a Certificate Profile for a Service Account

**Student Name:**  Lucy Arukwe
**Date Completed:**  May 24, 2026
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
- [x] Yes
- [ ] No — describe the issue and how you resolved it:

```
Initially, the Get-ADUser command returned a CommandNotFoundException because the Active Directory PowerShell module was unavailable in the session. The issue was resolved by installing/loading the RSAT Active Directory module, which allowed the AD cmdlets to function correctly.
```

---

## Part A — Design the CVI-ServiceAccount Template

Open the Certificate Templates console: **Run → certtmpl.msc**

### Template Duplication

**Source template duplicated:** User (recommended: User or Computer — document your choice)

**Reason for choosing this source template:**

```
The User template was selected because svc.autoenroll exists in Active Directory as a user principal rather than a computer object. Although the account functions as a non-human service identity operationally, the certificate still represents a user-based AD identity with a UPN and user security context. Using the User template provided a more appropriate starting point for Client Authentication and user-context enrollment workflows than the Computer template, which is primarily designed for machine identities, DNS-based authentication, and certificates issued to the Local Computer store.

During the lab, this became clearer while troubleshooting the MMC enrollment process. The “Service account” MMC workflow from the resource pack did not display the svc.autoenroll identity because it was not a true Windows managed service account or machine identity. The final successful enrollment was completed by launching MMC directly under the svc.autoenroll context using runas, then enrolling through the Current User certificate store.
```

**Compatibility settings selected:**
- Certification Authority: Windows Server 2003
- Certificate Recipient: Windows XP / Server 2003

### Design Decisions

Document every setting with a reason. This is the design record for the template.

**1 — Key Usage**

| Key Usage          | Included? | Reason                                                                  |
|--------------------|-----------|-------------------------------------------------------------------------|
| Digital Signature  | Yes       | Required for authentication signing                                     |
| Key Encipherment   | Yes       |Supports secure session key exchange during TLS authentication workflows |
| Data Encipherment  | No        | Not required for this authentication workflow                           |         
| Non-Repudiation    | No        | Not intended for legal signing or document integrity workflows          |                                       

**Explanation of Key Usage decisions:**

```
Digital Signature supports authentication signing operations, while Key Encipherment supports secure session key exchange during TLS authentication workflows. Data Encipherment and Non-Repudiation were removed because the certificate was not intended for file encryption, email protection, or legal signing purposes.
```

**2 — Extended Key Usage (EKU)**

| EKU                   | Included? | OID               | Reason                                         |
|-----------------------|-----------|-------------------|------------------------------------------------|
| Client Authentication | Yes       | 1.3.6.1.5.5.7.3.2 | Required for authenticating to services        |
| Server Authentication | No        | 1.3.6.1.5.5.7.3.1 | This certificate is not hosting TLS services   |
| Code Signing          | No        | 1.3.6.1.5.5.7.3.3 | Not intended for signing executables or scripts|
| Secure Email          | No        | 1.3.6.1.5.5.7.3.4 | Not intended for email encryption or signing   |

**Explanation of EKU decisions:**

```
Only Client Authentication was included because the service account certificate was intended to authenticate to services and systems. Other EKUs such as Secure Email, Code Signing, and Server Authentication were removed to follow the principle of least privilege and reduce unnecessary certificate capabilities and limit attack surface
```

**3 — Subject Name**

| Setting                                      | Value Selected            | Reason                                      |
|----------------------------------------------|---------------------------|---------------------------------------------|
| Subject name format                          | Build from AD             | Ensures the identity comes directly from AD |
| Include this information in the subject name | User Principal Name (UPN) | Provides a unique domain identity           |

**Explanation of Subject Name decision:**

```
The template used “Build from Active Directory” because the service account already existed as a managed domain identity. Allowing users to manually supply the subject name could allow unauthorized subject impersonation or incorrect identity assignment. Using the AD identity ensured consistency and centralized identity control.
```

**4 — Validity Period**

| Setting         | Value  | Reason                                                  |
|-----------------|--------|---------------------------------------------------------|
| Validity period | 1 Year | Limits long-term exposure and supports renewal workflows|
| Renewal period  | 6 Weeks| Provides enough time for renewal before expiration      |

**Explanation of validity period decision:**

```
A 1-year validity period was selected to balance operational stability with security. Shorter validity periods reduce long-term exposure if a certificate or key is compromised. The 6-week renewal period provides enough time for renewal processes and supports future autoenrollment workflows.
```

**5 — Security Tab — Enrollment Permissions**

| Group / Account     | Read | Enroll | Autoenroll | Reason                                      |
|---------------------|------|--------|------------|---------------------------------------------|
| Authenticated Users | Yes  | No     | No         | Allows visibility without enrollment rights |
| CORP\svc.autoenroll | Yes  | Yes    | No         | Intended service identity                   |
| Domain Computers    | No   | No     | No         | Not required for this template              |

**Explanation of enrollment permission decisions:**

```
Enrollment permissions were restricted to only the svc.autoenroll account to ensure that only the intended non-human identity could request the certificate. Restricting enrollment rights reduces the risk of unauthorized certificate issuance, subject impersonation, and misuse of the service account identity. Authenticated Users were granted Read permission only so the template remained visible in the enrollment process without allowing unrestricted certificate requests. This follows the principle of least privilege and helps reduce the blast radius if another domain account becomes compromised
```

**General tab — Template names:**

| Field                    | Value               |
|--------------------------|-------=-------------|
| Template display name    | CVI Service Account |
| Template name (internal) | CVI-ServiceAccount  |
| Schema version           | 2                   |

**Template saved and visible in certtmpl.msc:**
- [x] Yes

---

## Part B — Publish the Template and Issue the Certificate

### Publish the Template

**Steps performed on PKI-SRV01 as CORP\pki.admin:**

1. Opened **certsrv.msc**
2. Expanded **CVI Issuing CA 1** → right-clicked **Certificate Templates** → **New → Certificate Template to Issue**
3. Selected **CVI-ServiceAccount** from the list
4. Clicked **OK**

**CVI-ServiceAccount visible in Certificate Templates node:**
- [x] Yes
- [ ] No — describe what happened:

```
(describe here)
```

---

### Request the Certificate for svc.autoenroll

**Initial Attempt and Troubleshooting**

The first attempt followed the lab instructions exactly by selecting:

Certificates → Service account → Local computer

However, the MMC console only displayed Windows service identities and did not display the `svc.autoenroll` Active Directory account. This occurred because `svc.autoenroll` was a normal AD user object rather than a managed Windows service identity.

A second attempt was performed using the Current User certificate store while logged in as `pki.admin`, but the certificate issued to `PKI Admin` instead of `svc.autoenroll`.

The issue was resolved by launching MMC directly under the `svc.autoenroll` security context using:

`runas /profile /user:CORP\svc.autoenroll "mmc.exe"`

This ensured that certificate enrollment occurred under the correct user identity.

Final Successful Enrollment Steps
Opened `mmc.exe` using the `svc.autoenroll` account
Added the `certificates` snap-in
Selected `My user account`
Navigated to `Personal → Certificates`
Right-clicked `Personal → All Tasks → Request New Certificate`


**Steps performed:**

1. Opened **mmc.exe** → **File → Add/Remove Snap-in**
2. Added **Certificates** snap-in
3. Selected: **Service account** → **Local computer**
4. Entered service account: CVI-Service Account
5. Navigated to **Personal → Certificates**
6. Right-clicked → **All Tasks → Request New Certificate**

**Enrollment wizard — enrollment policy selected:**

```
Active Directory Enrollment Policy
```

**Templates visible in the wizard:**

```
Administrator
Basic EFS
CVI Service Account
EFS Recovery Agent
User
```

**CVI-ServiceAccount visible in wizard:**
- [x] Yes
- [ ] No — troubleshooting steps taken:

**Certificate request submitted:**
- [x] Yes — issued immediately
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
C:\Windows\system32>whoami
corp\svc.autoenroll

C:\Windows\system32>certutil -user -store My

My "Personal"

================ Certificate 0 ================
Serial Number: 4400000009217b3df4da50401000000000009
Issuer: CN=CVI Issuing CA 1, DC=corp, DC=cvilab, DC=local
NotBefore: 5/24/2026 9:25 AM
NotAfter: 4/25/2027 7:36 PM
Subject: CN=Svc Autoenroll
Non-root Certificate
Template: CV-ServiceAccount, CVI Service Account
Cert Hash(sha1): 762c31ae2ec7b9430ad86b6a616199e4369a74c7

  Key Container = 4379e4213699fcaf1408a0d217893ad_f0a99c17-76d3-498a-97de-2992c06105fd
  Simple container name: te-CVI-ServiceAccount-d363b350-8fbc-4e15-bc46-1bc11727e5bf
  Provider = Microsoft Enhanced Cryptographic Provider v1.0

Encryption test passed
CertUtil: -store command completed successfully.
```
`Key Usage  Digital Signature, Key Encipherment (configured in template)`

**From the certutil output — record the following:**

| Field                    | Value                                            |
|--------------------------|--------------------------------------------------|
| Subject                  | CN=Svc Autoenroll                                |
| Issuer                   | CN=CVI Issuing CA 1, DC=corp, DC=cvilab, DC=local|
| Serial Number            | 4400000009217b3df4da50401000000000009            |
| Key Usage                | Digital Signature, Key Encipherment              |
| Enhanced Key Usage (EKU) |Client Authentication                             |
| Validity: Not Before     | 5/24/2026 9:25 AM                                |
| Validity: Not After      | 4/25/2027 7:36 PM                                |
| Thumbprint               | 762c31ae2ec7b9430ad86b6a616199e4369a74c7         |

---

### In certsrv.msc — Issued Certificates Node

Navigate to **certsrv.msc → CVI Issuing CA 1 → Issued Certificates**.

**Certificate visible in Issued Certificates node:**
- [x] Yes

**Record from the Issued Certificates node:**

| Column                      | Value               |
|-----------------------------|---------------------|
| Request ID                  | 9                   |
| Requester Name              | CORP\svc.autoenroll |
| Certificate Template        | CVI-ServiceAccount  |
| Issued Common Name          | Svc Autoenroll      |
| Certificate Expiration Date | 4/25/2027           |

 **Save this Request ID.** You will use it in Week 12 if you revoke this certificate, and in Lab 03 for the comparison exercise.

---

## Part D — Written Explanation

**What makes a service account certificate different from a user certificate? Address the following:**

1. Key difference in EKU — what does a user certificate include that the service account certificate should not, and why?
2. Subject Name source — both use "Build from AD," but what is different about the identity being represented?
3. Enrollment — who requests a user certificate vs. who requests a service account certificate, and why does this matter?

```
A service account certificate is different from a regular user certificate because it represents a non-human identity used by applications, services, or automated systems. A user certificate may include additional purposes such as Secure Email or Smart Card Logon, while the service account certificate was intentionally restricted to Client Authentication only. This follows the principle of least privilege and reduces the risk of misuse.

Both user certificates and service account certificates can use “Build from Active Directory,” but the identities being represented are different. A normal user certificate represents a human identity, while the service account certificate represents an automated domain identity used by services and applications. Because of this, enrollment permissions must be more tightly controlled.

The enrollment process also differs operationally. User certificates are usually requested interactively by end users, while service account certificates are often issued to automated systems or service identities that support backend authentication workflows. This matters because service account identities are commonly tied to infrastructure services, APIs, and scheduled processes that require stable and secure authentication.
```

**What are the operational risks of relying on password authentication for service accounts instead of certificate-based authentication?**

```
Using passwords for service accounts creates operational and security risks. Service account passwords are often long-lived and may be stored in scripts, configuration files, or scheduled tasks. If a password becomes exposed, attackers may gain unauthorized access to services and systems.

Certificate-based authentication improves lifecycle control because certificates have defined expiration periods, can be revoked quickly, and are centrally issued, monitored, and audited by the CA. Certificates also reduce the risk of password spraying and credential reuse attacks.
```

---

## Reflection

**One thing about the CVI-ServiceAccount template design that was a non-obvious decision:**

```
One of the most valuable parts of this lab was understanding how identity type affects certificate enrollment behavior. Initially, the MMC “Service account” workflow from the resource pack did not display the svc.autoenroll account, even though it was functioning as a service identity. Troubleshooting this issue helped clarify the difference between a Windows managed service identity and an Active Directory user principal being used operationally as a service account.

Another important takeaway was realizing how much control certificate templates provide over security and operational behavior. Small configuration decisions such as EKUs, enrollment permissions, subject name handling, and private key export settings can significantly affect how certificates are used and protected in production environments.The lab also reinforced why certificate-based authentication provides stronger lifecycle control and operational security than relying only on passwords for service identities.
```

**What would you change about this template if this were a production environment rather than a lab?**

```
If this were a production environment, I would add stricter lifecycle and issuance controls around the service account certificate workflow. In the lab environment, certificates were configured to auto-issue for simplicity, but in production I would consider enabling approval workflows forhigh-value or highly privileged service accounts.

I would also implement certificate lifecycle monitoring and automated renewal workflows. The lesson notes emphasized how poor certificate tracking and expiration management can lead to operational outages if certificates are not renewed on time. In a production PKI environment, automated renewal, expiration alerting, and centralized certificate inventory management would help reduce operational risk and improve visibility across systems.

Another improvement would be integrating the certificate workflow into a broader identity and secrets management process. Service account certificates are commonly used in backend authentication, APIs, and automated systems, so having centralized monitoring, auditing, and revocation procedures would help improve long-term operational security and maintain better control over non-human identities.
```

---

## Submission Checklist

- [x] Pre-lab verification completed and outputs recorded
- [x] Part A: All five template design decisions documented with rationale
- [x] Part A: Template created as CVI-ServiceAccount and visible in certtmpl.msc
- [x] Part B: Template published to CVI Issuing CA 1
- [x] Part B: Certificate issued to svc.autoenroll — enrollment steps documented
- [x] Part C: certutil output pasted and key fields extracted into table
- [x] Part C: Request ID recorded from certsrv.msc Issued Certificates node
- [x] Part D: Written explanation completed in prose
- [x] Reflection section completed
- [x] File saved as `lab-01-service-account-profile.md`
- [x] File committed to portfolio repo under `labs/week-11/`
