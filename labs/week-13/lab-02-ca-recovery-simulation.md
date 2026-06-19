# Lab 02: CA Recovery Simulation — Snapshot Restore

**Student Name:**  
**Date Completed:**  
**Phase:** 2 | **Week:** 13  
**Submission Path:** `labs/week-13/lab-02-ca-recovery-simulation.md`

---

## Overview

In this lab, you simulate a CA failure and recover from it using a VirtualBox snapshot. You will take a snapshot of PKI-SRV01 after confirming the Lab 01 backup is in place, perform a destructive operation that takes the CA offline, and then restore from the snapshot to bring it back to full operational state. This lab demonstrates Scenario 2a from Lesson 3: OS-level failure with a recent snapshot available.

**Lab 01 must be complete before starting this lab.** The Lab 01 backup files must be present in C:\CABackup on PKI-SRV01.

---

## Lab Environment

| Component | Details |
|---|---|
| CA Server | PKI-SRV01 (192.168.10.20) |
| CA Name | CVI Issuing CA 1 |
| Domain Account | CORP\pki.admin |
| Snapshot Tool | VirtualBox (on your host machine) |
| Backup Location | C:\CABackup (from Lab 01) |

---

## Pre-Lab Verification

### Step 1 — Confirm Lab 01 Backup Is Present

Log into PKI-SRV01 as CORP\pki.admin. From an elevated PowerShell prompt:

```powershell
# Confirm backup files exist from Lab 01
Get-ChildItem C:\CABackup -Recurse | Select-Object FullName, Length, LastWriteTime
```

**Expected:** .p12 file and DataBase folder with .edb file from Lab 01.

```
(paste output here)
```

**Lab 01 backup files are present in C:\CABackup:**
- [ ] Yes — proceed to Part A
- [ ] No — Lab 01 must be completed before this lab can proceed

### Step 2 — Confirm CA Is Fully Operational

```powershell
certutil -ping
certutil -CRL
```

```
(paste both outputs here)
```

**CA is operational before starting the simulation:**
- [ ] Yes — proceed to Part A
- [ ] No — resolve any CA issues before continuing

---

## Part A — Take a Pre-Failure Snapshot

You will take a VirtualBox snapshot of PKI-SRV01 while it is in its known-good state. This is the snapshot you will restore from in Part C.

### Step 1 — Note the Pre-Snapshot CA State

Before taking the snapshot, record the current CA state. These values will be your baseline for verification after recovery.

```powershell
# Record CRL publication status
certutil -CRL

# Record CA certificate thumbprint
certutil -store My | findstr /i "Thumbprint\|CVI Issuing"

# Record the most recent certificate issued (for verification after recovery)
certutil -view -restrict "Disposition=20" -out "RequestID,SerialNumber,CommonName,NotAfter" | Select-Object -Last 10
```

```
(paste all three outputs here)
```

**Pre-snapshot CA state — record for comparison after recovery:**

```
CRL publication: successful / failed (circle one)
CA certificate thumbprint (first 8 chars): 
Most recent issued certificate RequestID:
```

### Step 2 — Shut Down PKI-SRV01 Cleanly

You must shut down the VM before taking a snapshot to ensure a consistent state.

From PKI-SRV01:
```powershell
Stop-Computer -Force
```

Wait for the VM to fully power off before proceeding.

**PKI-SRV01 has powered off completely:**
- [ ] Yes
- [ ] No — describe:

### Step 3 — Take the Snapshot

On your **host machine** (not inside the VM). Follow the instructions for your hypervisor.

---

**If you are using VirtualBox:**

1. Open **VirtualBox Manager**
2. Select **PKI-SRV01** in the left panel
3. Click **Machine → Take Snapshot** (or press Ctrl+Shift+S)
4. Name the snapshot: `Week13-Lab02-PreFailure-<today's date>`
5. Add a description: `Known good state after Lab 01 backup. CA database, private key, and system state backed up to C:\CABackup.`
6. Click **OK**

> **Alternative — live snapshot:** VirtualBox supports snapshots while the VM is running. Use Machine → Take Snapshot with the VM running if preferred. The snapshot will include VM memory state.

---

**If you are using UTM (macOS):**

1. Open **UTM**
2. Make sure PKI-SRV01 is powered off (the VM should show as stopped in the sidebar)
3. Right-click **PKI-SRV01** in the UTM sidebar → select **Snapshots...**
4. In the Snapshots window, click the **+** button (bottom left)
5. Name the snapshot: `Week13-Lab02-PreFailure-<today's date>`
6. Click **OK** or press Enter

> **UTM snapshot note:** UTM snapshots are available for QEMU-based VMs only. If your PKI-SRV01 was created using the Apple Virtualization framework (check UTM settings → System → Architecture), snapshots may not be available. In that case, use a full VM clone as an alternative — contact your instructor.

---

**Snapshot name used:**

```
(record exact snapshot name — you will need to find it in Part C)
```

**Hypervisor:**
- [ ] VirtualBox
- [ ] UTM

**Snapshot taken successfully:**
- [ ] Yes — snapshot appears in the snapshot list for PKI-SRV01
- [ ] No — describe the issue:

### Step 4 — Start PKI-SRV01

Start the VM and log back in as CORP\pki.admin before proceeding to Part B.

```powershell
# Confirm CA is running after restart
certutil -ping
```

```
(paste certutil -ping output here)
```

---

## Part B — The Destructive Operation (Simulating Failure)

You will now perform a destructive operation to simulate a CA failure. Choose one of the two options below — Option 1 is recommended. Read both options before choosing.

> ⚠️ **This step intentionally breaks the CA.** The recovery in Part C will restore it. Do not attempt to repair the CA before completing Part C.

### Choose Your Failure Scenario

**Option 1 — Delete the CA database files (Recommended)**

This simulates corruption or accidental deletion of the CA database — a Scenario 2 failure where the OS is running but the CA cannot start.

From an elevated PowerShell prompt on PKI-SRV01:

```powershell
# Stop the CA service first (required to unlock the database files)
Stop-Service CertSvc

# Delete the CA database files
Remove-Item "C:\Windows\System32\CertLog\*.edb" -Force
Remove-Item "C:\Windows\System32\CertLog\*.log" -Force

# Confirm files are deleted
dir "C:\Windows\System32\CertLog\"

# Attempt to start the CA service — it should fail
Start-Service CertSvc
```

---

**Option 2 — Corrupt the CA database via registry modification**

This simulates a misconfiguration that prevents the CA from locating its database — producing a different error but the same recovery requirement.

```powershell
# Stop the CA service
Stop-Service CertSvc

# Rename the database directory to simulate it being inaccessible
Rename-Item "C:\Windows\System32\CertLog" "C:\Windows\System32\CertLog_BROKEN"

# Attempt to start the CA — it should fail
Start-Service CertSvc
```

> If using Option 2, rename the folder back BEFORE attempting certutil -recover or any repair. The snapshot restore in Part C will undo this automatically.

---

**Which option did you choose?**
- [ ] Option 1 — CA database files deleted
- [ ] Option 2 — CertLog folder renamed

**Record the failure state:**

```powershell
# What does the CA service status show?
Get-Service CertSvc

# What does the event log show?
Get-WinEvent -LogName Application -Source "CertificationAuthority" -MaxEvents 5 |
    Select-Object TimeCreated, Id, Message | Format-List
```

```
(paste Get-Service output here)
```

```
(paste event log output here)
```

**Document the failure state in your own words:**

```
CA service status:
Event log errors observed (Event IDs and messages):
What you believe is causing the CA failure:
```

**CA service is in a failed state and the failure is documented:**
- [ ] Yes — proceed to Part C
- [ ] No — CA service started successfully (choose a more destructive option above)

---

## Part C — Recovery: Snapshot Restore

You will now restore PKI-SRV01 from the snapshot taken in Part A.

### Step 1 — Shut Down PKI-SRV01

The snapshot restore requires the VM to be powered off.

From PKI-SRV01 (if you can still log in):
```powershell
Stop-Computer -Force
```

If the VM is unresponsive, force power off from your hypervisor:
- **VirtualBox:** Machine → ACPI Shutdown, or as a last resort, Machine → Power Off
- **UTM:** Right-click the VM in the sidebar → Stop, or close the VM window

**PKI-SRV01 is powered off:**
- [ ] Yes
- [ ] Forced power off used — describe:

### Step 2 — Restore the Snapshot

On your **host machine**. Follow the instructions for your hypervisor.

---

**If you are using VirtualBox:**

1. Open **VirtualBox Manager**
2. Select **PKI-SRV01**
3. Click the **Snapshots** tab (clock icon in the right panel, or View → Snapshots)
4. Locate your `Week13-Lab02-PreFailure-<date>` snapshot
5. Right-click → **Restore Snapshot**
6. When prompted: **Do NOT check "Create a snapshot of the current machine state"** — this is not necessary for a recovery simulation
7. Click **Restore**

> Typically 30–60 seconds. The VirtualBox status bar shows progress.

---

**If you are using UTM:**

1. Open **UTM**
2. Make sure PKI-SRV01 is powered off
3. Right-click **PKI-SRV01** in the UTM sidebar → select **Snapshots...**
4. Locate your `Week13-Lab02-PreFailure-<date>` snapshot in the list
5. Select it and click the **restore icon** (the curved arrow / back arrow button at the bottom of the snapshot list)
6. When prompted to confirm, click **Restore**
7. UTM will restore the VM to the snapshot state — this takes 15–60 seconds

> **UTM restore note:** After the restore, the VM will be in a powered-off state. You will need to start it manually from the UTM sidebar (click the Play button).

---

**Snapshot restore completed without errors:**
- [ ] Yes
- [ ] No — describe the error:

### Step 3 — Start PKI-SRV01 and Log In

Start the VM and log in as CORP\pki.admin.

**VM started and login successful:**
- [ ] Yes
- [ ] No — describe:

---

## Part D — Post-Recovery Verification

Recovery is not complete until every item on the verification checklist passes.

### Step 1 — Confirm CA Service Is Running

```powershell
Get-Service CertSvc
```

**Expected:** Status = Running

```
(paste output here)
```

**CA service is running:**
- [ ] Yes
- [ ] No — attempt Start-Service CertSvc and document result:

### Step 2 — Confirm CA Responds

```powershell
certutil -ping
```

**Expected:** "Server 'CVI Issuing CA 1' ICertRequest2 interface is alive"

```
(paste output here)
```

### Step 3 — Publish a New CRL

```powershell
certutil -CRL
```

**Expected:** "CRL published successfully. CertUtil: -CRL command completed successfully."

```
(paste output here)
```

> **If certutil -CRL fails with "Access is denied":** Confirm you are running from an elevated prompt as CORP\pki.admin.

**CRL published successfully:**
- [ ] Yes
- [ ] No — describe:

### Step 4 — Confirm CRL Is Accessible at the Distribution Point

```powershell
# Find the CDP URL from a certificate
certutil -dump revoked.cer | findstr "http"
# If no certificate file is available, check the CertEnroll folder directly:
certutil -URL http://pki-srv01.corp.cvilab.local/CertEnroll/"CVI Issuing CA 1.crl"
```

Alternatively, navigate to the CDP URL in a browser on PKI-SRV01:
`http://pki-srv01.corp.cvilab.local/CertEnroll/`

**CRL is accessible at the HTTP distribution point:**
- [ ] Yes
- [ ] No — describe:

### Step 5 — Verify Database Is Intact

Confirm the CA database contains its issuance history.

```powershell
# Count issued certificates in the restored database
certutil -view -restrict "Disposition=20" -out "RequestID,CommonName" | Measure-Object -Line
```

```
(paste output here)
```

**Issued certificate count matches what you recorded before the failure:**
- [ ] Yes
- [ ] Approximately — slight difference, explain:
- [ ] No — describe:

### Step 6 — Confirm Event Log Is Clean

```powershell
Get-WinEvent -LogName Application -Source "CertificationAuthority" -MaxEvents 10 |
    Where-Object { $_.LevelDisplayName -eq "Error" } |
    Select-Object TimeCreated, Id, Message | Format-List
```

```
(paste output — or "No errors found")
```

**Event log shows no errors from CertificationAuthority after recovery:**
- [ ] Yes — no errors
- [ ] Errors found — document and explain:

### Step 7 — Record Recovery Completion

**Recovery completed at (date and time):**

```
(record here)
```

**Time from snapshot restore initiation to CA fully operational:**

```
(estimate in minutes)
```

---

## Part E — Lab Report

Answer all questions in complete sentences.

**1. Describe the failure state you simulated in Part B. What did Get-Service CertSvc show, and what Event IDs appeared in the event log? What would a real-world CA administrator see if they encountered this failure?**

```
(your answer here)
```

**2. Walk through the snapshot restore procedure step by step. What did VirtualBox do during the restore — and how did the restored VM state compare to the failure state you left it in?**

```
(your answer here)
```

**3. Walk through the post-recovery verification checklist. Which step confirmed that the CA was fully operational — not just running, but functional? Explain what each verification step tests and why it is not sufficient to just check that the CA service is running.**

```
(your answer here)
```

**4. The snapshot you restored from was taken immediately after Lab 01. If this were a production environment where the last snapshot was taken 72 hours ago, what operational data would be lost in this recovery — and why does that data loss matter to the organization?**

```
(your answer here)
```

**5. Compare snapshot restore to file-based restore (which you will perform in Lab 03). Based on what you experienced today, what is the primary advantage of snapshot restore — and in what failure scenario would snapshot restore NOT be available as an option?**

```
(your answer here)
```

---

## Submission Checklist

- [ ] Logged in as CORP\pki.admin — confirmed with whoami
- [ ] Lab 01 backup files confirmed present in C:\CABackup before starting
- [ ] Pre-failure CA state recorded (CRL status, certificate thumbprint, last issued RequestID)
- [ ] PKI-SRV01 shut down cleanly before snapshot
- [ ] Snapshot taken — name and description recorded
- [ ] Destructive operation performed — failure state documented (service status + event log)
- [ ] Option selected (delete database files OR rename CertLog folder) documented
- [ ] PKI-SRV01 powered off before snapshot restore
- [ ] Snapshot restore completed (VirtualBox or UTM) — no errors
- [ ] Post-recovery verification: CA service running
- [ ] Post-recovery verification: certutil -ping successful
- [ ] Post-recovery verification: certutil -CRL successful
- [ ] Post-recovery verification: CRL accessible at HTTP CDP
- [ ] Post-recovery verification: database certificate count matches pre-failure baseline
- [ ] Post-recovery verification: event log clean (no CA errors after recovery)
- [ ] Recovery time recorded
- [ ] All five lab report questions answered in complete sentences
- [ ] Lab file committed to `labs/week-13/lab-02-ca-recovery-simulation.md`
