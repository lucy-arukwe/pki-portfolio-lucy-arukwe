# Lab 01: AD CS Event Log Navigation

**Student Name:**  Lucy Arukwe

**Date Completed:**  July 1, 2026

**Phase:** 2 | **Week:** 14  

**Submission Path:** `labs/week-14/lab-01-event-log-navigation.md`

---

## Overview

In this lab, you navigate the two data sources that record what your CA has been doing: the Windows Application event log and the CA certificate database. Both contain a record of the operations you performed across Weeks 10 through 13 — certificate issuance, revocation, CRL publication, and CA service lifecycle events.

This lab has three parts. Part A navigates the Application event log on PKI-SRV01. Part B queries the CA certificate database using certutil -view. Part C asks you to connect both sources to reconstruct an operational picture of the CA.

**No new CA operations are required.** The data you need already exists from prior lab work.

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

### Step 1 — Confirm Login

```powershell
whoami
```

**Expected:** `corp\pki.admin`

```
corp\pki.admin
```

### Step 2 — Confirm CA Is Running

```powershell
Get-Service CertSvc
certutil -ping
```

```
Status   Name               DisplayName
------   ----               -----------
Running  CertSvc            Active Directory Certificate Services

Connecting to PKI-SRV01.corp.cvilab.local\CVI Issuing CA 1 ...
Server "CVI Issuing CA 1" ICertRequest2 interface is alive (16ms)
CertUtil: -ping command completed successfully.
```

**CA is running and responding:**
- [x] Yes — proceed to Part A
- [ ] No — describe issue:

---

## Part A — Application Event Log Navigation

The Windows Application event log records CA operational events by default — no configuration required. You will create a filtered view and document the CA event history on PKI-SRV01.

### Step 1 — Open Event Viewer

Log into PKI-SRV01. Open **Event Viewer**:
- Start → search "Event Viewer" → Open
- Or run: `eventvwr.msc`

### Step 2 — Navigate to the Application Log

In the left panel: **Windows Logs → Application**

### Step 3 — Create a Filtered View

Right-click **Application** → **Filter Current Log**

In the filter dialog:
- **Event sources:** Type `CertificationAuthority` and select it
- Leave all other fields at default
- Click **OK**

> **To save this filter as a Custom View (recommended):**
> Right-click **Custom Views** in the left panel → **Create Custom View** → same filter settings → Name it "CA Events" → OK

### Step 4 — Document Your Findings

Review the filtered event list. Identify at least **three distinct event types** present in your log. For each event, record the information below.

---

**Event 1**

Event ID:
```
26
```

Timestamp:
```
6/18/2026 9:35:55 PM
```

Source:
```
CertificationAuthority
```

Event Description (copy from the General tab of the event):
```
Active Directory Certificate Services for CVI Issuing CA 1 was started.  DC=DC01.corp.cvilab.local
```

What this event represents (in your own words):
```
The CA service (CertSvc) successfully started on PKI-SRV01 and completed its initialization,
including re-establishing its connection to the Active Directory domain controller DC01. This
event is logged every time the CA service comes online, whether from a scheduled restart,
a manual start, or a recovery following an error condition. In this case it followed the
Week 13 restore lab, confirming the CA was operational after the database was recovered.
```

---

**Event 2**

Event ID:
```
38
```

Timestamp:
```
6/18/2026 8:15:06 PM
```

Event Description:
```
Active Directory Certificate Services for CVI Issuing CA 1 was stopped.
```

What this event represents:
```
The CA service was cleanly shut down on PKI-SRV01. Event 38 is the expected counterpart to
Event 26 — together they bracket a CA service lifecycle. A stop event logged shortly before
a start event (as seen here, with Event 38 at 8:15 PM and Event 26 at 9:35 PM the same
evening) indicates the CA was taken offline, worked on, and then brought back up — consistent
with the recovery and restore operations performed during Week 13 labs.
```

---

**Event 3**

Event ID:
```
17
```

Timestamp:
```
6/18/2026 9:18:24 PM
```

Event Description:
```
Active Directory Certificate Services did not start: Unable to initialize the database
connection for CVI Issuing CA 1.  File not found 0xc8000713 (ESE: -1811 JET_errFileNotFound).
```

What this event represents:
```
The CA service attempted to start but failed because its Extensible Storage Engine (ESE)
database file could not be found. Error code JET_errFileNotFound is the Windows ESE engine's
signal that the expected .edb database file is missing from the CA's database directory. This
event was deliberately produced during the Week 13 Lab 03 recovery simulation, when the CA
database files were removed to simulate data loss. The error confirms the CA cannot operate
without its database and requires a restore before the service can start successfully.
```

---

**Additional events (optional — document as many as you find useful):**


Event ID 39 — Error — 6/10/2026 8:12:10 PM:
```
Source: CertificationAuthority
Description: Active Directory Certificate Services did not start: The Certification Authority
DCOM class for CVI Issuing CA 1 could not be registered. The class is configured to run as
a security id different from the caller 0x80004015 (-2147467243 CO_E_WRONG_SERVER_IDENTITY).
Use the services administration tool to change the Certification Authority logon context.

What this event represents: The CA service was unable to register its DCOM class during
startup because the service account identity did not match the identity expected by the DCOM
configuration. This error was generated during the Week 13 labs when the CA service was
restarted in the context of the recovery exercises. It indicates a service identity mismatch
that prevented the CA's COM interface from registering, which would block certificate
enrollment requests from reaching the CA.
```
---

Event ID 77 — Warning — 6/18/2026 7:46:42 PM: 
```
Source: CertificationAuthority
Description: The "Windows default" Policy Module logged the following warning: The Active
Directory connection to DC01.corp.cvilab.local has been reestablished to DC01.corp.cvilab.local.

What this event represents: The CA's Policy Module detected that its connection to the domain
controller DC01 had been interrupted and then successfully re-established. The Policy Module
uses the AD connection to validate certificate requests against AD attributes and to publish
issued certificates to the directory. A reestablishment warning indicates the CA temporarily
lost domain connectivity — likely during a CA service restart or brief network interruption
in the lab environment — and then recovered without requiring administrator intervention.
```
---

Event ID 128 — Warning — 5/31/2026 3:17:54 PM: 
```
Source: CertificationAuthority
Description: An Authority Key Identifier was passed as part of the certificate request 16.
This feature has not been enabled. To enable specifying a CA key for certificate signing,
run: "certutil -setreg ca\UseDefinedCACertInRequest 1" and then restart the service.

What this event represents: The CA received a certificate request (Request ID 16) that
included an Authority Key Identifier extension, which would instruct the CA to sign the
issued certificate using a specific CA key. This feature is disabled on CVI Issuing CA 1 by
default, so the CA logged a warning and processed the request using its standard signing key
instead. This was observed during Week 11 OCSP lab work when certificate requests were
submitted with explicit AKI values.
```

---

### Step 5 — Event Type Summary

Based on your filtered view, complete this table:

| Event Type | Present? | Approximate Count |
|------------|----------|-------------------|
| CRL publication events | No | 0 |
| CA service start/stop events | Yes | ~20+ |
| Certificate template update events | No | 0 |
| CA configuration change events | No | 0 |
| Error events (Level = Error) | Yes | 2 |


---

## Part B — CA Certificate Database Query

The CA certificate database contains every certificate request, issuance, and revocation since the CA was stood up. You will query it using certutil -view from an elevated PowerShell prompt on PKI-SRV01.

### Step 1 — Open an Elevated PowerShell Prompt

Right-click **Windows PowerShell** → **Run as Administrator**

Confirm: `whoami` returns `corp\pki.admin`

### Step 2 — Query All Issued Certificates

```powershell
certutil -view -restrict "Disposition=20"
```

This returns all certificates with Disposition = 20 (Issued and active).

```
Row 1: Request ID 9  | Requester: CORP\svc.autoenroll  | CN: Svc Autoenroll        | Template: CVI Service Account
Row 2: Request ID 10 | Requester: CORP\PKI-SRV01$      | CN: CVI-WebServer          | Template: CVI-WebServer
Row 3: Request ID 11 | Requester: CORP\PKI-SRV01$      | CN: PKI-SRV01.corp...      | Template: OCSP Response Signing
Row 4: Request ID 12 | Requester: CORP\PKI-SRV01$      | CN: PKI-SRV01.corp...      | Template: OCSP Response Signing
Row 5: Request ID 13 | Requester: CORP\PKI-SRV01$      | CN: PKI-SRV01.corp...      | Template: OCSP Response Signing
Row 6: Request ID 14 | Requester: CORP\PKI-SRV01$      | CN: OCSP-Valid             | Template: CVI-WebServer
Row 7: Request ID 16 | Requester: CORP\PKI-SRV01$      | CN: PKI-SRV01.corp...      | Template: OCSP Response Signing
Row 8: Request ID 17 | Requester: CORP\PKI-SRV01$      | CN: pki-srv01.corp...      | Template: WebServer

CertUtil: -view command completed successfully.
```

**Total number of issued certificates found:**
```
8
```

### Step 3 — Query All Revoked Certificates

```powershell
certutil -view -restrict "Disposition=21"
```

```
[Full certutil -view -restrict "Disposition=21" output — 8 rows returned]

Row 1: Request ID 3  | Requester: CORP\pki.admin   | CN: webserver.corp...    | Revocation Reason: Cessation of Operation
Row 2: Request ID 4  | Requester: CORP\PKI-SRV01$  | CN: CVI-WebServer        | Revocation Reason: Unspecified
Row 3: Request ID 5  | Requester: CORP\pki.admin   | CN: PKI Admin            | Revocation Reason: Cessation of Operation
Row 4: Request ID 6  | Requester: CORP\pki.admin   | CN: PKI Admin            | Revocation Reason: Cessation of Operation
Row 5: Request ID 7  | Requester: CORP\pki.admin   | CN: PKI Admin            | Revocation Reason: Key Compromise
Row 6: Request ID 8  | Requester: CORP\PKI-SRV01$  | CN: CVI-WebServer        | Revocation Reason: Cessation of Operation
Row 7: Request ID 15 | Requester: CORP\PKI-SRV01$  | CN: OCSP-Revoked         | Revocation Reason: Key Compromise
Row 8: Request ID 18 | Requester: CORP\PKI-SRV01$  | CN: pki-srv01.corp...    | Revocation Reason: Key Compromise

CertUtil: -view command completed successfully.

```

**Total number of revoked certificates found:**
```
8
```

### Step 4 — Find Certificates From Your Prior Labs

Use the requester filter to find certificates associated with the accounts used in your Week 10 and 11 labs.

```powershell
certutil -view -restrict "RequesterName=CORP\pki.admin"
```

```
(paste output here)
```

If you also used CORP\cert.manager:
```powershell
certutil -view -restrict "RequesterName=CORP\cert.manager"
```

```
[Full certutil -view -restrict "RequesterName=CORP\pki.admin" output — 4 rows returned]

Row 1: Request ID 3  | CN: webserver.corp.cvilab.local | Disposition: Revoked | Revocation Reason: Cessation of Operation
Row 2: Request ID 5  | CN: PKI Admin                   | Disposition: Revoked | Revocation Reason: Cessation of Operation
Row 3: Request ID 6  | CN: PKI Admin                   | Disposition: Revoked | Revocation Reason: Cessation of Operation
Row 4: Request ID 7  | CN: PKI Admin                   | Disposition: Revoked | Revocation Reason: Key Compromise

CertUtil: -view command completed successfully.
```

### Step 5 — Document Two Specific Certificate Records

Identify at least **two specific certificates** from your prior lab work (Weeks 10 or 11). For each, record the full certificate record fields.

---

**Certificate Record 1**

Request ID:
```
3
```

Requester Name:
```
CORP\pki.admin
```

Certificate Template:
```
CVI-WebServer
```

Issued Common Name:
```
webserver.corp.cvilab.local
```

Not Before:
```
5/13/2026 5:53 PM
```

Not After:
```
4/25/2027 7:36 PM
```

Serial Number:
```
440000000317ed0d04bd763dc4000000000003
```

Disposition:
```
21 (Revoked)
```

Revocation Date (if revoked):
```
5/16/2026 6:04 PM
```

Revocation Reason (if revoked):
```
Cessation of Operation (0x5)
```

**Which lab did this certificate come from?**
```
Week 10 Lab 01 — initial web server certificate enrollment exercise using certreq.exe with
the CVI-WebServer template. This certificate was the first one issued by CORP\pki.admin and
was subsequently revoked during Week 11 revocation labs.
```

---

**Certificate Record 2**

Request ID:
```
7
```

Requester Name:
```
CORP\pki.admin
```

Certificate Template:
```
CVI Code Signing
```

Issued Common Name:
```
PKI Admin
```

Not Before:
```
5/23/2026 9:22 AM
```

Not After:
```
4/25/2027 7:36 PM
```

Serial Number:
```
4400000007172a43e46a06421e000000000007
```

Disposition:
```
21 (Revoked)
```

Revocation Date (if revoked):
```
5/30/2026 7:07 AM
```

Revocation Reason (if revoked):
```
Key Compromise (0x1)
```

**Which lab did this certificate come from?**
```
Week 11 Lab 02 — code signing certificate enrollment exercise. This certificate was enrolled
by CORP\pki.admin using the CVI Code Signing template and was later revoked with reason Key
Compromise during the Week 11 revocation and CRL publication lab.
```

---

### Step 6 — Filter by Template

Run the following to see how certificates are distributed across templates:

```powershell
# List all issued certificates showing only template and CN
certutil -view -restrict "Disposition=20" -out "CertificateTemplate,CommonName,NotAfter"
```

```
Schema:
  Column Name       Localized Name              Type    MaxLength
  ----------------  --------------------------  ------  ---------
  CertificateTemplate  Certificate Template     String  254 -- Indexed
  CommonName           Issued Common Name       String  8192 -- Indexed
  NotAfter             Certificate Expiration Date  Date  8 -- Indexed

Row 1:
  Certificate Template: CVI Service Account
  Issued Common Name: "Svc Autoenroll"
  Certificate Expiration Date: 4/25/2027 7:36 PM

Row 2:
  Certificate Template: CVI-WebServer
  Issued Common Name: "CVI-WebServer"
  Certificate Expiration Date: 4/25/2027 7:36 PM

Row 3:
  Certificate Template: OCSP Response Signing
  Issued Common Name: "PKI-SRV01.corp.cvilab.local"
  Certificate Expiration Date: 6/14/2026 6:44 AM

Row 4:
  Certificate Template: OCSP Response Signing
  Issued Common Name: "PKI-SRV01.corp.cvilab.local"
  Certificate Expiration Date: 6/14/2026 6:54 AM

Row 5:
  Certificate Template: OCSP Response Signing
  Issued Common Name: "PKI-SRV01.corp.cvilab.local"
  Certificate Expiration Date: 6/14/2026 6:58 AM

Row 6:
  Certificate Template: CVI-WebServer
  Issued Common Name: "OCSP-Valid"
  Certificate Expiration Date: 4/25/2027 7:36 PM

Row 7:
  Certificate Template: OCSP Response Signing
  Issued Common Name: "PKI-SRV01.corp.cvilab.local"
  Certificate Expiration Date: 6/14/2026 3:07 PM

Row 8:
  Certificate Template: WebServer
  Issued Common Name: "pki-srv01.corp.cvilab.local"
  Certificate Expiration Date: 4/25/2027 7:36 PM

8 Rows
CertUtil: -view command completed successfully.
```

**Templates represented in your CA database:**
```
- CVI Service Account (1 certificate)
- CVI-WebServer (2 certificates)
- OCSP Response Signing (4 certificates)
- WebServer (1 certificate)
```

---

## Part C — Analysis

Answer the following questions using the event log data from Part A and the certificate database data from Part B.

### Analysis Question 1

Select **one event log entry** from Part A (any event type). In 3–5 sentences, explain what this event tells you about the CA's operational state at the time it was generated. Be specific about what action caused the event and what the event confirms about the CA.

```
Event ID 17 (Error), logged at 6/18/2026 9:18:24 PM, tells a precise story about the CA's
state at that moment: the CertSvc service attempted to start, opened the Windows ESE database
engine, and immediately failed when the engine returned error JET_errFileNotFound
(0xc8000713), meaning the primary database file (.edb) was absent from the expected path on
disk. This failure occurred because the CA database files had been deliberately removed as
part of the Week 13 Lab 03 recovery simulation, leaving the service with no database to
attach to. The event confirms that the CA was entirely non-functional at 9:18 PM — it could
not issue, sign, or process any certificate request — because the entire certificate record
store was unavailable. The presence of Event 26 (CA started successfully) at 9:35 PM in the
same log confirms that the database was successfully restored within approximately 17 minutes
of this failure, after which the CA returned to normal operation.
```

### Analysis Question 2

Select **one certificate record** from Part B. In 3–5 sentences, explain what this record tells you about the lifecycle of that specific certificate. Include the template used, who requested it, its current status, and what you would need to do next if the certificate were approaching expiry.

```
Certificate Record 1 (Request ID 3) documents the complete lifecycle of a web server
certificate issued to CN=webserver.corp.cvilab.local. The certificate was requested by
CORP\pki.admin on 5/13/2026 using the CVI-WebServer template, which grants Server
Authentication capability and is designed for machine-facing TLS endpoints. The CA issued it
immediately upon submission, with a validity window running from 5/13/2026 to 4/25/2027,
but the certificate never reached its natural expiry — CORP\pki.admin revoked it on
5/16/2026 with reason Cessation of Operation, indicating the endpoint or role that certificate
served was being decommissioned rather than replaced. If this certificate were still active and
approaching its 4/25/2027 expiry, the correct action would be to enroll a replacement using
the same CVI-WebServer template well before the expiry date, install the new certificate on
the target endpoint, and then revoke the expiring certificate with reason Superseded to
ensure it is properly reflected in the CRL before it naturally expires.
```

### Analysis Question 3

In 4–6 sentences, explain how the Application event log and the CA certificate database complement each other as operational data sources. Give a specific example scenario where you would need to consult **both** sources to fully understand what happened — and explain why one source alone would be insufficient.

```
The Application event log and the CA certificate database record fundamentally different
dimensions of CA activity: the event log captures what happened to the CA service itself —
starts, stops, errors, and policy module warnings — while the certificate database records
what the CA did on behalf of requesters — which certificates were issued, to whom, under
which templates, and whether they were later revoked and why. Neither source alone provides
a complete operational picture, which is why both must be consulted together for root cause
analysis. Consider a scenario where a relying party reports that a certificate they received
is suddenly being rejected as untrusted. The certificate database would show that the
certificate was revoked (Disposition 21), by whom, on what date, and with what reason code,
confirming that the rejection is legitimate and intentional. But the event log might reveal
that the CA service was stopped and restarted just before the revocation was recorded —
which, when combined with the database record, could indicate that an administrator noticed
an error condition, restarted the service to recover, and then immediately revoked a
certificate that may have been issued incorrectly during that error window. Without the event
log, the revocation appears routine; without the certificate database, the restart appears
routine — only together do they reveal that the revocation was a direct response to a
detected service anomaly.
```

---

## Lab Report Questions

Answer each question in complete sentences.

**1. What event type did you find most frequently in the Application event log? What does the frequency of that event type tell you about the CA's most common operation in your lab environment?**

```
Event ID 26 (CA service started) and Event ID 38 (CA service stopped) were the most
frequently occurring event types in the filtered view, with the two together accounting for
the majority of the 75 CertificationAuthority events present in the log. This frequency
reflects the nature of the lab environment rather than a production pattern — each lab
exercise that involved restarting CertSvc, recovering the CA database, or taking snapshots
generated a new start/stop pair, so the cumulative count grew across twelve weeks of lab
work. In a production environment, service lifecycle events should be rare and would stand
out immediately as significant; their high frequency in this lab environment is simply a
byproduct of the iterative, hands-on nature of the coursework rather than an indicator of
instability.
```

**2. The certutil -view command queries the CA database, not the event log. What is the difference between what these two sources record — and what would you lose operationally if you had access to the event log but not the CA database?**

```
The event log records the operational health and lifecycle of the CA service itself —
when it started, when it stopped, when it encountered errors, and when its infrastructure
components (such as its AD connection) experienced disruptions. The CA database, by contrast,
records every certificate transaction: the identity of the requester, the template used, the
subject name, the validity period, the serial number, and the revocation status of every
certificate the CA has ever issued. Without the CA database, an administrator would have no
way to answer fundamental operational questions: which certificates are currently active,
which have been revoked and why, how many certificates are approaching expiry, whether a
specific user or machine has an active certificate, or what the full issuance history of the
CA looks like. The event log can tell you the CA was running at 9:35 PM, but only the
database can tell you what the CA issued while it was running. Losing the database would
mean losing the authoritative record of the CA's entire certificate inventory — something
that cannot be reconstructed from event logs alone.
```

**3. In the Disposition field, you saw codes 20 (Issued) and 21 (Revoked). If a certificate shows Disposition 21, what additional fields should you check to understand the full context of the revocation, and why?**

```
When a certificate shows Disposition 21, four additional fields provide the context needed
to understand the revocation fully. The Revocation Date field establishes when the revocation
took effect, which determines whether relying parties checking the CRL after that time would
have been denied trust — critical for assessing whether any harm occurred in the window
between issuance and revocation. The Effective Revocation Date matters separately from the
Revocation Date because it can be backdated, meaning the revocation is treated as having
occurred earlier than when it was actually entered; this is particularly important for Key
Compromise revocations. The Revocation Reason code (such as 0x0 Unspecified, 0x1 Key
Compromise, or 0x5 Cessation of Operation) tells you why the certificate was revoked, which
has audit and legal implications — a Key Compromise revocation carries significantly more
urgency and follow-up requirements than a Cessation of Operation revocation. Finally, the
Request Disposition Message field often records who performed the revocation
("Revoked by CORP\pki.admin"), establishing accountability and confirming the action was
authorized rather than automated. Together these fields transform a disposition code into a
complete revocation narrative.
```

**4. The Application event log records CA events by default. The Security event log does not record CA events without additional configuration (covered in Lesson 3). Based on what you found in Part A, what operational information is present in the Application log — and what is absent that would be important in a production environment?**

```
The Application event log captures CA service lifecycle events (start, stop, failure to
start), infrastructure connectivity events (AD connection loss and reestablishment), policy
module warnings (such as the Authority Key Identifier warning in Event 128), and DCOM
registration errors. This provides a solid record of whether the CA was running and whether
its core dependencies were healthy. What is notably absent from the Application log is any
record of individual certificate operations: there are no events for certificate issuance,
no events for certificate revocation, no events recording which accounts requested which
certificates, and no events capturing CRL publication activity. In a production environment
these omissions are significant — a security team investigating a suspected unauthorized
certificate issuance would find no evidence in the Application log, because that log does
not record who requested certificates or what was issued. The Security event log, once
configured with the appropriate audit policies (covered in Lesson 3), fills this gap by
recording enrollment requests, approvals, and revocations as auditable security events
tied to specific identities. Without Security log auditing, the CA has no per-transaction
accountability trail, only a service-level health log.
```

---

## Submission Checklist

- [x] Logged in as CORP\pki.admin — whoami output included
- [x] CA running and responding — Get-Service and certutil -ping output included
- [x] Application event log filtered by Source = CertificationAuthority
- [x] At least three distinct event types documented with Event ID, timestamp, and description
- [x] Event type summary table completed
- [x] certutil -view -restrict "Disposition=20" output included (all issued certs)
- [x] certutil -view -restrict "Disposition=21" output included (all revoked certs)
- [x] certutil -view requester filter output included
- [x] At least two specific certificate records documented in full
- [x] Template distribution output included
- [x] All three Part C analysis questions answered (minimum sentence counts met)
- [x] All four lab report questions answered in complete sentences
- [x] File committed to `labs/week-14/lab-01-event-log-navigation.md`
