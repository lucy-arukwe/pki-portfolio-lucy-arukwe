# Lab 02: AD CS Console Exploration & CA Hierarchy Documentation

**Student Name:**  Lucy Arukwe
**Date Completed:**  May 4, 2026
**Phase:** 2 | **Week:** 9  
**Submission Path:** `labs/week-09/lab-02-environment-documentation.md`

---

## Part A — AD CS Console Exploration (PKI-SRV01)

### CA Console Nodes — Observations

| Node | Contents / Observations |
|------|------------------------|
| Revoked Certificates | No revoked certificates were present. The console displayed standard certificate-related columns:Columns visible: Request ID, Serial Number, Certificate Expiration Date, Certificate Effective Date, Revocation Date, Revocation Reason |
| Issued Certificates | No certificates had been issued yet at this stage of the lab |
| Pending Requests | The folder was empty with no pending certificate requests |
| Certificate Templates | The following certificate templates are currently published to CVI Issuing CA 1 and available for issuance: Administrator, Authenticated Session, Basic EFS, CA Exchange, CEP Encryption, Code Signing, Computer, Cross Certification Authority, Directory Email Replication, Domain Controller, Domain Controller Authentication, EFS Recovery Agent, Enrollment Agent, Enrollment Agent(Computer), Exchange Enrollment Agent (Offline request), Exchange Signature Only, Exchange User, IPSec, IPSec (Offline request), Kerberos Authentication, Key Recovery Agent, OCSP Response Signing, RAS and IAS Server, Root Certification Authority, Router (Offline request), Smartcard Logon, Smartcard User, Subordinate Certification Authority, Trust List Signing, User, Web Server, Workstation |

### CA Properties — Key Settings

**General Tab**
- CA Name: CVI Issuing CA 1
- Computer Name: PKI-SRV01

**Extensions Tab — CRL Distribution Points (CDP):**

```
C:\Windows\System32\CertSrv\CertEnroll\<CRLNameSuffix><CRLName><DeltaCRLAllowed>.crl
ldap:///CN=<CATruncatedName><CRLNameSuffix>,CN=<ServerShortName>,CN=CDP,CN=Public Key Services,
CN=Services,<ConfigurationContainer><CDPObjectClass>
```

**Extensions Tab — Authority Information Access (AIA):**

```
C:\Windows\System32\CertSrv\CertEnroll\<ServerDNSName>_<CaName><CertificateName>.crt
ldap:///CN=<CATruncatedName>,CN=AIA,CN=Public Key Services,CN=Services,CN=Configuration,
DC=corp,DC=cvilab,DC=local
```

**Storage Tab**
- Database Path: C:\Windows\System32\CertLog
- Log Path: C:\Windows\System32\CertLog

### Certificate Templates Console (certtmpl.msc)

Templates visible in the forest (list what you observed):

```
Administrator
Authenticated Session
Basic EFS
CA Exchange
CEP Encryption
Code Signing
Computer
Cross Certification Authority
Directory Email Replication
Domain Controller
Domain Controller Authentication
EFS Recovery Agent
Enrollment Agent
Exchange User
IPsec
Kerberos Authentication
Key Recovery Agent
OCSP Response Signing
RAS and IAS Server
Root Certification Authority
Router (Offline request)
Smartcard Logon
Smartcard User
Subordinate Certification Authority
Trust List Signing
User
Web Server
Workstation
```

---

## Part B — CA Hierarchy Verification (PKI-SRV01)

### Command: certutil -store -enterprise Root

```
Root "Trusted Root Certification Authorities"
================ Certificate 0 ================
Serial Number: 26373e51a6ab669340c47caef2232ce1
Issuer: CN=CVI Root CA, DC=corp, DC=cvilab, DC=local
NotBefore: 4/25/2026 6:15 PM
NotAfter: 4/25/2046 6:25 PM
Subject: CN=CVI Root CA, DC=corp, DC=cvilab, DC=local
CA Version: V0.0
Signature matches Public Key
Root Certificate: Subject matches Issuer
Cert Hash(sha1): b805e6ab548f6e7c57d3989f61de7fe6a51031d1

CertUtil: -store command completed successfully.
```

**What did you see?** (Subject, Issuer, Thumbprint — describe in your own words):

```
The Root certificate store contained the CVI Root CA certificate. The subject and issuer matched,
showing that it is a self-signed Root CA certificate. The SHA1 thumbprint is a unique identifier
for the certificate and helps confirm it as the trusted root of the PKI environment.
```

### Command: certutil -store -enterprise CA

```
CA "Intermediate Certification Authorities"
================ Certificate 0 ================
Serial Number: 5800000002f7714edc7f317c46008000000002
Issuer: CN=CVI Root CA, DC=corp, DC=cvilab, DC=local
NotBefore: 4/25/2026 7:26 PM
NotAfter: 4/25/2027 7:36 PM
Subject: CN=CVI Issuing CA 1, DC=corp, DC=cvilab, DC=local
CA Version: V0.0
Certificate Template Name (Certificate Type): SubCA
Non-root Certificate
Template: SubCA, Subordinate Certification Authority
Cert Hash(sha1): 5137a597de2c3085ec5816c7f11edc18cfcdbaf8
  No key provider information
  Provider = Microsoft Software Key Storage Provider
  Simple container name: CVI Issuing CA 1
  Unique container name: b52f658bb3f6465529f3a0187c63bc_f0a99c17-76d3-498a-97de-2992c06105fd
  ERROR: missing key association property: CERT_KEY_IDENTIFIER_PROP_ID
  Signature test passed
Certutil: -store command completed successfully.
```

**What did you see?** (Subject, Issuer, Thumbprint — describe in your own words):

```
The Intermediate Certification Authorities store contained the CVI Issuing CA 1 certificate.
The issuer was the CVI Root CA, showing that the Root CA signed and trusted the Issuing CA
certificate. This relationship forms the certificate chain of trust used throughout the environment.
```

---

## Part C — Active Directory Structure (DC01)

### Active Directory Users and Computers (dsa.msc)

**PKI Admins OU — accounts found:**

```
Cert Manager
PKI Admin
PKI Admins (Security Group)
```
**Role:** The PKI Admins OU contains the accounts and groups used for managing the PKI environment. 
Inside the OU were the Cert Manager account, the PKI Admin account, and the PKI Admins security group.


**pki.admin account — group memberships:**

```
Domain Admins (corp.cvilab.local/Users)
Domain Users (corp.cvilab.local/Users)
PKI Admins (corp.cvilab.local/PKI Admins)
```
**Role:** The pki.admin account is a member of Domain Admins, Domain Users, and PKI Admins, giving it full administrative 
access across the environment. This account is mainly used for managing servers, Active Directory, and Certificate Services configuration.


**cert.manager account — group memberships:**
```
Domain Users
PKI Admins
```
**Role:** The cert.manager account is a member of Domain Users and PKI Admins only. Its role is more focused on certificate management tasks rather 
than full domain administration, making it a lower-privileged account compared to pki.admin.

**Domain-joined computer accounts found:**

```
PKI-SRV01
DC01
```

### Active Directory Sites and Services (dssite.msc)

**Server registered under Default-First-Site-Name:**

```
DC01
```

### Certificate Templates Console (certtmpl.msc on DC01)

Did templates appear here, confirming they are stored in AD?
- [x] Yes
- [ ] No — describe what happened:

---

## Part D — Environment Summary Write-Up

### 1. Environment Topology

*The lab environment consists of three virtual machines. DC01 is the Domain Controller and 
DNS server with the IP address 192.168.10.10. PKI-SRV01 is the Issuing Certificate Authority 
running Active Directory Certificate Services with the IP address 192.168.10.20. 
The Root-CA VM has the IP address 192.168.10.30 and acts as the offline Root Certificate Authority. 
During normal operation, only DC01 and PKI-SRV01 are powered on while the Root CA remains offline for 
security purposes.*

### 2. CA Hierarchy

*The environment uses a two-tier PKI hierarchy consisting of a Root CA and an Issuing CA. 
The CVI Root CA is the trusted root of the environment and is self-signed. The CVI Issuing CA 1 certificate was issued and signed by the Root CA, 
creating the chain of trust for certificates issued in the environment. The Root CA is kept offline to reduce the risk of compromise because if 
the Root CA private key were stolen, the entire PKI environment would no longer be trusted.*

### 3. Certificate Templates

*The following certificate templates were published to CVI Issuing CA 1:

• Directory Email Replication
• Domain Controller Authentication
• Kerberos Authentication
• EFS Recovery Agent
• Basic EFS
• Domain Controller
• Web Server
• Computer
• User
• Subordinate Certification Authority
• Administrator

Examples of Template Purposes

• Web Server: Designed to issue certificates used for HTTPS and TLS encryption on websites and servers. These certificates help secure communication between clients and servers.

• User: Designed to issue certificates for user authentication, secure email, encryption, and digital signatures.

• Computer: Used to issue certificates to domain computers for machine authentication and secure communication within the network.

• Subordinate Certification Authority: Used to issue certificates to subordinate or issuing CAs that operate under a trusted Root CA in the PKI hierarchy.

### 4. Active Directory Structure

*The PKI Admins OU contains the PKI-related administrative accounts and groups used in the environment. The pki.admin account is a member of Domain Admins, 
Domain Users, and PKI Admins, giving it elevated administrative privileges across the environment. The cert.manager account is a member of Domain Users and PKI Admins only, 
meaning it has more limited responsibilities related to certificate management rather than full domain administration.*

### 5. One Thing I Found Interesting or Unexpected

*One thing I found interesting was that certificate templates are stored in Active Directory rather than directly on the CA server itself. This design explains why the Domain Controller 
must be online for the CA to issue certificates correctly, the CA reads template definitions from AD at issuance time. 
It also revealed how tightly integrated Active Directory and Certificate Services are in an enterprise PKI environment, 
with AD serving as the central repository for PKI policy and configuration rather than just authentication.*

---

## Submission Checklist

- [x] Part A: All five console nodes documented
- [x] Part A: CA Properties (CDP, AIA, Storage) recorded
- [x] Part A: Certificate Templates console observed
- [x] Part B: certutil -store -enterprise Root output included
- [x] Part B: certutil -store -enterprise CA output included
- [x] Part C: PKI Admins OU and both accounts documented
- [x] Part C: Sites and Services server noted
- [x] Part C: certtmpl.msc confirmed templates in AD
- [x] Part D: All five summary areas completed in own words
- [x] File committed to portfolio repo at `labs/week-09/lab-02-environment-documentation.md`
