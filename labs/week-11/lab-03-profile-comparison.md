# Lab 03: Compare Certificate Profiles Side by Side

**Student Name:**   Lucy Arukwe
**Date Completed:** May 24, 2026
**Phase:** 2 | **Week:** 11  
**Submission Path:** `labs/week-11/lab-03-profile-comparison.md`

---

## Overview

Lab 03 is a synthesis exercise. You have issued three certificates across Weeks 10 and 11:

1. **TLS certificate** — issued from CVI-WebServer in Week 10, Lab 02
2. **Service account certificate** — issued from CVI-ServiceAccount in Week 11, Lab 01
3. **Code signing certificate** — issued from CVI-CodeSigning in Week 11, Lab 02

In this lab, you inspect all three certificates using certutil and document their settings in a structured comparison table. Then you write an analytical explanation of why the differences exist.

> **Prerequisite:** All three certificates must be present in their respective stores on PKI-SRV01. If the Week 10 TLS certificate is no longer available, contact the instructor before proceeding.

---

## Pre-Lab — Locate All Three Certificates

**Step 1 — Confirm the Week 10 TLS certificate is in the store**

```powershell
# List all certificates in the personal store
certutil -store My
```

**Week 10 TLS certificate present:**
- [X] Yes — Thumbprint: 11280bd800780eadada46133da11ee63d3c63e534d
- [ ] No — contact instructor before proceeding

**Step 2 — Record all three thumbprints**

| Certificate              | Template           | Thumbprint                                |
|--------------------------|--------------------|-------------------------------------------|
| TLS (Week 10, Lab 02)    | CVI-WebServer      | 11280bd800780eadada46133da11ee63d3c63e534d|
| Service Account (Lab 01) | CVI-ServiceAccount | 762c31ae2ec7b9430ad86b6a616199e4369a74c7  |
| Code Signing (Lab 02)    | CVI-CodeSigning    | 647515E560C6CD47BC3A9507024D1356F73EEA32  |

---

## Part A — Inspect All Three Certificates

Run `certutil -store My "<thumbprint>"` for each certificate. Paste the full output for each.

### Certificate 1 — TLS (CVI-WebServer)

```powershell
certutil -store My "<TLS-thumbprint>"
```

**Full certutil output:**

```
PS C:\Windows\system32> certutil -store My

My "Personal"

================ Certificate 0 ================
Serial Number: 5800000002f7714edc7f317c46000000000002
Issuer: CN=CVI Root CA, DC=corp, DC=cvilab, DC=local
NotBefore: 4/25/2026 7:26 PM
NotAfter: 4/25/2027 7:36 PM
Subject: CN=CVI Issuing CA 1, DC=corp, DC=cvilab, DC=local
CA Version: V0.0
Certificate Template Name (Certificate Type): SubCA
Non-root Certificate
Template: SubCA, Subordinate Certification Authority

Cert Hash(sha1): 5137a597de2c3085ec5816c7f11edc18cfcdbaF8

  Key Container = CVI Issuing CA 1
  Unique container name: b52f658bb3f263e6f529f3a0187c63bc_f0a99c17-76d3-498a-97de-2992c06105fd
  Provider = Microsoft Software Key Storage Provider

Signature test passed

================ Certificate 1 ================
Serial Number: 440000000a0d2ff137199552aa00000000000a
Issuer: CN=CVI Issuing CA 1, DC=corp, DC=cvilab, DC=local
NotBefore: 5/24/2026 12:46 PM
NotAfter: 4/25/2027 7:36 PM
Subject: CN=CVI-WebServer
Non-root Certificate
Template: CVI-WebServer

Cert Hash(sha1): 11280bd800780eadada46133da11ee63d3c63e534d

  Key Container = a330acdeaad10128a9b113d1e949ce6a_f0a99c17-76d3-498a-97de-2992c06105fd
  Simple container name: te-CVI-WebServer-70056b4a-5fd1-4994-977c-bb8fe0ba5d78
  Provider = Microsoft RSA SChannel Cryptographic Provider

Private key is NOT exportable
Encryption test passed

CertUtil: -store command completed successfully.
```

---

### Certificate 2 — Service Account (CVI-ServiceAccount)

```powershell
certutil -store My "<ServiceAccount-thumbprint>"
```

Or, if stored in the service account store:

```powershell
certutil -store -service svc.autoenroll My "<ServiceAccount-thumbprint>"
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
Template: CVI-ServiceAccount, CVI Service Account

Cert Hash(sha1): 762c31ae2ec7b9430ad86b6a616199e4369a74c7

  Key Container = 4379e4213699fcaf1408a0d217893ad_f0a99c17-76d3-498a-97de-2992c06105fd
  Simple container name: te-CVI-ServiceAccount-d363b350-8fbc-4e15-bc46-1bc11727e5bf
  Provider = Microsoft Enhanced Cryptographic Provider v1.0

Encryption test passed

CertUtil: -store command completed successfully.
```

---

### Certificate 3 — Code Signing (CVI-CodeSigning)

`certutil -store My "<CodeSigning-thumbprint>"`

```powershell
certutil -store My "<CodeSigning-thumbprint>"
```

**Full certutil output:**

```
```powershell
PSPath                      : Microsoft.PowerShell.Security\Certificate::CurrentUser\My\647515E560C6CD47BC3A9507024D1356F73EEA32
PSParentPath                : Microsoft.PowerShell.Security\Certificate::CurrentUser\My
PSChildName                 : 647515E560C6CD47BC3A9507024D1356F73EEA32
PSDrive                     : Cert
PSProvider                  : Microsoft.PowerShell.Security\Certificate
PSIsContainer               : False
EnhancedKeyUsageList        : {Code Signing (1.3.6.1.5.5.7.3.3)}
DnsNameList                 : {PKI Admin}
Archived                    : False
Issuer                      : CN=CVI Issuing CA 1, DC=corp, DC=cvilab, DC=local
NotAfter                    : 4/25/2027 7:36:58 PM
NotBefore                   : 5/23/2026 9:22:21 AM
HasPrivateKey               : True
SerialNumber                : 4400000007172A43E46A06421E000000000007
Thumbprint                  : 647515E560C6CD47BC3A9507024D1356F73EEA32
Issuer                      : CN=CVI Issuing CA 1, DC=corp, DC=cvilab, DC=local
Subject                     : CN=PKI Admin, OU=PKI Admins, DC=corp, DC=cvilab, DC=local
```
---

### Comparison Table

Complete the following table using the certutil outputs above.

| Field           | TLS Certificate                          | Service Account Certificate              | Code Signing Certificate                 |
| --------------- | ---------------------------------------- | ---------------------------------------- | ---------------------------------------- |
| Template Name   | CVI-WebServer                            | CVI-ServiceAccount                       | CVI-CodeSigning                          |
| Subject         | CN=CVI-WebServer                         | CN=Svc Autoenroll                        | CN=PKI Admin                             |
| Subject Source  | Supplied in request                      | Build from Active Directory              | Build from Active Directory              |
| Issuer          | CVI Issuing CA 1                         | CVI Issuing CA 1                         | CVI Issuing CA 1                         |
| Key Usage       | Digital Signature, Key Encipherment      | Digital Signature and Key Encipherment (Configured) | Digital Signature             |
| EKU             | Server Authentication                    | Client Authentication                    | Code Signing                             |
| EKU OID(s)      | 1.3.6.1.5.5.7.3.1                        | 1.3.6.1.5.5.7.3.2                        | 1.3.6.1.5.5.7.3.3                        |
| Validity Period | 5/24/2026 → 4/25/2027                    | 5/24/2026 → 4/25/2027                    | 5/23/2026 → 4/25/2027                    |
| Serial Number   | 440000000a0d2ff137199552aa00000000000a    | 4400000009217b3df4da50401000000000009    | 4400000007172A43E46A06421E000000000007   |
| Thumbprint      | 11280bd800780eadada46133da11ee63d3c63e534d | 762c31ae2ec7b9430ad86b6a616199e4369a74c7 | 647515E560C6CD47BC3A9507024D1356F73EEA32 |
| Request ID      | 10                                        | 9                                        | 7                                        |
---

## Part B — Written Analysis

This section is the substance of Lab 03. Write in prose paragraphs — not bullet points.

**1 — Key Usage**

Each of the three certificates has different Key Usage settings. Explain *why* each certificate has the Key Usage it has. For each, trace the setting back to the cryptographic operation the use case requires.

```
Each certificate used different Key Usage settings because each one supported a different cryptographic operation and trust workflow.

The TLS certificate and the service account certificate both included Digital Signature and Key Encipherment. In the TLS certificate, Digital Signature supported the TLS handshake and server authentication process, while Key Encipherment supported secure session key exchange between the client and the server. The service account certificate used the same two Key Usage settings because it also participated in secure authentication workflows and certificate-based authentication workflows where encrypted session establishment was required. Even though the TLS certificate represented a server identity and the service account certificate represented a non-human AD identity, both certificates still needed to support secure authenticated communication channels.

The code signing certificate was different because it was not encrypting communications or establishing TLS sessions. Its only purpose was to digitally sign PowerShell scripts and prove file integrity. Because of this, only Digital Signature was required. Key Encipherment was intentionally removed because the certificate was not designed for encryption or secure channel establishment. During the hash mismatch test, modifying the signed script caused the signature validation to fail immediately, demonstrating that the certificate was protecting the integrity of the script contents rather than encrypting the file itself.
```

---

**2 — Extended Key Usage (EKU)**

Each certificate has a different EKU. Explain why each certificate has the EKU it has. For each, identify the relying party application that enforces that EKU and what it would do if the EKU were absent or wrong.

```
The EKU extension controlled what each certificate was trusted to do at the operating system and application layer.

The TLS certificate contained the Server Authentication EKU because it represented a service endpoint that clients connected to during TLS communication. Browsers, operating systems, and TLS clients check for the Server Authentication EKU before trusting a certificate for HTTPS or other TLS-based services. If this EKU were missing or incorrect, browsers and clients would reject the certificate even if the certificate chain itself was otherwise valid.

The service account certificate contained only the Client Authentication EKU because it authenticated a non-human identity to backend systems and services. The certificate was intentionally restricted to Client Authentication to reduce misuse potential and follow the principle of least privilege. During the lab, this certificate represented the svc.autoenroll identity, which authenticated to systems rather than hosting services itself. Systems relying on mutual TLS or service authentication workflows would expect the Client Authentication EKU to be present before accepting the certificate for identity verification.

The code signing certificate contained only the Code Signing EKU (1.3.6.1.5.5.7.3.3). PowerShell and the Windows trust layer check specifically for this EKU before accepting a signature as trusted. The lab demonstrated that even if a cryptographic signature exists, the operating system still evaluates whether the certificate is authorized for code signing purposes. If the Code Signing EKU were missing, PowerShell could reject the signature even if the mathematical signature validation succeeded.
```

---

**3 — Subject Name Source**

The TLS certificate uses "Supplied in request" for its subject. The service account and code signing certificates use "Build from Active Directory." Explain why the subject name source is different for the TLS certificate — what is it about the TLS use case that makes AD-supplied subject names impractical?

```
The TLS certificate used “Supply in the request” for the subject name because server certificates often represent hostnames, DNS names, or external services that may not directly exist as Active Directory identities. In real-world environments, web servers, APIs, VPN gateways, and applications commonly require certificates tied to specific DNS names or SAN entries rather than user principals stored in AD. Because of this, manually supplying the subject name is more flexible for TLS service identities.

The service account and code signing certificates used “Build from Active Directory” because both certificates represented identities that already existed in AD. The service account certificate represented the svc.autoenroll non-human identity, while the code signing certificate represented the pki.admin administrative identity. Using AD-supplied subject information ensured the identities were centrally managed, consistently named, and not manually self-asserted during enrollment.
```

---

**4 — Security Question**

Imagine a single certificate that combined all three EKUs: Server Authentication, Client Authentication, and Code Signing. What security risk would this create? Be specific — what could an attacker do with a compromised private key from this combined certificate that they could not do with any one of the three individual certificates?

```
Combining Server Authentication, Client Authentication, and Code Signing into a single certificate would create a major security risk because one compromised private key could be abused across multiple trust boundaries simultaneously.

If an attacker obtained the private key from such a combined certificate, they could impersonate trusted servers, authenticate as trusted service identities, and sign malicious software or scripts that appear legitimate to operating systems and users. Instead of compromising only one trust relationship, the attacker would gain access to multiple high-trust functions using the same credential. This would significantly increase the blast radius of a single key compromise.

Separating EKUs across different certificate templates limits the scope of trust associated with each private key. This follows the principle of least privilege and is one of the core architectural design principles in enterprise PKI environments.
```

---

## Reflection

**Which of the three certificates would you consider most critical to revoke quickly if the private key were compromised — and why?**

```
The code signing certificate would likely be the most critical certificate to revoke quickly because of the level of trust operating systems and users place in signed software. A compromised code signing certificate could allow an attacker to distribute malicious scripts, malware, or administrative tools that appear trusted and legitimate within the environment. Since users and systems may automatically trust signed code, the impact of a compromised code signing key could spread very quickly across multiple systems.

The service account certificate would also be highly sensitive because it could allow unauthorized backend authentication to systems and services. However, the code signing certificate presents a larger blast radius because it could potentially impact many users and systems simultaneously through trusted software distribution.
```

**What would you add to this comparison if you were presenting it to a security team evaluating the PKI environment?**

```
If this comparison were presented to a security team, I would also include certificate lifecycle controls such as renewal workflows, revocation procedures, expiration monitoring, and private key protection methods. I would additionally document whether the certificates supported autoenrollment, whether private keys were exportable, and what approval workflows existed for high-trust certificates such as code signing certificates.

Another useful addition would be a trust-boundary diagram showing which systems, services, or applications relied on each certificate type. This would help visualize the operational impact and blast radius if one of the certificates were compromised or expired unexpectedly.
```

---

## Submission Checklist

- [x] Pre-lab: All three certificate thumbprints recorded
- [x] Part A: certutil output for all three certificates pasted in code blocks
- [x] Part A: Comparison table fully completed — no blank cells
- [x] Part B: Key Usage analysis written in prose — explains *why*, not just *what*
- [x] Part B: EKU analysis written in prose — identifies the relying party for each
- [x] Part B: Subject Name source difference explained with reasoning
- [x] Part B: Security question answered — specific risk identified for combined-EKU certificate
- [x] Reflection completed
- [x] File saved as `lab-03-profile-comparison.md`
- [x] File committed to portfolio repo under `labs/week-11/`
- [x] All three Week 11 labs committed with a single meaningful commit message
- [x] Request IDs for all three certificates recorded (TLS from Week 10, Service Account, Code Signing) — needed for Week 12
