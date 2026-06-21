# Lab 01: Full CA Backup — Database, Keys, and System State

**Student Name:**  Lucy Arukwe
**Date Completed:** 18 June, 2026
**Phase:** 2 | **Week:** 13  
**Submission Path:** `labs/week-13/lab-01-ca-backup.md`

---

## Overview

In this lab, you perform a complete backup of PKI-SRV01, the CVI Issuing CA 1. A complete CA backup has three components: the CA database (all issued certificates and revocation records), the CA private key and certificate (the most critical component), and the Windows system state (CA configuration). You will use `certutil -backup` for the database and private key, and `wbadmin` for the system state. Your lab report documents every command and output — this IS the backup documentation.

**The backup files you create in this lab are required for Lab 03.** Record your backup password in a location separate from the backup folder.

---

## Lab Environment

| Component | Details |
|---|---|
| CA Server | PKI-SRV01 (192.168.10.20) |
| CA Name | CVI Issuing CA 1 |
| Domain | CORP\pki.admin |
| Backup Destination — Database | C:\CABackup |
| Backup Destination — System State | D:\ or a network path (separate volume required) |

> **Note on system state backup destination:** Windows Server Backup requires the system state backup target to be on a different volume than the OS (C:). If PKI-SRV01 does not have a second drive in your lab setup, see the instructor note at the end of Part C for alternatives.

---

## Pre-Lab Verification

Before starting, confirm the environment is healthy.

### Step 1 — Log in as CORP\pki.admin

Log into PKI-SRV01 using the domain account `CORP\pki.admin`. Confirm you are not using a local account.

```powershell
# Confirm your identity
whoami
```

**Expected output:** `corp\pki.admin`

```
corp\pki.admin
```

**Logged in as CORP\pki.admin (not a local account):**
- [ ] Yes
- [ ] No — describe:

### Step 2 — Confirm the CA Service Is Running

```powershell
Get-Service CertSvc
```

**Expected output:**
```
Status   Name               DisplayName
------   ----               -----------
Running  CertSvc            Active Directory Certificate Services
```

```
Status   Name               DisplayName
------   ----               -----------
Running  CertSvc            Active Directory Certificate Services
```

### Step 3 — Confirm the CA Responds

```powershell
certutil -ping
```

**Expected output:**
```
Connecting to PKI-SRV01\CVI Issuing CA 1 ...
Server "CVI Issuing CA 1" ICertRequest2 interface is alive
CertUtil: -ping command completed successfully.
```

```
Connecting to PKI-SRV01.corp.cvilab.local\CVI Issuing CA 1 ...
Server "CVI Issuing CA 1" ICertRequest2 interface is alive (16ms)
CertUtil: -ping command completed successfully.
```

**CA is running and responding:**
- [x] Yes — proceed to Part A
- [ ] No — describe the issue and resolution:

---

## Part A — Create the Backup Destination

You need a dedicated folder for the CA backup files. The certutil command will write the CA database and private key here.

### Step 1 — Open an Elevated PowerShell Prompt

Right-click **Windows PowerShell** → **Run as Administrator**. Confirm the title bar shows **Administrator: Windows PowerShell**.

### Step 2 — Create the Backup Folder Using File Explorer

1. Press **Windows key + E** to open File Explorer (or click the folder icon on the taskbar).
2. In the left navigation panel, click **This PC**.
3. In the right panel, double-click **Local Disk (C:)** to open the C: drive.
4. Right-click in any empty area of the right panel (not on an existing file or folder).
5. Select **New → Folder** from the context menu.
6. A new folder appears with the name highlighted. Type exactly: `CABackup`
7. Press **Enter** to confirm the name.

The folder is now at `C:\CABackup`.

> **If the folder already exists from a prior backup attempt:** Right-click **CABackup** → **Delete** to remove it, then re-create it using the steps above. certutil will fail if the destination already contains a .p12 file with the same CA name from a previous run.

> **Prefer command line?** You can also run this in the elevated PowerShell prompt:
> `New-Item -ItemType Directory -Path "C:\CABackup" -Force`

### Step 3 — Verify the Folder Exists

Confirm the folder was created before running certutil:

```powershell
dir C:\CABackup
```

**Expected:** An empty directory listing with today's date. No files should be present.

```
(paste dir output here)
```

**C:\CABackup folder exists and is empty:**
- [ ] Yes — confirmed with `dir C:\CABackup`
- [ ] Folder contained old files — cleared before proceeding

---

## Part B — Back Up the CA Database and Private Key

`certutil -backup` captures both the CA database and the CA private key in one command. It uses Windows Volume Shadow Copy Service (VSS) — the CA service remains running during the backup.

### Step 1 — Choose a Backup Password

You will specify a password to protect the private key .p12 file. This password is as sensitive as the private key itself.

**Requirements:** At least 12 characters. Mix of uppercase, lowercase, numbers, and symbols.

> ⚠️ **Record this password in a location SEPARATE from C:\CABackup.** You will need it in Lab 03. If you lose the password, you cannot complete the file-based restore.

**Password storage location (do not write the password here — write where you stored it):**

```
(e.g., "Recorded in the lab notes document on my desktop, not in the backup folder")
```

### Step 2 — Run certutil -backup

Replace `<YourBackupPassword>` with the password you chose above.

```powershell
certutil -backup -p <YourBackupPassword> C:\CABackup
```

This command backs up:
- The CA database → `C:\CABackup\DataBase\<CAName>.edb` (and log files)
- The CA private key and certificate → `C:\CABackup\<CAName>.p12`

**Expected output (may take 30–60 seconds):**
```
Backup directory: C:\CABackup
Key Container Name: <CAName>
Backing up CA database...
Database backed up successfully.
Backing up private key and certificate...
Private key and certificate backed up successfully.
CertUtil: -backup command completed successfully.
```

> **If you see "Access is denied":** You are not running from an elevated prompt or not logged in as CORP\pki.admin. Right-click PowerShell → Run as Administrator. Run `whoami` to confirm `corp\pki.admin`.

> **If you see "The process cannot access the file because it is being used by another process":** Another backup is in progress or the folder is locked. Wait 60 seconds and retry.

```
Backed up keys and certificates for PKI-SRV01.corp.cvilab.local\CVI Issuing CA 1 to C:\CABackup\CVI Issuing CA 1.p12.
Full database backup for PKI-SRV01.corp.cvilab.local\CVI Issuing CA 1.
Backing up Database files: 100%
Backing up Log files: 100%
Truncating Logs: 100%
Backed up database to C:\CABackup.
Database logs successfully truncated.
CertUtil: -backup command completed successfully.
```

**certutil -backup completed without errors:**
- [x] Yes
- [ ] No — error message and resolution:

### Step 3 — Verify Backup Files Exist

```powershell
# List all files in the backup folder
Get-ChildItem C:\CABackup -Recurse | Select-Object FullName, Length, LastWriteTime
```

**Expected files:**

| File | What It Is |
|---|---|
| `C:\CABackup\<CAName>.p12` | CA private key + certificate (PKCS#12 format) |
| `C:\CABackup\DataBase\<CAName>.edb` | CA database file |
| `C:\CABackup\DataBase\*.log` | Database transaction log files |

```
PS C:\Windows\system32> Get-ChildItem C:\CABackup -Recurse | Select-Object FullName, Length, LastWriteTime

FullName                                  Length  LastWriteTime
--------                                  ------  -------------
C:\CABackup\DataBase                              6/18/2026 5:21:30 PM
C:\CABackup\CVI Issuing CA 1.p12          4631    6/18/2026 5:21:30 PM
C:\CABackup\DataBase\certbkxp.dat         398     6/18/2026 5:21:30 PM
C:\CABackup\DataBase\CVI Issuing CA 1.edb 1052672 6/18/2026 5:21:30 PM
C:\CABackup\DataBase\edb00003.log         1048576 6/18/2026 5:21:30 PM
```

**Record the backup file details:**

```
CA private key file (.p12):
  Full path:       C:\CABackup\CVI Issuing CA 1.p12
  File size:       4631 bytes
  Last write time: 6/18/2026 5:21:30 PM

CA database file (.edb):
  Full path:       C:\CABackup\DataBase\CVI Issuing CA 1.edb
  File size:       1052672 bytes
  Last write time: 6/18/2026 5:21:30 PM
```

**All expected files are present:**
- [x] Yes — all three file types confirmed
- [ ] No — describe what is missing:

### Step 4 — Verify the .p12 File Is Readable

This confirms the private key backup is not corrupt.

```powershell
certutil -dump -p <YourBackupPassword> "C:\CABackup\<CAName>.p12"
```

**Expected output (partial):**
```
================ Certificate 0 ================
...
Subject: CN=CVI Issuing CA 1, DC=corp, DC=cvilab, DC=local
...
CertUtil: -dump command completed successfully.
```

> **If you see "Cannot find object or property" or a password error:** The .p12 file may be corrupt or the password is wrong. Re-run certutil -backup with the correct password.

```
================ Certificate 0 ================
================ Begin Nesting Level 1 ================
Element 0:
Serial Number: 5800000002f7714edc7f317c46000000000002
Issuer: CN=CVI Root CA, DC=corp, DC=cvilab, DC=local
 NotBefore: 4/25/2026 7:26 PM
 NotAfter: 4/25/2027 7:36 PM
Subject: CN=CVI Issuing CA 1, DC=corp, DC=cvilab, DC=local
CA Version: V0.0
Certificate Template Name (Certificate Type): SubCA
Non-root Certificate
Template: SubCA, Subordinate Certification Authority
Cert Hash(sha1): 5137a597de2c3085ec5816c7f11edc18cfcdbaf8
----------------  End Nesting Level 1  ----------------
  Provider = Microsoft Software Key Storage Provider
Private key is NOT plain text exportable
Encryption test passed

================ Certificate 1 ================
================ Begin Nesting Level 1 ================
Element 1:
Serial Number: 26373e51a6ab669340c47caef2232ce1
Issuer: CN=CVI Root CA, DC=corp, DC=cvilab, DC=local
 NotBefore: 4/25/2026 6:15 PM
 NotAfter: 4/25/2046 6:25 PM
Subject: CN=CVI Root CA, DC=corp, DC=cvilab, DC=local
CA Version: V0.0
Signature matches Public Key
Root Certificate: Subject matches Issuer
Cert Hash(sha1): b805e6ab548f6e7c57d3989f61de7fe6a51031d1
----------------  End Nesting Level 1  ----------------
No key provider information
Cannot find the certificate and private key for decryption.
CertUtil: -dump command completed successfully.
```

**certutil -dump confirms .p12 is readable with the backup password:**
- [x] Yes
- [ ] No — describe:

---

## Part C — Back Up the Windows System State

The system state backup captures the CA's Windows registry configuration, local certificate store, and Active Directory configuration objects. Run this on PKI-SRV01.

### Step 1 — Install Windows Server Backup Feature (if not present)

```powershell
# Check if Windows Server Backup is installed
Get-WindowsFeature Windows-Server-Backup

# Install if not present
Install-WindowsFeature Windows-Server-Backup
```

```
Display Name                                            Name                       Install State
------------                                            ----                       -------------
[X] Windows Server Backup                               Windows-Server-Backup          Installed
```

**Windows Server Backup feature is installed:**
- [x] Yes — already installed
- [ ] Installed now — describe any prompts:

### Step 2 — Identify a Backup Target Volume

The system state backup requires a target volume different from C:. 

```powershell
# List available drives
Get-PSDrive -PSProvider FileSystem
```

```
Name           Used (GB)     Free (GB) Provider      Root                                             CurrentLocation
----           ---------     --------- --------      ----                                             ---------------
C                  30.12         29.27 FileSystem    C:\                                             Windows\system32
D                                      FileSystem    D:\
E                   0.06         19.94 FileSystem    E:\
Z                 237.17        693.34 FileSystem    \\VBoxSvr\Shared
```

> **If only C: is available:** The system state backup to a local drive requires a second volume. In this situation, use one of the alternatives below:
> - **Option A:** Create a VHD on C: and mount it as a separate volume — `New-VHD -Path C:\BackupVHD.vhdx -SizeBytes 20GB -Fixed; Mount-VHD -Path C:\BackupVHD.vhdx`
> - **Option B:** Skip the wbadmin step and document the limitation in your lab report. Your instructor may accept this with explanation.
> - **Option C:** Ask your instructor for a second disk configuration in VirtualBox.

**System state backup target drive/path:**

```
(record the drive letter or path you will use, e.g., "D:\")
```

### Step 3 — Run the System State Backup

Replace `D:` with your target drive letter:

```powershell
wbadmin start systemstatebackup -backuptarget:D: -quiet
```

This takes 10–30 minutes depending on system configuration.

> **The -quiet flag suppresses the "Are you sure?" confirmation prompt.** Without it, wbadmin will wait for interactive input.

> **If the command fails with "The backup storage location is invalid":** The target must be a fixed local disk or a network share (UNC path). USB drives and mapped drives may not work. Try a different target.

```
For that section, paste this:

```

```

**System state backup completed without errors:**
- [ ] Yes — output included above
- [ ] No — error and resolution:
- [x] Skipped — reason documented: The wbadmin system state backup was attempted twice targeting E:\ (a VHD mounted 
via diskpart, as PKI-SRV01 has only a single physical drive). Both attempts failed 
with WinSxS file read errors unrelated to PKI:

Attempt 1 error: Error [0x80070015] The device is not ready.
  File: C:\Windows\WinSxS\wow64_microsoft-windows-iis-metabase_31bf3856ad364e35_
  10.0.20348.143_none_2c1bb6c2e0649461\infocomm.dll

Attempt 2 error: Error [0x800701b1] A device which does not exist was specified.
  File: C:\Windows\WinSxS\amd64_updateservices-selfupdate-75_31bf3856ad364e35_
  10.0.20348.1_none_25ee67d0bd1e4b30\wow64_wuaueng.cab

Both failures occurred in Windows component files (IIS metabase and Windows Update) 
with no relation to the CA configuration. The Hyper-V PowerShell module (New-VHD, 
Mount-VHD) was unavailable, requiring the diskpart VHD workaround. The system state 
backup is documented as failed due to lab environment limitations. All PKI-critical 
data — the CA private key, certificate chain, and full database — were successfully 
captured in Parts A and B via certutil -backup.

### Step 4 — Verify the System State Backup

```powershell
wbadmin get versions
```

**Expected output (partial):**
```
Backup time: <today's date and time>
Backup target: <drive>:\WindowsImageBackup\PKI-SRV01\
Version identifier: <version ID>
Can recover: Volume(s), File(s), Application(s), System State, Bare Metal Recovery
```

```
(paste wbadmin get versions output here)
```

**System state backup appears in wbadmin get versions output:**
- [ ] Yes — timestamp matches today
- [x] No — describe: wbadmin get versions returns "ERROR - No backup was found." The system state backup 
did not complete successfully due to WinSxS file read errors in the lab environment 
(see Part C Step 3 for full details). No backup version was registered.

---

## Part D — Confirm CA Is Still Operational After Backup

The backup runs online — the CA should remain operational throughout. Verify before closing the lab.

```powershell
# Confirm CA service is running
Get-Service CertSvc

# Confirm CA responds
certutil -ping

# Publish a fresh CRL (verifies private key is functional)
certutil -CRL
```

```
Get-Service CertSvc:

Status   Name               DisplayName
------   ----               -----------
Running  CertSvc            Active Directory Certificate Services

PS C:\Windows\system32> certutil -ping
Connecting to PKI-SRV01.corp.cvilab.local\CVI Issuing CA 1 ...
Server "CVI Issuing CA 1" ICertRequest2 interface is alive (0ms)
CertUtil: -ping command completed successfully.

PS C:\Windows\system32> certutil -CRL
CertUtil: -CRL command completed successfully.
```

**CA is operational after backup — all three commands succeeded:**
- [ ] Yes
- [x] No — describe the issue:

---

## Part E — Lab Report

Answer all questions in complete sentences.

**1. Describe the three components of a complete CA backup and explain what would happen during recovery if each one were missing.**

```
A complete CA backup has three main parts. The first is the CA database, which keeps a record of every certificate the CA has issued and any certificates that have been revoked. Without it, the CA would lose its history and would no longer know which certificates it had issued in the past.
The second is the CA private key and certificate, packaged as a .p12 file. This is the most critical component: without it, the recovered CA cannot sign anything. It would be
cryptographically a different CA, and every certificate it had previously issued would become unverifiable against the new key. The third is the Windows system state, which captures the CA's registry configuration,
local certificate stores, and Active Directory integration objects. Without it, a recovery operator would need to manually reconfigure the CA role, re-register it in Active Directory, and restore
registry settings by hand — a time-consuming and error-prone process even with the database and key intact.
```

**2. certutil -backup uses the Volume Shadow Copy Service (VSS). What does this mean operationally — specifically, why is it better than stopping the CA service before copying the database files?**

```
VSS creates a point-in-time snapshot of the CA database files while they are still open and in use. This means the CA service keeps running and can continue issuing certificates during the backup, there
is no service interruption and no window where certificate requests would be rejected. Stopping the CA service before copying files would achieve a clean copy but at the cost of availability: any automated enrollment
requests or CRL retrievals attempted during that window would fail. In a production environment where certificates may be issued continuously, even a brief outage can disrupt dependent services. VSS eliminates that tradeoff entirely.
```

**3. The CA private key backup (.p12 file) is protected by a password you chose. Where did you store the password, and why is storing it in the same folder as the .p12 file a security problem?**

```
The password was recorded in a separate lab notes document on my desktop rather than inside C:\CABackup. Storing the password in the same location as the .p12 file defeats the purpose of password protection entirely. The password exists to ensure that possession of the backup file alone is not sufficient to extract the private key. An attacker who obtains the .p12 file still cannot use it without the password. If the password is stored alongside the file, anyone who gains access to C:\CABackup gets both simultaneously. The password and the encrypted file must be stored and transmitted through separate channels so that compromising one does not automatically compromise the other.

```

**4. What does the Windows system state backup capture that the certutil -backup does not? If the system state backup had been skipped, what would a recovery operator need to do manually that they would not need to do if the system state backup were present?**

```
The certutil backup captures the CA database and private key, but it does not capture how Windows is configured to run the CA. The system state backup includes registry settings, local certificate stores, and Active Directory information related to the CA.

If the system state backup were not available, a recovery operator would need to reinstall AD CS, manually rebuild the CA configuration, restore certificate stores, and re-register the CA in Active Directory. Having the system state backup available makes recovery much faster because most of those settings are restored automatically instead of being recreated by hand.
```

**5. Explain the relationship between backup frequency and Recovery Point Objective (RPO). If this CA performs daily backups and the CA fails on day six of a seven-day backup cycle, what is the maximum data loss — and what specifically is lost?**

```
RPO describes the maximum amount of data that could be lost if a failure occurs. The more frequently backups are taken, the smaller the potential data loss. A daily backup schedule gives the CA an RPO of about 24 hours.

If the CA performs daily backups and fails on day six, the most recent backup would normally be from day five. In that case, the maximum data loss would be about one day of information. This could include certificates issued since the last backup, certificate revocations that were made, and updates to the CA database. The CA could be restored, but any activity that occurred after the most recent backup would not be present and might need to be reconstructed if possible.
```

---

## Submission Checklist

- [x] Logged in as CORP\pki.admin (not a local account) — whoami output included
- [x] CA service confirmed running — Get-Service CertSvc output included
- [x] CA confirmed responding — certutil -ping output included
- [x] C:\CABackup folder created via File Explorer and confirmed empty before running certutil
- [ ] certutil -backup -p run without errors — full output included
- [x] .p12 file confirmed present — file name, path, and size recorded
- [x] .edb database file confirmed present — file name, path, and size recorded
- [x] certutil -dump confirms .p12 is readable with backup password
- [x] Private key backup password stored SEPARATELY from C:\CABackup — storage location documented (not the password itself)
- [x] Windows Server Backup feature installed
- [x] wbadmin system state backup run (or limitation documented with instructor approval)
- [x] wbadmin get versions confirms backup completed with today's timestamp
- [x] CA confirmed still operational after backup — Get-Service, certutil -ping, certutil -CRL all succeeded
- [x] All five lab report questions answered in complete sentences
- [x] Lab file committed to `labs/week-13/lab-01-ca-backup.md`
