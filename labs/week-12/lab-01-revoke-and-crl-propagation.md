# Lab 01: Revoke a Certificate and Observe CRL Propagation

**Student Name:** Lucy Arukwe

**Date Completed:** 30/05/2026 

**Phase:** 2 |**Week:** 12  

**Submission Path:** `labs/week-12/lab-01-revoke-and-crl-propagation.md`

---

## Pre-Lab Verification

If you can log into PKI-SRV01 as CORP\pki.admin, you are communicating with DC01 and the environment is ready. Proceed to Part A.

---

## Part A — Choose Your Certificate

You will revoke one certificate issued during Weeks 10 or 11. The choice of certificate matters — one is reserved for Week 15 and must not be revoked.

### Step 1 — Open the CA Console

1. Press **Win + R**, type `certsrv.msc`, and press **Enter**
2. The Certification Authority console opens
3. In the left pane, expand **CVI Issuing CA 1** → click **Issued Certificates**

### Step 2 — Identify a Certificate to Revoke

1. Browse the list to locate certificates issued in Weeks 10 or 11
2. Right-click the certificate you plan to revoke → **Properties**
3. Note the Common Name, Serial Number, and Certificate Template name
4. Close Properties — do not revoke yet

> **⚠️ Critical — Do NOT select svc.autoenroll:** The `svc.autoenroll` service account certificate is required for Week 15 autoenrollment testing. Revoking it now will break that lab environment. If you accidentally revoke it and the reason code was NOT Key Compromise, you can unrevoce it: go to **Revoked Certificates** → right-click the certificate → **Unrevoke**.

**Recommended revocation candidates (in order of preference):**

| Candidate | Template | Why It's a Good Choice |
|---|---|---|
| Code signing cert issued to CORP\pki.admin | CVI-CodeSigning | Best choice — no forward dependency in any remaining week |
| TLS certificate from Week 10 | CVI-WebServer | Acceptable |
| Any other issued cert that is not svc.autoenroll | Any | Acceptable |

### Step 3 — Record the Certificate Details

1. From an elevated PowerShell prompt, retrieve the serial number of your chosen certificate:

```powershell
certutil -view -restrict "CommonName=<cn-of-your-cert>" -out "RequestID,SerialNumber,CommonName,NotAfter"
```

**Certificate selected for revocation:**

```
Common Name / Subject: CN=PKI Admin
```

**Template name:**

```
CVI-CodeSigning
```

**Serial number:**

```
certutil -view -restrict "CommonName=PKI Admin" -out "RequestID,SerialNumber,CommonName,NotAfter"

Row 3:

Issued Request ID: 0x7
Serial Number: "4400000007172a43e46a06421e000000000007"
Issued Common Name: "PKI Admin"
Certificate Expiration Date: 4/25/2027 7:36 PM
```

**Why you selected this certificate (and confirmed it is not svc.autoenroll):**

```
I selected the CVI-CodeSigning certificate issued to the pki.admin account because it had no dependency on future lab activities.
```

---

## Part B — Revoke the Certificate

### Step 1 — Initiate the Revocation

1. In certsrv.msc → **Issued Certificates**, locate your chosen certificate in the list
2. Right-click the individual certificate row → **All Tasks → Revoke Certificate**

> **If the option is greyed out:** You right-clicked the Issued Certificates folder rather than an individual certificate row. Click the folder to open the list, then right-click a specific certificate row. Also confirm you are logged in as CORP\pki.admin, not a local account.

### Step 2 — Choose a Revocation Reason Code

1. The Revoke Certificate dialog opens with a dropdown for the reason code
2. Review the reason codes and choose the most appropriate one for your scenario:

| Reason Code | When to Use | Urgency | Reversible? |
|---|---|---|---|
| Key Compromise | Private key was exposed or stolen | Immediate — publish CRL now | No |
| CA Compromise | CA private key was exposed | Immediate + CA rebuild required | No |
| Affiliation Changed | Employee left or changed roles | Same-day | Yes |
| Superseded | Replacing with a new certificate | Next scheduled CRL cycle | Yes |
| Cessation of Operation | Service or system decommissioned | Next scheduled CRL cycle | Yes |

> **Why reason codes matter:** Key Compromise is the only code that requires immediate CRL publication regardless of the normal schedule. It also cannot be reversed — once a certificate is revoked for Key Compromise, it cannot be unrevoked. All other reason codes can be reversed from the Revoked Certificates view if revoked by mistake (except CA Compromise).

3. Select your reason code from the dropdown

### Step 3 — Confirm the Revocation

1. Click **Yes** to confirm
2. The certificate immediately moves from Issued Certificates to **Revoked Certificates**

### Step 4 — Record the Revocation Details

1. Click **Revoked Certificates** in the left pane
2. Locate your certificate and note the timestamp shown in the **Revocation Date** column

**Revocation reason code selected:**

| Reason Code            | Selected? |
|------------------------|-----------|
| Key Compromise         | Yes       |
| CA Compromise          | No        |
| Affiliation Changed    | No        |
| Superseded             | No        |
| Cessation of Operation | No        |

**Why you chose this reason code:**

```
I selected Key Compromise because a compromised code signing private key represents a serious security risk.
If an attacker obtained the key, they could sign malicious scripts or software that would appear legitimate and trusted.
Key Compromise requires immediate CRL publication and cannot be reversed, making it a useful scenario for demonstrating the complete certificate revocation workflow.
```

**Certificate appears in Revoked Certificates view:**
- [x] Yes
- [ ] No — describe:

**Timestamp of revocation (from Revoked Certificates view):**

```
| Setting                   | Value             |
| ------------------------- | ----------------- |
| Request ID                | 7                 |
| Certificate Template      | CVI-CodeSigning   |
| Revocation Reason         | Key Compromise    |
| Revocation Date           | 5/30/2026 7:07 AM |
| Effective Revocation Date | 5/30/2026 7:06 AM |

```

---

## Part C — Publish the CRL

After marking the certificate revoked, the revocation is internal to the CA. Relying parties have no way to know about it until a new CRL is published and distributed. This step makes the revocation visible to the outside world.

### Step 1 — Publish a New CRL from the CA Console

1. In certsrv.msc, right-click **CVI Issuing CA 1** — the CA name itself in the left pane, not a subfolder
2. Select **All Tasks → Publish**
3. In the Publish CRL dialog, select **New CRL** → click **OK**

> **Why "New CRL" and not "Delta CRL":** A New CRL is a complete, standalone revocation list. A Delta CRL only contains changes since the last Base CRL, so relying parties must have the Base CRL to use it. For a fresh lab environment where you want the revocation to be immediately verifiable without dependencies, publish a New CRL.

### Step 2 — Confirm Publication from PowerShell

1. In your elevated PowerShell prompt, run:

```powershell
certutil -CRL
```

Expected output:
```
CRL published successfully.
CertUtil: -CRL command completed successfully.
```

> **If you see "Access is denied":** You are not running from an elevated prompt, or you are not logged in as CORP\pki.admin. Right-click PowerShell → **Run as Administrator**. Confirm the session is CORP\pki.admin (not a local account) with `whoami`.

```
PS C:\Windows\system32> certutil -CRL

CertUtil: -CRL command completed successfully.
```

**CRL published without errors:**
- [x] Yes
- [ ] No — describe the error:

---

## Part D — Locate and Inspect the CRL

### Step 1 — Export the Revoked Certificate

1. In certsrv.msc → **Revoked Certificates**, right-click your certificate → **Export**
2. Save it as `revoked.cer` — you will use this file in Parts D and E

### Step 2 — Find the CRL Distribution Point URL in the Certificate

1. Run the following to extract the CDP URL from the certificate:

```powershell
certutil -dump revoked.cer | findstr "CDP http ldap"
```

The CDP URL in your environment will typically be in this format:
```
http://pki-srv01.corp.cvilab.local/CertEnroll/CVI Issuing CA 1.crl
```

There may also be an LDAP CDP entry for Active Directory-based distribution.

**CRL Distribution Point URL found in the certificate:**

`Only LDAP url was present`
```
URL=ldap:///CN=CVI Issuing CA 1,CN=AIA,CN=Public Key Services,CN=Services,CN=Configuration,DC=corp,DC=cvilab,DC=local?certificateRevocationList?base?objectClass=cRLDistributionPoint
```

### Step 3 — Inspect the CRL on Disk

1. Dump the CRL and examine the key fields:

```powershell
certutil -dump "C:\Windows\System32\CertSrv\CertEnroll\CVI Issuing CA 1.crl"
```

> You can also download it via the HTTP CDP URL and inspect the downloaded file:
> ```powershell
> Invoke-WebRequest -Uri "<CDP-URL>" -OutFile downloaded.crl
> certutil -dump downloaded.crl
> ```

**Record the key fields from the CRL output:**

```
ThisUpdate: 5/30/2026 7:06 AM
```

```
NextUpdate: 6/6/2026 7:26 PM
```

> **ThisUpdate** is when this CRL was published — it should closely match the time you ran `certutil -CRL` in Part C. **NextUpdate** is when this CRL expires. Relying parties must download a fresh CRL before this time. With default settings, NextUpdate will be approximately one week from ThisUpdate.

### Step 4 — Confirm Your Revoked Certificate Appears in the CRL

1. Search the CRL for your certificate's serial number:

```powershell
certutil -dump "C:\Windows\System32\CertSrv\CertEnroll\CVI Issuing CA 1.crl" | findstr /i "<your-serial-number>"
```

```
Serial Number: 4400000007172a43e46a06421e000000000007
Revocation Date: 5/30/2026 7:06 AM

CRL Reason Code

Key Compromise (1)
```

> **If the serial number does not appear:** You most likely published the CRL before completing the revocation, or you are looking at the wrong CRL file. Confirm the revocation appears in the Revoked Certificates view in certsrv.msc, then republish with `certutil -CRL`. Use the CDP URL from the certificate itself to confirm you are inspecting the correct CRL file.

**Your revoked certificate's serial number appears in the CRL:**
- [x] Yes
- [ ] No — describe what you see and what you suspect:

---

## Part E — Verify Revocation Status

### Step 1 — Run Certificate Verification

1. Run a full certificate verification against the revoked certificate:

```powershell
certutil -verify revoked.cer
```

### Step 2 — Clear the CRL Cache If Needed

1. If verification still shows the certificate as valid, the local CRL cache may be serving a stale copy. Clear it and retry:

```powershell
certutil -urlcache CRL delete
certutil -verify revoked.cer
```

**Expected output when revocation is confirmed:**
```
ERROR: Verifying leaf certificate revocation status returned The certificate is revoked.
0x80092010 (-2146885616 CRYPT_E_REVOKED)
CertUtil: -verify command failed. 0x80092010 (-2146885616 CRYPT_E_REVOKED)
```

> **If you see CRYPT_E_REVOCATION_OFFLINE instead:** The CRL is not accessible at the CDP URL — check network connectivity within the lab environment and confirm the CRL file is in `C:\Windows\System32\CertSrv\CertEnroll\`. Either output is acceptable for this lab if you include a written explanation of what you observed.

```
The certificate is revoked.
0x80092010 (-2146885616 CRYPT_E_REVOKED)

Certificate is REVOKED
Leaf certificate is REVOKED (Reason=1)

CertUtil: -verify command completed successfully.
```

**certutil -verify reports the certificate as revoked:**
- [x] Yes — paste the relevant error text:
- [ ] No — describe what you see and what you tried:

---

## Part F — Lab Report

Answer all questions in your own words. Write in complete sentences.

**1. Walk through the two-step revocation workflow you performed. What happens at each step — and why is the CRL publication step not optional even when the certificate has already been revoked in the CA?**

```
The revocation process consisted of two separate steps. First, the certificate was revoked in the CA console, which marked the certificate as no longer trusted within the CA database. However, this information remained internal to the CA and was not yet visible to relying parties.
The second step was publishing a new CRL. Publishing the CRL distributed the updated revocation information so systems validating certificates could learn that the certificate had been revoked. Without CRL publication, relying parties could continue trusting the certificate because
they would have no way to retrieve the updated revocation status.
```

**2. You chose a specific revocation reason code. What would have changed operationally if you had chosen Key Compromise instead — particularly regarding CRL publication timing and whether the revocation could be reversed?**

```
I selected Key Compromise for this lab. This reason code represents the most urgent revocation scenario because it indicates that the private key may have been exposed or stolen. Operationally, Key Compromise requires immediate CRL publication rather than waiting for the next scheduled publication cycle.
Unlike many other revocation reasons, a certificate revoked for Key Compromise cannot be unrevoked because trust in the private key has been permanently lost.
```

**3. What does the NextUpdate field in the CRL tell a relying party? If a relying party tries to verify your revoked certificate after NextUpdate has passed and no new CRL has been published, what happens?**

```
The NextUpdate field tells relying parties when the current CRL expires and when a newer CRL should be obtained. If a relying party attempts to validate a certificate after the NextUpdate time has passed and no newer CRL is available, the revocation information becomes stale.
Depending on application settings and policy requirements, certificate validation may fail or the certificate status may be considered unknown.
```

**4. A relying party in a soft-fail environment had already cached the CRL before you published the updated one in Part C. Would they know the certificate is revoked? Why or why not — and what mechanism exists to reduce this propagation lag?**

```
Not immediately. If a relying party had already cached an older CRL before the updated CRL was published, it would continue using the older revocation information until the cache expired or a newer CRL was downloaded. As a result, the relying party could temporarily continue trusting the revoked certificate.
Delta CRLs and shorter publication intervals can reduce this delay by distributing revocation updates more quickly.
```

**5. What is one thing you would do differently or check additionally in a production environment that you did not need to do in this lab?**

```
In a production environment, I would verify that all CRL Distribution Points and any OCSP responders were functioning correctly after publishing the revocation.
I would also validate that multiple relying-party systems could retrieve the updated revocation information and confirm that monitoring and alerting systems detected the revocation event.
These additional checks help ensure that revocation information is successfully distributed throughout the environment and not only within the CA itself.
```

---

## Submission Checklist

- [x] Logged in as CORP\pki.admin (not a local account)
- [x] Certificate selected — NOT svc.autoenroll — serial number documented
- [x] Revocation reason code chosen with written rationale
- [x] Certificate confirmed in Revoked Certificates view with timestamp
- [x] CRL published — certutil -CRL output shows "CRL published successfully"
- [x] CRL Distribution Point URL identified from the certificate
- [x] CRL inspected: ThisUpdate and NextUpdate recorded
- [x] Revoked serial number confirmed present in the CRL
- [x] certutil -verify shows CRYPT_E_REVOKED (0x80092010) or CRYPT_E_REVOCATION_OFFLINE with explanation
- [x] All five lab report questions answered in complete sentences
- [x] Lab file committed to `labs/week-12/lab-01-revoke-and-crl-propagation.md`
