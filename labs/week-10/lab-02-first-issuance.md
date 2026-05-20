# Lab 02: Issue Your First Certificate from a Custom Template

**Student Name:** Lucy Arukwe
**Date Completed:** May 19, 2026 
**Phase:** 2 | **Week:** 10  
**Submission Path:** `labs/week-10/lab-02-first-issuance.md`

---

## Pre-Lab Verification

Run on PKI-SRV01 before starting.

```powershell
Get-Service -Name CertSvc
certutil -ping
```

**CertSvc status:** Running
**CA responding (certutil -ping):**
- [x] Yes
- [ ] No — action taken:

**CVI-WebServer template visible in certtmpl.msc (from Lab 01):**
- [x] Yes
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
- [x] Yes
- [ ] No — describe what happened:

```
The CVI-WebServer template appeared successfully under the Certificate Templates node in certsrv.msc. The Intended Purpose displayed as Server Authentication.
```

![CVI-WebServer Template Published](../../assets/screenshots/week-10/cvi-webserver-template-node.png)

---

## Part B — Request the Certificate via MMC

**Steps performed on PKI-SRV01, logged in as CORP\pki.admin:**

1. Opened **mmc.exe** → **File → Add/Remove Snap-in**
2. Added **Certificates** snap-in
3. Selected: Computer account (`Computer account` / My user account / Service account)
4. Navigated to **Personal → Certificates**
5. Right-clicked → **All Tasks → Request New Certificate**
6. Proceeded through the Certificate Enrollment wizard

**Certificate Enrollment wizard — enrollment policy selected:**

```
Active Directory Enrollment Policy 
```

**Templates shown in the wizard:**

```
Administrator
Basic EFS
EFS Recovery Agent
User
CVI-WebServer
```

**CVI-WebServer template visible:**
- [ ] Yes
- [x] No — troubleshooting steps taken:
```
Initial enrollment attempts did not work because the Certificates snap-in was first added using “My user account,” and the certificate did not appear in the Current User certificate store.
The following troubleshooting steps were then performed:

1. Reconfirmed the CVI-WebServer template settings in certtmpl.msc:
   - Validity period: 1 year
   - Renewal period: 6 weeks
   - Subject Name: Supply in the request
   - PKI Admins: Full Control
   - Authenticated Users: Read, Write, Enroll

2. Republished the CVI-WebServer template in certsrv.msc.

3. Reopened mmc.exe and added the Certificates snap-in using:
   - Computer account
   - Local Computer

4. Navigated to:
   Certificates (Local Computer)
   → Personal
   → Certificates

5. Started a new certificate request:
   - All Tasks → Request New Certificate
   - Selected Active Directory Enrollment Policy
   - Selected CVI-WebServer

6. A yellow warning icon appeared and the Enroll button was grayed out.

7. Opened the warning link → Certificate Properties → Subject tab.

8. Added:
   - Type: Common Name
   - Value: CVI-WebServer

9. Clicked OK.

10. The warning cleared, the Enroll button became available, and the certificate was issued successfully.
```

**Subject name entered (if prompted):**

```
CVI-WebServer
```

**Certificate request submitted:**
- [x] Yes — certificate issued immediately
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

| Field      | Value                        |
|------------|------------------------------|
| Issued to  | CVI-WebServer                |
| Issued by  | CVI Issuing CA 1             |
| Valid from | 5/19/2026 6:45 AM            |
| Valid to   | 4/25/2027 7:36 PM            |

**Details tab — record the following fields:**

| Field                | Value                                         |
|----------------------|-----------------------------------------------|
| Serial Number        | 44000000049ed3c730770f574700000000004         |
| Signature Algorithm  | sha256RSA                                     |
| Subject              | CN=CVI-WebServer                              |
| Key Usage            | Digital Signature, Key Encipherment           |
| Enhanced Key Usage   | Server Authentication                         |
| Subject Alternative Name (if present) |Not Present                   |
| Thumbprint           | 7bcd7f64ed6f40d278293635afaae52c8e8cc7e3      |          

---

### Via certutil

Export the certificate thumbprint from the Details tab, then run:

```powershell
certutil -store My "<thumbprint>"
```

Replace `<thumbprint>` with the thumbprint value (no spaces).

**Full certutil output:**

```
**Full certutil output:**

```powershell
================ Certificate 0 ================
Serial Number: 44000000049ed3c730770f574700000000004
Issuer: CN=CVI Issuing CA 1, DC=corp, DC=cvilab, DC=local
NotBefore: 5/19/2026 6:45 AM
NotAfter: 4/25/2027 7:36 PM
Subject: CN=CVI-WebServer
Non-root Certificate
Template: CVI-WebServer

Cert Hash(sha1): 7bcd7f64ed6f40d278293635afaae52c8e8cc7e3

Provider = Microsoft RSA SChannel Cryptographic Provider

Private key is NOT exportable
Encryption test passed

CertUtil: -store command completed successfully.
```

---

### In certsrv.msc — Issued Certificates Node

Navigate to **certsrv.msc → CVI Issuing CA 1 → Issued Certificates**.

**Does the certificate appear in the Issued Certificates node?**
- [x] Yes

**Record from the Issued Certificates node:**

| Column                      | Value             |
|---------------------------- |-------------------|
| Request ID                  | 4                 |
| Requester Name              | CORP\PKI-SRV01$   |
| Certificate Template        | CVI-WebServer     |
| Issued Common Name          | CVI-WebServer     |
| Certificate Expiration Date | 4/25/2027 7:36 PM |

---

## Part D — Write-Up: The Issuance Workflow

Describe the full certificate issuance workflow in your own words. Cover:

1. What happened in Active Directory when you published the template
2. What the MMC Certificate Enrollment wizard sent to the CA
3. What the CA evaluated before issuing the certificate
4. Where the issued certificate was placed and why

```
Publishing the CVI-WebServer template to CVI Issuing CA 1 basically made the template available for certificate requests on that CA. The template itself stayed stored in Active Directory, but publishing it told the CA that it was allowed to issue certificates using that template. The template also contained the rules the CA would follow, including the validity period, enrollment permissions, key usage settings, and the requirement to manually supply the subject name during enrollment.

When the MMC Certificate Enrollment wizard was opened, it pulled the available templates from Active Directory and displayed CVI-WebServer as an option. After the common name was entered, Windows generated a 2048-bit RSA key pair locally and created a certificate signing request containing the public key and certificate information. The CA then checked whether the requester had permission to enroll, confirmed that the request matched the template settings, and applied the configured certificate extensions and validity period. Since all the requirements were met, the CA signed the certificate using its private key and returned it to the system. The certificate was automatically installed into the Local Computer Personal certificate store because the request was performed using the computer account context. A record of the issuance was also added to the CA database and appeared in the Issued Certificates node in certsrv.msc for tracking and future management tasks such as auditing or certificate revocation.
```

**One thing about the issuance process that you did not expect or want to understand better:**

```
One thing I did not expect was that publishing a certificate template to Active Directory was not enough by itself for certificate issuance. I originally thought that once a template existed and users had enrollment permissions, any Enterprise CA in the environment would automatically be able to issue certificates from it. The lab helped me understand that the template must also be explicitly published to a specific CA before that CA will recognize and service requests for it.

What I would like to understand better is how this works in larger enterprise environments with multiple issuing CAs. For example, if an organization has separate issuing CAs for servers, users, VPN authentication, and smart cards, how do administrators decide which templates are published to which CAs, and how is that managed securely at scale?
```

---

## Submission Checklist

- [x] Pre-lab verification completed
- [x] Part A: CVI-WebServer template published to CVI Issuing CA 1
- [x] Part A: Template visible in certsrv.msc Certificate Templates node — confirmed
- [x] Part B: Certificate requested via MMC — request submitted
- [x] Part B: Enrollment wizard observations documented
- [x] Part C: Certificate details recorded from MMC (General + Details tabs)
- [x] Part C: certutil -store My output pasted
- [x] Part C: Certificate confirmed in certsrv.msc Issued Certificates node
- [x] Part D: Issuance workflow write-up completed in own words
- [x] File saved as `lab-02-first-issuance.md`
- [x] File committed to portfolio repo under `labs/week-10/`
