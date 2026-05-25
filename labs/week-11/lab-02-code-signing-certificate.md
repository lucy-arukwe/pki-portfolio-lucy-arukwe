# Lab 02: Issue a Code Signing Certificate

**Student Name:**  Lucy Arukwe
**Date Completed:** May 23, 2026 
**Phase:** 2 | **Week:** 11  
**Submission Path:** `labs/week-11/lab-02-code-signing-certificate.md`

---

## Pre-Lab Verification

Run on PKI-SRV01 before starting.

```powershell
Get-Service -Name CertSvc
certutil -ping
```

**CertSvc status:** Running
**CA responding:** Yes

---

## Part A — Design the CVI-CodeSigning Template

Open the Certificate Templates console: **Run → certtmpl.msc**

### Template Duplication

**Source template duplicated:** Code Signing (use the built-in Code Signing template)

**Compatibility settings:**
- Certification Authority: Windows Server 2003
- Certificate Recipient:   Windows XP / Server 2003

### Design Decisions

**1 — Key Usage**

| Key Usage         | Included? | Reason                                                |
|-------------------|-----------|-------------------------------------------------------|
| Digital Signature |  Yes      | Required for signing scripts and validating integrity |                                       
| Key Encipherment  |  No       | Code signing certificates are not used for encryption |                                                  
| Non-Repudiation   |  No       | Not required for PowerShell Authenticode signing      |                                                

**Explanation of Key Usage decision:**

```
Digital Signature was the only required key usage because the certificate was being used strictly
for signing PowerShell scripts. The purpose of the signature is to prove integrity and signer identity,
not to encrypt data. Adding unnecessary usages such as Key Encipherment would expand the certificate’s
trust scope beyond its intended purpose and violate the principle of least privilege. The resource pack
specifically identifies Digital Signature as the correct key usage for Code Signing templates.
```

**2 — EKU**

| EKU                              | Included? | Reason                                                                |
|----------------------------------|-----------|-----------------------------------------------------------------------|
| Code Signing (1.3.6.1.5.5.7.3.3) | Yes       | Required for PowerShell and the operating system to trust signed code | 
| Client Authentication            | No        | This certificate is not intended for user authentication              |                                                        
| Other                            | No        | Additional EKUs would unnecessarily broaden certificate trust         |                                                            

**Explanation of EKU decision:**

```
The Code Signing EKU controls what the certificate is trusted to do at the operating system and application layer.
PowerShell checks for the presence of the Code Signing EKU before accepting a signature as trusted. If additional EKUs were added unnecessarily, the certificate could become trusted for purposes beyond code signing, which would unnecessarily expand the certificate’s trust scope.
```

**3 — Subject Name**

| Setting             | Value                                   | Reason                                             |
|---------------------|-----------------------------------------|----------------------------------------------------|
| Subject name source | Build from Active Directory information | Identity comes directly from AD                    |
| Subject built from  | User principal name (UPN)               | Ensures the signer identity maps to the AD account |

**4 — Validity and Enrollment Permissions**

| Setting                     | Value     | Reason                                                         |
|-----------------------------|-----------|----------------------------------------------------------------|
| Validity period             | 1 year    | Matches the software distribution lifecycle                    |
| Enroll — account(s) granted | pki.admin | Restricts issuance to a trusted administrator                  |                              
| Autoenroll                  | Disabled  | Prevents automatic issuance of high-trust signing certificates |                                          

**Template names:**

| Field                    | Value            |
|--------------------------|------------------|
| Template display name    | CVI Code Signing |
| Template name (internal) | CVI-CodeSigning  |

**Template saved:**
- [x] Yes — visible in certtmpl.msc

---

## Part B — Publish and Issue the Certificate

### Publish the Template

**Steps performed:**

1. certsrv.msc → CVI Issuing CA 1 → Certificate Templates → New → Certificate Template to Issue
2. Selected **CVI-CodeSigning**

**CVI-CodeSigning visible in Certificate Templates node:**
- [x] Yes

### Request the Certificate (as pki.admin)

**Steps performed:**

1. mmc.exe → Certificates snap-in → **My user account**
2. Personal → All Tasks → Request New Certificate
3. Enrollment policy: Active Directory Enrollment Policy
4. Template selected: **CVI-CodeSigning**
5. Enrolled

**Certificate issued:**
- [X] Yes — immediately
- [ ] Pending — describe:

```
(describe outcome)
```

**Request ID from certsrv.msc Issued Certificates node:** 7

**Save this Request ID.** It is used in Week 12 revocation and in Lab 03.

### Verify the Certificate

```powershell
certutil -store My
```
certutil -store My "<thumbprint>" did not properly resolve the certificate object in this environment, so 
verification was completed using PowerShell’s certificate provider instead.

Verification command used
`Get-ChildItem -Path Cert:\CurrentUser\My -CodeSigningCert | Format-List *`

**Full certutil output for the code signing certificate:**

```
PSPath                      : Microsoft.PowerShell.Security\Certificate::CurrentUser\My\647515E560C6CD47BC3A9507024D1356F73EEA32
PSParentPath                : Microsoft.PowerShell.Security\Certificate::CurrentUser\My
PSChildName                 : 647515E560C6CD47BC3A9507024D1356F73EEA32
PSDrive                     : Cert
PSProvider                  : Microsoft.PowerShell.Security\Certificate
PSIsContainer               : False
EnhancedKeyUsageList        : {Code Signing (1.3.6.1.5.5.7.3.3)}
DnsNameList                 : {PKI Admin}
SendAsTrustedIssuer         : False
EnrollmentPolicyEndPoint    : Microsoft.CertificateServices.Commands.EnrollmentEndPointProperty
EnrollmentServerEndPoint    : Microsoft.CertificateServices.Commands.EnrollmentEndPointProperty
PolicyId                    : {41635678-B3E8-4BD7-8FE7-D49A1E336991}
Archived                    : False
Extensions                  : {System.Security.Cryptography.Oid, System.Security.Cryptography.Oid,
                               System.Security.Cryptography.Oid, System.Security.Cryptography.Oid...}
FriendlyName                :
IssuerName                  : System.Security.Cryptography.X509Certificates.X500DistinguishedName
NotAfter                    : 4/25/2027 7:36:58 PM
NotBefore                   : 5/23/2026 9:22:21 AM
HasPrivateKey               : True
PrivateKey                  : System.Security.Cryptography.RSACryptoServiceProvider
PublicKey                   : System.Security.Cryptography.X509Certificates.PublicKey
RawData                     : {48, 130, 6, 16...}
SerialNumber                : 4400000007172A43E46A06421E000000000007
SubjectName                 : System.Security.Cryptography.X509Certificates.X500DistinguishedName
SignatureAlgorithm          : System.Security.Cryptography.Oid
Thumbprint                  : 647515E560C6CD47BC3A9507024D1356F73EEA32
Version                     : 3
Handle                      : 1696970724912
Issuer                      : CN=CVI Issuing CA 1, DC=corp, DC=cvilab, DC=local
Subject                     : CN=PKI Admin, OU=PKI Admins, DC=corp, DC=cvilab, DC=local

```

**Key fields confirmed:**

| Field      | Value                                                     |
|------------|-----------------------------------------------------------|
| Subject    | CN=PKI Admin, OU=PKI Admins, DC=corp, DC=cvilab, DC=local |
| EKU        | Code Signing (1.3.6.1.5.5.7.3.3)                          |
| Validity   | 5/23/2026 9:22:21 AM → 4/25/2027 7:36:58 PM               |
| Thumbprint | 647515E560C6CD47BC3A9507024D1356F73EEA32                  |

**EKU = 1.3.6.1.5.5.7.3.3 (Code Signing) confirmed:**
- [x] Yes
- [ ] No — describe discrepancy:

---

## Part C — Sign a PowerShell Script

### Create the Test Script

Create a simple PowerShell script on PKI-SRV01:

```powershell
# Create the test script
$scriptContent = @'
# CVI Phase 2 — Week 11 Code Signing Test
Write-Host "This script is signed with a CVI code signing certificate."
Write-Host "Issued to: pki.admin"
Write-Host "Date: $(Get-Date)"
'@

New-Item -Path "C:\Scripts" -ItemType Directory -Force
Set-Content -Path "C:\Scripts\Test-CVI.ps1" -Value $scriptContent
```

**Script created at C:\Scripts\Test-CVI.ps1:**
- [x] Yes

---

### Sign the Script

```powershell
# Get the code signing certificate
$cert = Get-ChildItem -Path Cert:\CurrentUser\My -CodeSigningCert | Select-Object -First 1

# Confirm this is the right certificate
$cert | Select-Object Subject, Thumbprint, NotAfter
```

**Output of certificate selection:**

```
Subject:
CN=PKI Admin, OU=PKI Admins, DC=corp, DC=cvilab, DC=local

Thumbprint:
647515E560C6CD47BC3A9507024D1356F73EEA32

NotAfter:
4/25/2027 7:36:58 PM
```

```powershell
# Sign the script
$result = Set-AuthenticodeSignature -FilePath "C:\Scripts\Test-CVI.ps1" -Certificate $cert
$result
```

**Set-AuthenticodeSignature output:**

```
Directory: C:\Scripts

SignerCertificate                           Status   Path
-----------------                           ------   ----
647515E560C6CD47BC3A9507024D1356F73EEA32    Valid    Test-CVI.ps11
```

---

### Verify the Signature

```powershell
Get-AuthenticodeSignature -FilePath "C:\Scripts\Test-CVI.ps1"
```

**Full Get-AuthenticodeSignature output:**

```
Status : Valid
StatusMessage : Signature verified.
SignatureType : Authenticode
```

**Status:**
- [x] Valid
- [ ] Other — describe:

**Is a timestamp present?**

```powershell
(Get-AuthenticodeSignature "C:\Scripts\Test-CVI.ps1").TimeStamperCertificate
```

**TimeStamperCertificate output:**

```
$null 
```

**Timestamp present:**
- [ ] Yes — note the timestamp authority:
- [x] No — note this in Part D

---

### Hash Mismatch Test (Destructive)

Modify the script after signing to verify the signature breaks:

```powershell
# Add content to the signed script
Add-Content -Path "C:\Scripts\Test-CVI.ps1" -Value "# Modified after signing"

# Re-verify
Get-AuthenticodeSignature -FilePath "C:\Scripts\Test-CVI.ps1"
```

**Get-AuthenticodeSignature output after modification:**

```
Directory: C:\Scripts

SignerCertificate                           Status         Path
-----------------                           ------         ----
647515E560C6CD47BC3A9507024D1356F73EEA32   HashMismatch   Test-CVI.ps1
```

**Status after modification:**
- [x] HashMismatch
- [ ] Other — describe:

---

## Part D — Written Explanation

**What does the Code Signing EKU enforce, and at what layer?**

Cover: what application or OS component checks for the Code Signing EKU, what it does when the EKU is present vs. absent, and how this is different from the cryptographic validity check.

```
The Code Signing EKU is enforced primarily at the operating system and application trust layer rather than at the raw cryptographic layer.
When PowerShell or Windows verifies a signed script, it first checks whether the signing certificate contains the Code Signing EKU (1.3.6.1.5.5.7.3.3).
If the EKU is missing, the operating system may reject the signature even if the cryptographic signature itself is mathematically valid.

The cryptographic validity check only proves that the file hash matches the signed hash and that the certificate chain is trusted.
The EKU check adds an additional authorization layer that determines what the certificate is actually allowed to do. This separation is important because
a certificate could technically sign data correctly while still being unauthorized for code signing purposes.
```

**What did the hash mismatch test demonstrate about what the signature is protecting?**

Cover: what the signature covers (the code hash), what the mismatch status means, and why this matters for software integrity in a production environment.

```
The hash mismatch test demonstrated that the digital signature protects the integrity of the script contents. When the script was originally signed, PowerShell
calculated a hash of the file and signed that hash using the private key associated with the Code Signing certificate.

After modifying the script, the file contents changed, which caused the calculated hash to differ from the original signed hash.
As a result, Get-AuthenticodeSignature returned HashMismatch, proving that the file had been altered after signing.

This matters in production environments because it prevents tampering. If malware or an attacker modifies a signed administrative
script after distribution, the signature verification process detects the change immediately.
```

**Should the CVI-CodeSigning template require CA certificate manager approval in a production environment? Why or why not?**

```
Yes, in a production environment the Code Signing template should typically require certificate manager approval because code signing certificates
are high-trust credentials. A compromised code signing certificate could allow malicious software to appear legitimate and trusted within the environment.

Restricting enrollment alone is important, but adding managerial approval creates another control point before issuance.
This reduces the risk of accidental issuance, privilege abuse, or compromised administrative accounts obtaining signing certificates without oversight.
This is especially important because a compromised code signing certificate becomes a software supply chain security risk rather than only a PKI issue.
```

---

## Reflection

**Why is a timestamp operationally significant for a code signing certificate — particularly for software that will be distributed and executed over a long period?**

```
A timestamp records the exact moment the signature was created. This allows signed software to remain trusted even after the signing certificate expires, as long as the certificate was valid at the time of signing.

Without timestamping, software signatures may become invalid immediately after the certificate expires, forcing organizations to re-sign older software releases.
Timestamping is especially important for long-term software distribution, archived scripts, and production environments where applications may remain deployed for years.
```

**One thing about the code signing workflow you would want to understand better or configure differently:**

```
One thing I would want to understand better is how enterprise environments securely automate code signing within CI/CD pipelines while still protecting the private signing key.
I would also want to explore how Hardware Security Modules (HSMs) are integrated into enterprise signing workflows so that private keys remain non-exportable while still allowing automated build systems to sign software securely.
```

---

## Submission Checklist

- [x] Pre-lab verification completed
- [x] Part A: CVI-CodeSigning template designed with rationale for all settings
- [x] Part A: Template created and visible in certtmpl.msc
- [x] Part B: Template published to CVI Issuing CA 1
- [x] Part B: Certificate issued to pki.admin — Request ID recorded
- [x] Part B: certutil output pasted with EKU confirmed as Code Signing
- [x] Part C: Test script created
- [x] Part C: Script signed and Set-AuthenticodeSignature output pasted
- [x] Part C: Get-AuthenticodeSignature output = Valid, pasted
- [x] Part C: Timestamp check output pasted
- [x] Part C: Hash mismatch test performed and output pasted (Status = HashMismatch)
- [x] Part D: Written explanation in prose — EKU enforcement and hash mismatch
- [x] Reflection completed
- [x] File saved as `lab-02-code-signing-certificate.md`
- [x] File committed to portfolio repo under `labs/week-11/`
