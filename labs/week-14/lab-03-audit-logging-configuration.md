# Lab 03: Audit Logging Configuration and Verification *(Stretch)*

**Student Name:**  Lucy Arukwe

**Date Completed:**  July 10, 2026

**Phase:** 2 | **Week:** 14  

**Submission Path:** `labs/week-14/lab-03-audit-logging-configuration.md`

---

## Overview

In this stretch lab, you configure full CA audit logging on PKI-SRV01, generate test events, and verify that the correct Windows Security log events appear. After this lab, your CA will record certificate issuance, revocation, CRL publication, configuration changes, and key operations as auditable Security log events.

This configuration is non-destructive. Audit logging can be disabled at any time with `certutil -setreg CA\AuditFilter 0`. No CA data is modified — you are only changing what the CA records about its own actions.

**This lab requires two configuration steps — both are required:**
1. Set the CA AuditFilter registry value (tells the CA what to log)
2. Enable Audit Object Access in Local Security Policy (tells Windows where to write the events)

---

## Lab Environment

| Component | Details |
|-----------|---------|
| CA Server | PKI-SRV01 (192.168.10.20) |
| CA Name | CVI Issuing CA 1 |
| Login Account | CORP\pki.admin |
| Domain | corp.cvilab.local |

---

## Pre-Lab Check

### Step 1 — Confirm Login and CA Status

```powershell
whoami
Get-Service CertSvc
certutil -ping
```

```
PS C:\Windows\system32> whoami
corp\pki.admin
PS C:\Windows\system32> Get-Service CertSvc
Status   Name               DisplayName
------   ----               -----------
Running  CertSvc            Active Directory Certificate Services
PS C:\Windows\system32> certutil -ping
Connecting to PKI-SRV01.corp.cvilab.local\CVI Issuing CA 1 ...
Server "CVI Issuing CA 1" ICertRequest2 interface is alive (0ms)
CertUtil: -ping command completed successfully.
```

---

## Part A — Document the Pre-Configuration State

Before making any changes, record the current audit configuration. This creates a baseline for the before/after comparison in Part D.

### Step 1 — Check Current CA AuditFilter Value

```powershell
certutil -getreg CA\AuditFilter
```

**Expected output (default — no audit logging):**
```
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\CertSvc\Configuration\CVI Issuing CA 1
  AuditFilter REG_DWORD = 0 (0)
CertUtil: -getreg command completed successfully.
```

```
PS C:\Windows\system32> certutil -getreg CA\AuditFilter
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\CertSvc\Configuration\CVI Issuing CA 1\AuditFilter:

  AuditFilter REG_DWORD = cc (204)
CertUtil: -getreg command completed successfully.
```

**Current AuditFilter value:** ` AuditFilter REG_DWORD = cc (204)`

### Step 2 — Check Local Security Policy — Audit Object Access

Run the following to check the current audit policy state:

```powershell
auditpol /get /subcategory:"Certification Services"
```

```
System audit policy
Category/Subcategory                      Setting
Object Access
  Certification Services                  No Auditing
```

Alternatively, navigate manually:
- Run: `secpol.msc`
- Navigate: Security Settings → Local Policies → Audit Policy → Audit object access
- Record the current setting:

**Current Audit Object Access setting:**
- [x] Not Configured / No Auditing
- [ ] Success Only
- [ ] Failure Only
- [ ] Success and Failure

### Step 3 — Verify No CA Events in Security Log (Pre-Configuration)

```powershell
# Check for any existing CA-related events in the Security log
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = @(4887, 4870, 4872, 4880, 4881)
} -ErrorAction SilentlyContinue | Select-Object TimeCreated, Id -First 10
```

```
(No output returned)
```

**CA-related Security log events found before configuration:**
- [x] None — baseline confirmed
- [ ] Some found — record count and Event IDs:

---

## Part B — Configure CA Audit Logging

### Step 1 — Set the CA AuditFilter

This command sets the CA to log four categories: certificate issuance (0x4), revocation (0x8), configuration changes (0x40), and key operations (0x80). Combined value: 0xCC.

Run from an elevated PowerShell prompt on PKI-SRV01:

```powershell
certutil -setreg CA\AuditFilter 0xCC
```

**Expected output:**
```
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\CertSvc\Configuration\CVI Issuing CA 1
Old Value:
  AuditFilter REG_DWORD = 0 (0)
New Value:
  AuditFilter REG_DWORD = 0xcc (204)
CertUtil: -setreg command completed successfully.
The CertSvc service may need to be restarted for changes to take effect.
```

```
PS C:\Windows\system32> certutil -setreg CA\AuditFilter 0xCC
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\CertSvc\Configuration\CVI Issuing CA 1\AuditFilter:

New Value:
  AuditFilter REG_DWORD = cc (204)
CertUtil: -setreg command completed successfully.
The CertSvc service may need to be restarted for changes to take effect.
```

**AuditFilter set to 0xCC without errors:**
- [x] Yes
- [ ] No — error:

### Step 2 — Restart the CA Service to Apply the Change

```powershell
net stop certsvc
net start certsvc
```

```
PS C:\Windows\system32> net stop certsvc
The Active Directory Certificate Services service is stopping.
The Active Directory Certificate Services service was stopped successfully.

PS C:\Windows\system32> net start certsvc
The Active Directory Certificate Services service is starting.
The Active Directory Certificate Services service was started successfully.
```

**CA service restarted:**
- [x] Yes
- [ ] No — error:

### Step 3 — Verify the New AuditFilter Value

```powershell
certutil -getreg CA\AuditFilter
```

**Expected:**
```
AuditFilter REG_DWORD = 0xcc (204)
```

```
PS C:\Windows\system32> certutil -getreg CA\AuditFilter
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\CertSvc\Configuration\CVI Issuing CA 1\AuditFilter:

  AuditFilter REG_DWORD = cc (204)
CertUtil: -getreg command completed successfully.
```

**AuditFilter is now 0xcc (204):**
- [x] Yes — confirmed
- [ ] No — actual value:

### Step 4 — Enable Audit Object Access in Local Security Policy

**Method A — GUI (secpol.msc):**
1. Run: `secpol.msc`
2. Navigate: Security Settings → Local Policies → Audit Policy
3. Double-click: **Audit object access**
4. Check: **[✓] Success** and **[✓] Failure**
5. Click **OK**

**Method B — Command line:**
```powershell
auditpol /set /subcategory:"Certification Services" /success:enable /failure:enable
```

```
The command was successfully executed.
```

### Step 5 — Verify Audit Object Access Is Enabled

```powershell
auditpol /get /subcategory:"Certification Services"
```

**Expected:**
```
System audit policy
Category/Subcategory                      Setting
Object Access
  Certification Services                  Success and Failure
```

```
PS C:\Windows\system32> auditpol /get /subcategory:"Certification Services"
System audit policy
Category/Subcategory                      Setting
Object Access
  Certification Services                  Success and Failure
```

**Audit Object Access is set to Success and Failure:**
- [x] Yes — both steps confirmed
- [ ] No — describe what is missing:

---

## Part C — Generate Test Events

With audit logging configured, generate three types of auditable CA events: certificate issuance, certificate revocation, and CRL publication. Each will produce a corresponding Security log event.

### Step 1 — Issue a Test Certificate

Issue a certificate from one of the templates configured in Week 10. You can use the Certification Authority MMC or certreq.

**Option A — Using the Certification Authority MMC:**
1. Open: `certsrv.msc` (Certification Authority console)
2. Right-click the CA name → **All Tasks → Submit new request**
3. Or: expand **Certificate Templates** → right-click a template → **Issue** (if this option appears)
4. For a simpler approach: use the web enrollment page at `http://PKI-SRV01/certsrv`

**Option B — Using certreq (from an elevated PowerShell prompt):**
```powershell
# Create a minimal INF file for a test request
$inf = @"
[Version]
Signature="\$Windows NT\$"
[NewRequest]
Subject = "CN=audit-test.corp.cvilab.local"
KeySpec = 1
KeyLength = 2048
Exportable = TRUE
MachineKeySet = TRUE
SMIME = False
PrivateKeyArchive = FALSE
UserProtected = FALSE
UseExistingKeySet = FALSE
ProviderName = "Microsoft RSA SChannel Cryptographic Provider"
ProviderType = 12
RequestType = CMC
[RequestAttributes]
CertificateTemplate=WebServer
"@
$inf | Out-File -FilePath C:\Temp\audit-test.inf -Encoding ASCII
New-Item -ItemType Directory -Path C:\Temp -Force | Out-Null

# Submit the request
certreq -new C:\Temp\audit-test.inf C:\Temp\audit-test.req
certreq -submit -config "PKI-SRV01\CVI Issuing CA 1" C:\Temp\audit-test.req C:\Temp\audit-test.cer
```

> **If the template requires manager approval:** The request will be in pending state. In the Certification Authority console, navigate to Pending Requests, right-click the request, and select **Issue**.

**Certificate issued — confirmation:**

```powershell
# Verify the certificate appears in the database
certutil -view -restrict "Disposition=20" -out "RequestID,CommonName,NotAfter" | tail -10
```

```
PS C:\Windows\system32> certutil -view -restrict "RequestID=21" -out "SerialNumber,CommonName"
Schema:
  Column Name                   Localized Name                Type    MaxLength
  ----------------------------  ----------------------------  ------  ---------
  SerialNumber                  Serial Number                 String  128 -- Indexed
  CommonName                    Issued Common Name            String  8192 -- Indexed

Row 1:
  Serial Number: "4400000015ae990afbe0514748000000000015"
  Issued Common Name: "audit-test.corp.cvilab.local"

Maximum Row Index: 1

1 Rows
   2 Row Properties, Total Size = 132, Max Size = 76, Ave Size = 66
   0 Request Attributes, Total Size = 0, Max Size = 0, Ave Size = 0
   0 Certificate Extensions, Total Size = 0, Max Size = 0, Ave Size = 0
   2 Total Fields, Total Size = 132, Max Size = 76, Ave Size = 66
CertUtil: -view command completed successfully.
PS C:\Windows\system32>
```

Request ID of the issued test certificate: `21`
Serial Number: `4400000015ae990afbe0514748000000000015`

### Step 2 — Revoke the Test Certificate

```powershell
# Replace <serial_number> with the serial number from Step 1
certutil -revoke <serial_number> 5
# Reason 5 = Cessation of Operation
```

**Expected output:**
```
Certificate "1A 00 00 00 0x..." revoked
CertUtil: -revoke command completed successfully.
```

```
PS C:\Windows\system32> certutil -revoke 4400000015ae990afbe0514748000000000015 5
Revoking "4400000015ae990afbe0514748000000000015" -- Reason: Cessation of Operation
CertUtil: -revoke command completed successfully.
```

**Certificate revoked without errors:**
- [x] Yes
- [ ] No — error:

### Step 3 — Publish a Fresh CRL

```powershell
certutil -CRL
```

```
PS C:\Windows\system32> certutil -CRL
CertUtil: -CRL command completed successfully.
```

**CRL published without errors:**
- [x] Yes
- [ ] No — error:

### Step 4 — Wait 30 Seconds

The Security log may take a brief moment to write events after CA operations. Wait 30 seconds before proceeding to Part D.

---

## Part D — Verify Security Log Events

### Step 1 — Search for Issuance Event (4887)

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4887
    StartTime = (Get-Date).AddHours(-1)
} | Select-Object TimeCreated, Id, Message | Format-List
```

```
TimeCreated : 7/10/2026 1:29:20 PM
Id          : 4887
Message     : Certificate Services approved a certificate request and issued a certificate.

              Request ID:       21
              Requester:        CORP\pki.admin
              Attributes:
              ccm:PKI-SRV01.corp.cvilab.local
              Disposition:      3
              SKI:              da 33 a9 82 dd 17 6d c3 97 6f f6 41 3e e7 32 bf a6 21 21 1d
              Subject:  CN=audit-test.corp.cvilab.local

```

**Event 4887 (Certificate Issued) found:**
- [x] Yes — timestamp: `7/10/2026 1:29:20 PM`
- [ ] No — troubleshoot: confirm both AuditFilter and Audit Object Access are configured

**Key fields from Event 4887:**
- Request ID: `21`
- Requester: `CORP\pki.admin`
- Certificate Template: `WebServer`

### Step 2 — Search for Revocation Event (4870)

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4870
    StartTime = (Get-Date).AddHours(-1)
} | Select-Object TimeCreated, Id, Message | Format-List
```

```
TimeCreated : 7/10/2026 1:45:49 PM
Id          : 4870
Message     : Certificate Services revoked a certificate.

              Serial Number:    4400000015ae990afbe0514748000000000015
              Reason:   5
```

**Event 4870 (Certificate Revoked) found:**
- [x] Yes — timestamp: `7/10/2026 1:45:49 PM`
- [ ] No — troubleshoot:

**Key fields from Event 4870:**
- Serial Number: `4400000015ae990afbe0514748000000000015`
- Reason Code: `5 (Cessation of Operation)` (should be 5 — Cessation of Operation)

### Step 3 — Search for CRL Publication Event (4872)

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4872
    StartTime = (Get-Date).AddHours(-1)
} | Select-Object TimeCreated, Id, Message | Format-List
```

```
TimeCreated : 7/10/2026 1:47:20 PM
Id          : 4872
Message     : Certificate Services published the certificate revocation list (CRL).

              Base CRL: No
              CRL Number:       34
              Key Container:    CVI Issuing CA 1
              Next Publish:     7/11/2026 8:47 PM 11.633s
              Publish URLs:     C:\Windows\system32\CertSrv\CertEnroll\CVI Issuing CA 1+.crl;

TimeCreated : 7/10/2026 1:47:20 PM
Id          : 4872
Message     : Certificate Services published the certificate revocation list (CRL).

              Base CRL: Yes
              CRL Number:       34
              Key Container:    CVI Issuing CA 1
              Next Publish:     7/17/2026 8:47 PM 11.633s
              Publish URLs:     C:\Windows\system32\CertSrv\CertEnroll\CVI Issuing CA 1.crl;

```

**Event 4872 (CRL Published) found:**
- [x] Yes — timestamp: `7/10/2026 1:47:20 PM`
- [ ] No — troubleshoot:

### Step 4 — Event Viewer Verification (Screenshot or Copy)

Open Event Viewer → Windows Logs → Security. Filter for Event IDs 4887, 4870, and 4872.

Provide either a screenshot of the filtered Security log showing all three events, or paste the full Message field for each event.

```
Keywords       Date and Time         Source                            Event ID  Task Category
Audit Success  7/10/2026 1:47:20 PM  Microsoft Windows security audit  4872      Certification Services
Audit Success  7/10/2026 1:47:20 PM  Microsoft Windows security audit  4872      Certification Services
Audit Success  7/10/2026 1:45:49 PM  Microsoft Windows security audit  4870      Certification Services
Audit Success  7/10/2026 1:29:20 PM  Microsoft Windows security audit  4887      Certification Services
```
The General tab of the selected Event 4872 (Delta CRL) showed:


Base CRL: No
CRL Number: 34
Key Container: CVI Issuing CA 1
Next Publish: 7/11/2026 8:47 PM
Publish URLs: C:\Windows\system32\CertSrv\CertEnroll\CVI Issuing CA 1+.crl


All three event types (4887, 4870, 4872) are confirmed present in the Security log.

---

## Part E — Before/After Comparison and Analysis

### Configuration State Comparison

| Configuration Item | Before | After |
|---|---|---|
| CA AuditFilter value | | 0xCC (204) |
| Audit Object Access policy | | Success and Failure |
| CA-related Security log events | None | 4887, 4870, 4872 present |

### Analysis Questions

**1. The AuditFilter value 0xCC enables four categories. List the four categories and their individual hex values that combine to produce 0xCC. (Hint: 0xCC = 0x4 + 0x8 + 0x40 + 0x80)**

```
The AuditFilter value of 0xCC is made up of four audit categories:

a. 0x4 (Certificate Issuance) – Records when the CA approves and issues a certificate. This creates Event ID 4887 and includes useful information such as the requester, certificate template, Request ID, and subject.

b. 0x8 (Certificate Revocation) – Records when a certificate is revoked. This creates Event ID 4870 and includes details such as the certificate serial number and the reason for the revocation.

c. 0x40 (CA Configuration Changes) – Records changes made to the CA's configuration, such as registry settings. These events help administrators track who made changes and when they were made.

d. 0x80 (CA Key Operations) – Records important operations involving the CA's private keys, such as key archival and key recovery. These events are useful for monitoring sensitive key management activities.

When these four values are added together (4 + 8 + 64 + 128), the result is 204, which is 0xCC in hexadecimal.
```

**2. You configured Audit Object Access in Local Security Policy as a required second step. Why was the AuditFilter change alone insufficient to produce Security log events? What does the Local Security Policy setting control?**

```
Changing the AuditFilter alone was not enough because it only tells the Certification Authority which events it should generate. Windows also needs to know that those events are allowed to be written to the Security log. That is why enabling Audit Object Access in the Local Security Policy is also required.

If only the AuditFilter is configured, the CA is ready to generate audit events, but Windows will not record them in the Security log. Likewise, if only Audit Object Access is enabled but the AuditFilter remains disabled, there are no CA audit events for Windows to record. Both settings must be enabled for Security log events such as certificate issuance, revocation, and CRL publication to appear.
```

**3. Event 4887 (Certificate Issued) includes the Requester Name, Certificate Template, and Serial Number. Compare this to what you found in the Application event log in Lab 01. What information does Event 4887 provide that the Application log does not — and why does that matter for an audit trail?**

```
Event 4887 provides much more detailed information than the Application log. It records who requested the certificate, which certificate template was used, the Request ID, the certificate subject, and other details that uniquely identify the certificate that was issued.

In Lab 01, the Application log mainly showed what was happening with the CA service itself, such as service starts, shutdowns, and operational warnings. It did not record which user requested a certificate or what certificate was issued. That extra information in Event 4887 is important because it creates a proper audit trail, making it possible to trace every certificate back to the person who requested it and verify exactly what the CA issued.

```

**4. You enabled audit logging for four categories (0xCC). The full set of categories would be 0xFF. In a production CA issuing 500 certificates per day, what would be the operational consequence of enabling all categories (0xFF) vs. only the production minimum (0xCC)? What specific high-volume category would you likely want to exclude?**

```
Using the full AuditFilter value of 0xFF would cause the CA to record every available audit category. On a busy production CA issuing hundreds of certificates each day, this would generate a very large number of Security log events. While this provides more information for investigations, it also increases log storage requirements and makes it more difficult to find important events among the large volume of data.

Using the production minimum of 0xCC focuses on the most important security-related activities, such as certificate issuance, revocation, configuration changes, and key operations. This provides a useful audit trail without generating unnecessary log entries. One category that would often be excluded is CA service start and stop events (0x1), since these can occur regularly during maintenance or planned restarts and may create additional log noise without providing much value during normal operations.
```

---

## Lab Report Questions

**1. Explain why both the CA AuditFilter setting and the Local Security Policy Audit Object Access setting are required for Security log events to appear. What happens if only one of the two is configured?**

```
Both settings are required because they perform two different jobs. The AuditFilter tells the Certification Authority which activities should be audited, such as certificate issuance, certificate revocation, CRL publication, and configuration changes. The Local Security Policy setting (Audit Object Access) tells Windows that these audit events are allowed to be written to the Security log.
If only the AuditFilter is configured, the CA can generate audit events, but Windows will not record them in the Security log. If only Audit Object Access is enabled, Windows is ready to record events, but the CA is not generating any audit events to send. Both settings have to work together before Security log events like Event 4887, 4870, and 4872 will appear.
```

**2. A junior administrator tells you: "I can see CRL publication events in the Application log, so I know the CA is being audited." What is wrong with this statement, and what would you tell them to check to determine whether the CA is producing a proper security audit trail?**

```
That statement is not completely accurate because seeing CRL publication events in the Application log only confirms that the CA is recording operational activity. It does not mean that a full security audit trail is being created. The Application log mainly shows events such as service starts, stops, warnings, and CRL publication, but it does not provide the detailed information needed to show who requested, issued, or revoked a certificate.
To confirm that proper CA auditing is enabled, I would check that the AuditFilter is configured correctly and that Audit Object Access is enabled in the Local Security Policy. I would also review the Security log for events such as Event 4887 for certificate issuance, Event 4870 for certificate revocation, and Event 4872 for CRL publication. These Security log events provide the detailed record needed to trace certificate activity and identify who performed each action.
```

**3. The AuditFilter value 0xCC does not include CA service start/stop events (0x1) or backup/restore events (0x2). In what operational scenario would you add these categories — and what would you be looking for in those events?**

```
I would consider enabling the additional audit categories during activities such as scheduled maintenance, troubleshooting, disaster recovery testing, or when investigating a security incident. Recording CA service start and stop events (0x1) would help confirm exactly when the service was taken offline, restarted, or unexpectedly stopped. Backup and restore events (0x2) would also be useful because they create a record whenever the CA database or configuration is backed up or restored.
These events can help administrators build a timeline of what happened during an incident. For example, if the CA suddenly stopped working after a restore, the audit logs could confirm when the restore took place, who performed it, and whether the service restarted successfully afterwards. Although these categories are not always necessary for everyday operations, they become very valuable when troubleshooting problems or investigating unexpected changes.

```

---

## Submission Checklist

## Submission Checklist

- [x] Logged in as CORP\pki.admin — whoami output included
- [x] Pre-configuration AuditFilter value documented (key absent — certutil -getreg output included)
- [x] Pre-configuration Audit Object Access state documented (auditpol and secpol.msc)
- [x] Pre-configuration Security log check performed (no CA events)
- [x] certutil -setreg CA\AuditFilter 0xCC output included
- [x] CA service restarted — net stop/start output included
- [x] Post-configuration certutil -getreg confirms 0xCC (204)
- [x] Audit Object Access enabled — auditpol /get output confirms Success and Failure
- [x] Test certificate issued — Request ID 21 and Serial Number recorded
- [x] Test certificate revoked with reason code 5 — certutil -revoke output included
- [x] CRL published — certutil -CRL output included
- [x] Event 4887 (Issued) located and documented — timestamp and key fields recorded
- [x] Event 4870 (Revoked) located and documented — timestamp and key fields recorded
- [x] Event 4872 (CRL Published) located and documented — timestamp recorded
- [x] Event Viewer filtered view documented — all four events (two 4872, one 4870, one 4887)        confirmed
- [x] Before/after comparison table completed
- [x] All four Part E analysis questions answered
- [x] All three lab report questions answered in complete sentences
- [x] File committed to `labs/week-14/lab-03-audit-logging-configuration.md`
