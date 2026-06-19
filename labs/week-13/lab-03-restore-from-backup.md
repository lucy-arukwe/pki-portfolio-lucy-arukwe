# Lab 03: Restore from Backup Files *(Stretch)*

**Student Name:**  
**Date Completed:**  
**Phase:** 2 | **Week:** 13  
**Submission Path:** `labs/week-13/lab-03-restore-from-backup.md`

---

## Overview

This stretch lab has you restore the CA from the backup files you created in Lab 01 — not from a snapshot. This is a file-based restore: you will import the CA private key from the .p12 file, restore the CA database using `certutil -restoredb`, and verify the CA returns to full operational state. Lab 03 simulates a Scenario 2b or Scenario 3 recovery (Lesson 3) — the procedure that applies when no recent snapshot is available or when hardware must be replaced.

**Requirements before starting:**
- Lab 01 must be complete — you need the C:\CABackup files and your backup password
- Lab 02 must be complete — Lab 03 uses a different starting state than Lab 02

**If you do not have the Lab 01 backup password:** Lab 03 cannot be completed without it. Contact your instructor.

---

## Lab Environment

| Component | Details |
|---|---|
| CA Server | PKI-SRV01 (192.168.10.20) |
| CA Name | CVI Issuing CA 1 |
| Domain Account | CORP\pki.admin |
| Backup Source | C:\CABackup (from Lab 01) |
| Backup Password | The password you used in Lab 01 for certutil -backup -p |

---

## Pre-Lab Setup

### Step 1 — Confirm Lab 01 Backup Files Are Present

The Lab 03 restore requires the backup files from Lab 01 to still be present.

```powershell
Get-ChildItem C:\CABackup -Recurse | Select-Object FullName, Length, LastWriteTime
```

**Confirm these files exist:**

| File | Present? |
|---|---|
| C:\CABackup\<CAName>.p12 | [x] Yes / [ ] No |
| C:\CABackup\DataBase\<CAName>.edb | [x] Yes / [ ] No |

> **If the backup files were removed by the Lab 02 snapshot restore:** The snapshot restored PKI-SRV01 to its Lab 01 post-backup state — the backup files should be present. If they are not, you need to re-run Lab 01 before proceeding.

```

FullName                                  Length  LastWriteTime
--------                                  ------  -------------
C:\CABackup\DataBase                              6/18/2026 5:21:30 PM
C:\CABackup\CVI Issuing CA 1.p12          4631    6/18/2026 5:21:30 PM
C:\CABackup\DataBase\certbkxp.dat         398     6/18/2026 5:21:30 PM
C:\CABackup\DataBase\CVI Issuing CA 1.edb 1052672 6/18/2026 5:21:30 PM
C:\CABackup\DataBase\edb00003.log         1048576 6/18/2026 5:21:30 PM
```

### Step 2 — Confirm Your Backup Password

You will need the password you specified in Lab 01's `certutil -backup -p <password>` command. Do not proceed without it.

**I have the backup password from Lab 01:**
- [x] Yes — password is in hand (do not record it here)
- [ ] No — I need to re-run Lab 01 before continuing

### Step 3 — Take a New Snapshot Before Starting

Before performing any destructive operations, take a fresh snapshot of PKI-SRV01. This gives you a rollback point if the restore procedure encounters unexpected issues.

Shut down PKI-SRV01 cleanly:
```powershell
Stop-Computer -Force
```

Then take the snapshot on your **host machine**. Follow the instructions for your hypervisor:

**VirtualBox:** Machine → Take Snapshot → Name it `Week13-Lab03-PreRestore-<today's date>` → Click OK

**UTM:** Right-click **PKI-SRV01** in the UTM sidebar → **Snapshots...** → click **+** → Name it `Week13-Lab03-PreRestore-<today's date>` → click OK

Then start PKI-SRV01 and log back in as CORP\pki.admin.

**Hypervisor:**
- [x] VirtualBox
- [ ] UTM

**Lab03 pre-restore snapshot taken:**
- [x] Yes — snapshot name recorded:
- [ ] No — reason:

---

## Part A — Simulate the Failure State

To demonstrate a file-based restore, you need the CA to be in a broken state — one that cannot be fixed with a snapshot restore alone. You will delete the CA database, the CA private key from the Windows certificate store, and clear the CertLog folder.

> ⚠️ **This is the point of no return for snapshot-based recovery.** After Part A, the only path back is the file-based restore in Parts B and C. The Lab03 pre-restore snapshot from Step 3 above is your safety net — it uses the Lab03 snapshot, not the Lab01 snapshot.

### Step 1 — Stop the CA Service

```powershell
Stop-Service CertSvc
Get-Service CertSvc
```

**CA service is stopped:**
- [x] Yes — status shows Stopped

### Step 2 — Delete the CA Database Files

```powershell
# Remove all CA database files
Remove-Item "C:\Windows\System32\CertLog\*" -Force -Recurse
dir "C:\Windows\System32\CertLog\"
```

**Expected after deletion:** Empty directory or "File Not Found."

```
Empty directory
```

### Step 3 — Remove the CA Private Key from the Windows Certificate Store

This simulates key material being deleted or unavailable — the more severe failure scenario.

```powershell
# List the CA certificate in the local machine store
certlm.msc
# Navigate to: Certificates (Local Computer) → Personal → Certificates
# Locate the "CVI Issuing CA 1" certificate
# Right-click → Delete
```

Alternatively, from PowerShell (find and remove the CA cert):

```powershell
# Find the CA certificate thumbprint
$caCert = Get-ChildItem Cert:\LocalMachine\My | Where-Object { $_.Subject -like "*CVI Issuing CA*" }
$caCert | Select-Object Subject, Thumbprint

# Remove the certificate (including private key if present)
Remove-Item "Cert:\LocalMachine\My\$($caCert.Thumbprint)"
```

```
(paste the commands and output here)
```

**CA certificate removed from local machine certificate store:**
- [x] Yes
- [ ] Not found — document and continue (the certutil -restorekey step will handle this)

### Step 4 — Attempt to Start the CA and Document the Failure

```powershell
Start-Service CertSvc
Get-Service CertSvc

# Check event log for errors
Get-WinEvent -LogName Application -Source "CertificationAuthority" -MaxEvents 5 |
    Select-Object TimeCreated, Id, Message | Format-List
```

```
Status   Name               DisplayName
------   ----               -----------
Running  CertSvc            Active Directory Certificate Services


PS C:\Windows\system32> Get-WinEvent -LogName Application -MaxEvents 20 | Where-Object {$_.ProviderName -eq "Microsoft-Windows-CertificationAuthority"} | Select-Object TimeCreated, Id, Message | Format-List

TimeCreated : 6/18/2026 9:18:24 PM
Id          : 17
Message     : Active Directory Certificate Services did not start: Unable to initialize the database
              connection for CVI Issuing CA 1.  File not found 0xc8000713 (ESE: -1811 JET_errFileNotFound).

TimeCreated : 6/18/2026 9:15:32 PM
Id          : 38
Message     : Active Directory Certificate Services for CVI Issuing CA 1 was stopped.

TimeCreated : 6/18/2026 9:14:34 PM
Id          : 26
Message     : Active Directory Certificate Services for CVI Issuing CA 1 was started.
              DC=DC01.corp.cvilab.local
```

**Failure state documented:**

```
CA service status: Running (but non-functional — RPC server unavailable, 
certutil -ping and certutil -CRL both failed with error 0x800706ba)

Event IDs observed: Event ID 17 — "Active Directory Certificate Services did not start: Unable 
to initialize the database connection for CVI Issuing CA 1. File not found 
0xc8000713 (ESE: -1811 JET_errFileNotFound)."
Event ID 38 — "Active Directory Certificate Services for CVI Issuing CA 1 
was stopped."
Event ID 26 — "Active Directory Certificate Services for CVI Issuing CA 1 
was started."

Failure description in your own words: The CA database files were deleted from C:\Windows\System32\CertLog\ and the 
CA certificate and private key were removed from the local machine certificate 
store. Although the CertSvc process started, it could not initialize because 
the database was missing. certutil -ping and certutil -CRL both failed with 
RPC_S_SERVER_UNAVAILABLE, confirming the CA was non-functional despite 
showing a Running service status.
```

---

## Part B — Restore the CA Private Key

The first step of a file-based restore is restoring the CA private key from the .p12 backup. Without the private key, the CA service cannot sign certificates or CRLs.

### Step 1 — Run certutil -restorekey

Replace `<YourBackupPassword>` with the password from Lab 01.

```powershell
certutil -restorekey "C:\CABackup\<CAName>.p12" <YourBackupPassword>
```

**Expected output:**
```
Restoring to container: <ContainerName>
CertUtil: -restorekey command completed successfully.
```

> **If you see "Cannot find object or property":** The path to the .p12 file is incorrect. The CA name contains spaces — the path must be in quotes. Use the exact form: `certutil -restorekey "C:\CABackup\CVI Issuing CA 1.p12" <YourBackupPassword>`. Verify the exact filename first: `dir C:\CABackup\*.p12`

> **If you see "The password does not meet the password policy requirements" or "The credentials supplied to the package were not recognized":** The password is incorrect. Re-check your Lab 01 notes. The password is case-sensitive.

> **If you see "The system cannot find the file specified":** The .p12 file does not exist at the specified path. Run `dir C:\CABackup` to confirm the filename and path.

```
Restored keys and certificates for PKI-SRV01.corp.cvilab.local\CVI Issuing CA 1 from C:\CABackup\CVI Issuing CA 1.p12.
CertUtil: -restoreKey command completed successfully.
The CertSvc service may need to be restarted for changes to take effect.
PS C:\Windows\system32>
```

**certutil -restorekey completed successfully:**
- [x] Yes
- [ ] No — error message and resolution:

### Step 2 — Verify the CA Certificate Is Back in the Certificate Store

```powershell
# Confirm the CA certificate and key are now in the local machine store
Get-ChildItem Cert:\LocalMachine\My | Where-Object { $_.Subject -like "*CVI Issuing CA*" } |
    Select-Object Subject, Thumbprint, HasPrivateKey
```

**Expected:** `HasPrivateKey = True` — confirming the key was restored.

```
Subject                                           Thumbprint                               HasPrivateKey
-------                                           ----------                               -------------
CN=CVI Issuing CA 1, DC=corp, DC=cvilab, DC=local 5137A597DE2C3085EC5816C7F11EDC18CFCDBAF8          True
```

**CA certificate is in the local machine store with private key:**
- [x] Yes — HasPrivateKey = True
- [ ] No — describe:

> **If HasPrivateKey shows False despite certutil -restorekey reporting success:** The key was restored to a different CSP container than the CA service expects. Run `certutil -store My` to see all certificates in the personal store and verify the correct CA certificate is listed. If the thumbprint does not match the CA certificate, re-run certutil -restorekey and confirm you are using the correct .p12 file.

---

## Part C — Restore the CA Database

With the private key restored, you now restore the CA database from the certutil backup.

### Step 1 — Run certutil -restoredb

```powershell
certutil -restoredb "C:\CABackup"
```

**Expected output:**
```
Restoring CA database...
Database restored successfully.
CertUtil: -restoredb command completed successfully.
```

> **If you see "The directory is not empty" or "File already exists":** The CertLog directory may have residual files. Clear it first:
> `Remove-Item "C:\Windows\System32\CertLog\*" -Force -Recurse`
> Then re-run certutil -restoredb.

> **If you see "The backup directory does not contain a valid database backup":** The path to the backup is incorrect, or the DataBase subfolder is missing. Verify: `dir C:\CABackup\DataBase\`

```
Restoring database for PKI-SRV01.corp.cvilab.local\CVI Issuing CA 1.
Restoring Database files: 100%
Restoring Log files: 100%
Full database restore for PKI-SRV01.corp.cvilab.local\CVI Issuing CA 1.
Stop and Start Active Directory Certificate Services to complete database restore from C:\CABackup.
CertUtil: -restoreDB command completed successfully.
The CertSvc service may need to be restarted for changes to take effect.
```

**certutil -restoredb completed successfully:**
- [x] Yes
- [ ] No — error message and resolution:

### Step 2 — Verify the Database Files Were Restored

```powershell
dir "C:\Windows\System32\CertLog\"
```

**Expected:** .edb file and log files are present.

```
  Directory: C:\Windows\System32\CertLog


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----         6/18/2026   9:34 PM        1052672 CVI Issuing CA 1.edb
-a----         6/18/2026   9:34 PM        1048576 edb00003.log

```

---

## Part D — Start the CA Service and Verify

### Step 1 — Start the CA Service

```powershell
Start-Service CertSvc
Get-Service CertSvc
```

**Expected:** Status = Running

```
Status   Name               DisplayName
------   ----               -----------
Running  CertSvc            Active Directory Certificate Services

```

> **If the service fails to start with Event ID 100 — "CA certificate not found":** certutil -restorekey was not run before certutil -restoredb, or the restorekey step failed silently. Confirm HasPrivateKey = True in the certificate store (Step 2 above). If not, re-run certutil -restorekey first, then Start-Service CertSvc again.

> **If the service fails to start with Event ID 100 — "CA database could not be opened":** The CertLog directory still has residual files. Run `Remove-Item "C:\Windows\System32\CertLog\*" -Force -Recurse` and re-run certutil -restoredb, then Start-Service CertSvc.

> **If the service starts but certutil -CRL fails with "The system cannot find the path specified":** The CertEnroll folder may be missing or the service account lacks write access. Check: `dir "C:\Windows\System32\CertSrv\CertEnroll\"`. If the folder does not exist, create it: `New-Item -ItemType Directory -Path "C:\Windows\System32\CertSrv\CertEnroll" -Force`. Then retry certutil -CRL.

> **If Event ID 100 shows a CA name mismatch:** The CA name in the restored database does not match the installed CA configuration. Restore the system state backup from Lab 01 (wbadmin start systemstaterecovery) and retry.

**CA service started without errors:**
- [x] Yes — proceed to verification
- [ ] No — error and resolution:

### Step 2 — Run the Full Post-Recovery Verification Checklist

```powershell
# 1. CA responds
certutil -ping

# 2. Publish CRL
certutil -CRL

# 3. Check event log for errors
Get-WinEvent -LogName Application -Source "CertificationAuthority" -MaxEvents 10 |
    Where-Object { $_.LevelDisplayName -eq "Error" } |
    Select-Object TimeCreated, Id, Message | Format-List
```

```

PS C:\Windows\system32> certutil -ping
Connecting to PKI-SRV01.corp.cvilab.local\CVI Issuing CA 1 ...
Server "CVI Issuing CA 1" ICertRequest2 interface is alive (0ms)
CertUtil: -ping command completed successfully.

PS C:\Windows\system32> certutil -CRL
CertUtil: -CRL command completed successfully.

PS C:\Windows\system32> Get-WinEvent -LogName Application -MaxEvents 20 | Where-Object {$_.ProviderName -eq "Microsoft-Windows-CertificationAuthority" -and $_.LevelDisplayName -eq "Error"} | Select-Object TimeCreated, Id, Message | Format-List
"No output"
```

### Step 3 — Confirm Certificate History Is Intact

```powershell
# View issued certificates from the restored database
certutil -view -restrict "Disposition=20" -out "RequestID,SerialNumber,CommonName,NotAfter" | head -20
```

```
PS C:\Windows\system32> certutil -view -restrict "Disposition=20" -out "RequestID,SerialNumber,CommonName,NotAfter" | Select-Object -First 20
Schema:
  Column Name                   Localized Name                Type    MaxLength
  ----------------------------  ----------------------------  ------  ---------
  RequestID                     Issued Request ID             Long    4 -- Indexed
  SerialNumber                  Serial Number                 String  128 -- Indexed
  CommonName                    Issued Common Name            String  8192 -- Indexed
  NotAfter                      Certificate Expiration Date   Date    8 -- Indexed

Row 1:
  Issued Request ID: 0x9
  Serial Number: "4400000009a217b3df4da50401000000000009"
  Issued Common Name: "Svc Autoenroll"
  Certificate Expiration Date: 4/25/2027 7:36 PM

Row 2:
  Issued Request ID: 0xa (10)
  Serial Number: "440000000a0d2ff137199552aa00000000000a"
  Issued Common Name: "CVI-WebServer"
  Certificate Expiration Date: 4/25/2027 7:36 PM
```

**Issued certificate history is present in the restored database:**
- [x] Yes — records are present
- [ ] No — database appears empty (document and explain):

### Step 4 — Confirm CRL Is Accessible

```powershell
certutil -URL http://pki-srv01.corp.cvilab.local/CertEnroll/"CVI Issuing CA 1.crl"
```

Or navigate to the URL in a browser on PKI-SRV01.

```
(paste output or describe browser result)
```

**CRL is accessible at the HTTP CDP:**
- [x] Yes
- [ ] No — describe:

**Recovery timestamp:**

```
June 18, 2026 — approximately 9:35 PM
```

**Estimated time from Part A (simulated failure) to CA fully operational:**

```
Approximately 15–20 minutes
```

---

## Part E — Snapshot vs. File-Based Restore Comparison

Reflect on your experience with both Lab 02 (snapshot restore) and Lab 03 (file-based restore).

### Direct Comparison

| Factor              | Lab 02 — Snapshot Restore      | Lab 03 — File-Based Restore                |
|---------------------|--------------------------------|--------------------------------------------|
| Time to complete    | ~5–10 minutes                  | ~15–20 minutes                             |
| Password required?  | No                             | Yes — backup password required             |
| Data currency       | Point-in-time of snapshot      | Point-in-time of certutil backup           |
| Hardware dependency | Requires same hypervisor/host  | Can restore to any Windows Server          |
| Steps required      | 1 (snapshot restore)           | 4 (restorekey, restoredb, restart, verify) |

Fill in this table based on your direct experience.

---

## Part F — Lab Report

Answer all questions in complete sentences.

**1. Describe the file-based restore procedure you performed in Parts B and C. Walk through each command — certutil -restorekey and certutil -restoredb — and explain what each one does and why the order matters.**

```
The file-based restore required two certutil commands executed in a specific order. The first was certutil -restorekey -p <password> "C:\CABackup\CVI Issuing CA 1.p12", which imported the CA's private key and certificate from the PKCS#12 backup file back into the Windows local machine certificate store.
Without this step, the CA service has no signing material — it cannot issue certificates, sign CRLs, or authenticate itself to Active Directory. The second command was certutil -restoredb "C:\CABackup", which copied the CA database files from the backup folder back into C:\Windows\System32\CertLog\, restoring
the full issuance history. The order matters because the CA service validation at startup checks for both the private key and the database simultaneously. Restoring the database first without the private key would result in the CA service failing to start — it would find the database but have no key to associate with it.
Restoring the key first ensures that when the database is restored and the service starts, all components are present and consistent.
```

**2. In Part B, you restored the CA private key using certutil -restorekey. What would have happened if you had restored the CA database first (certutil -restoredb) without restoring the private key first — specifically, would the CA service have started? Why or why not?**

```
The CA service would not have started successfully, or if it appeared to start, it would have been non-functional — exactly as observed in the failure simulation in Part A. The CA service requires both the database and the private key to initialize. The database tells the CA what
certificates it has issued, but the private key is what allows it to cryptographically sign new certificates and CRLs. Without the private key in the local machine certificate store, the CA service cannot bind to its own certificate at startup. The event log would show an error indicating the
CA certificate could not be found or the private key was inaccessible, and certutil -ping would fail with an RPC unavailable error. The restore must always follow the sequence: key first, database second, then restart the service.
```

**3. Compare your experience with Lab 02 (snapshot restore) and Lab 03 (file-based restore). Which was faster? Which required more steps? Which required the backup password? Which procedure would apply if the CA server hardware had been destroyed and you were restoring to a new machine?**

```
Snapshot restore in Lab 02 was significantly faster — the entire recovery took approximately 5–10 minutes and required only a single operation in VirtualBox with no commands to run inside the VM. File-based restore in Lab 03 took approximately 15–20 minutes and required multiple sequential
steps: stopping the service, running certutil -restorekey, clearing the CertLog directory, running certutil -restoredb, restarting the service, and running through the full verification checklist. Only the file-based restore required the backup password — snapshot restore has no password requirement since
it reverts the entire VM state directly. If the CA server hardware were destroyed and a new machine were needed, only the file-based restore would apply. Snapshots are tied to the hypervisor and the specific VM — they cannot be transferred to new hardware. The file-based restore procedure works on any Windows
Server with the AD CS role installed, making it the only viable option for hardware replacement scenarios.
```

**4. Explain the "environment mismatch" failure mode described in Lesson 3. What would cause certutil -restoredb to succeed but the CA service to fail to start? What would the event log show, and how would you resolve it?**

```
An environment mismatch occurs when the CA database is restored successfully but the configuration of the Windows environment does not match what the database expects. The most common cause is restoring a database to a machine where the CA was reinstalled with a slightly different configuration — a different CA name,
a different key container name, or a different cryptographic provider than what was used when the backup was taken. In this situation, certutil -restoredb completes without errors because it is only copying files, not validating them against the current CA configuration. The CA service then fails to start because when it reads the database,
it finds records that reference a CA name or key container that does not match what is currently registered. The event log would show Event ID 100 with a message indicating a CA name mismatch or that the CA certificate could not be found. The resolution depends on the cause: if the CA name is wrong, the AD CS role must be reinstalled with the
correct name before restoring; if the key container is mismatched, certutil -restorekey must be re-run to ensure the key is in the expected container. This is precisely why the system state backup — which captures the CA's registry configuration — is so valuable: it eliminates environment mismatch by restoring the configuration alongside the database.
```

**5. If this were a production CA with an RTO of 4 hours, would the file-based restore procedure you performed today meet that RTO? What factors could cause it to take longer — or shorter — in a production environment compared to your lab?**

```
The file-based restore performed today took approximately 15–20 minutes from simulated failure to full operational state, which would comfortably meet a 4-hour RTO in isolation. However, several factors in a production environment could significantly extend that time. If the CA server hardware were destroyed, provisioning and configuring a new Windows Server
before the restore could even begin might take hours depending on the organization's infrastructure provisioning process. A production CA database is likely far larger than the lab database — the lab .edb file was just over 1 MB, while a production CA that has issued tens of thousands of certificates could have a database many gigabytes in size, making the restore itself
take considerably longer. Retrieving the backup files from off-site storage or a secure vault, and obtaining the backup password through whatever approval process the organization requires, adds additional time. Factors that could shorten recovery include having a pre-provisioned standby server, storing backups on a network share accessible without physical retrieval, and
having documented runbook procedures so the recovery operator does not need to look up commands. In a well-prepared environment, the 4-hour RTO is achievable; in an unprepared one, the same procedure could easily exceed it.
```

---

## Submission Checklist

- [ ] Logged in as CORP\pki.admin (not a local account)
- [ ] Lab 01 backup files confirmed present in C:\CABackup before starting
- [ ] Backup password available from Lab 01 records
- [ ] Lab03 pre-restore snapshot taken (VirtualBox or UTM) — snapshot name documented
- [ ] Failure state simulated — CA database deleted AND CA private key removed from cert store (both operations required)
- [ ] Failure state documented — CA service status and event log errors recorded
- [ ] certutil -restorekey run successfully — output included
- [ ] CA certificate confirmed back in local machine store with HasPrivateKey = True
- [ ] certutil -restoredb run successfully — output included
- [ ] CA database files confirmed in C:\Windows\System32\CertLog\ after restore
- [ ] CA service started without errors — Get-Service CertSvc shows Running
- [ ] certutil -ping successful after recovery
- [ ] certutil -CRL successful after recovery
- [ ] Event log clean (no CA errors after recovery)
- [ ] Issued certificate history confirmed present in restored database
- [ ] CRL accessible at HTTP CDP
- [ ] Recovery timestamp and estimated duration recorded
- [ ] Snapshot vs. file-based comparison table completed
- [ ] All five lab report questions answered in complete sentences
- [ ] Lab file committed to `labs/week-13/lab-03-restore-from-backup.md`
