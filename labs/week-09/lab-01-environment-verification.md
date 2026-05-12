# Lab 01: Environment Verification & VM Connectivity Check

**Student Name:**  Lucy Arukwe
**Date Completed:**  May 4, 2026
**Phase:** 2 | **Week:** 9  
**Submission Path:** `labs/week-09/lab-01-environment-verification.md`

---

## Step 1 – VM Startup & Login

**VMs started in correct order (DC01 → PKI-SRV01 → Root-CA):**
- [x] Yes
- [ ] No — describe what happened:

**Login credentials used:**

| VM | Account Used | Login Successful? |
|----|-------------|-------------------|
| DC01 | CORP\pki.admin | Yes |
| PKI-SRV01 | CORP\pki.admin | Yes |
| Root-CA | .\Administrator | Yes |

**Notes / issues encountered:**

```
DC01 was started first and allowed to fully boot with Server Manager loaded before starting PKI-SRV01. 
Both VMs started successfully and accepted domain login credentials without issues. Root-CA was left 
powered off as instructed in the lab guide.
```

---

## Step 2 – VM Connectivity Test

**Command run on PKI-SRV01:**

```powershell
Test-Connection -ComputerName DC01 -Count 2
```

**Output received:**

```
Source        Destination     IPV4Address     IPV6Address                              Bytes    Time(ms)
------        -----------     -----------     -----------                              -----    --------
PKI-SRV01     192.168.10.10   192.168.10.10                                            32       1
PKI-SRV01     192.168.10.10   192.168.10.10                                            32       0
PKI-SRV01     192.168.10.10   192.168.10.10                                            32       1
PKI-SRV01     192.168.10.10   192.168.10.10                                            32       0

Additional IPv4 verification: IPv4 Address. . . . . . . . . . . : 192.168.10.20
My IP Address
```

**DC01 responded successfully:**
- [x] Yes
- [ ] No — troubleshooting steps taken:

---

## Step 3 – CertSvc Service Status

**Command run on PKI-SRV01:**

```powershell
Get-Service -Name CertSvc
```

**Output received:**

```
Status   Name                               DisplayName
------   ----                               -----------
Running  CertSvc                            Active Directory Certificate Services
```

**CertSvc status shown:** ________________

**Service was Running:**
- [x] Yes
- [ ] No — action taken:

---

## Step 4 – Certification Authority Console

**Steps completed on PKI-SRV01:**
- [x] Opened certsrv.msc via Run dialog
- [x] Confirmed CA name visible: ________________
- [x] Confirmed CA status: ________________
- [x] Expanded left pane — folders visible (Revoked Certificates, Issued Certificates, etc.)

**Screenshot or description of what you observed in certsrv.msc:**

![CVI Issuing CA 1 Console](../../../../assets/screenshots/week-09/lab01-certsrv-console.png)

```
The Certification Authority console opened successfully and displayed CVI Issuing CA 1 in the left 
navigation pane with a green icon indicating the CA service is running. All five management folders 
were visible:
- Revoked Certificates
- Issued Certificates
- Pending Requests
- Failed Requests
- Certificate Templates
```

---

## Step 5 – Certificate Log File Path

**Command run on PKI-SRV01:**

```powershell
Get-ChildItem "C:\Windows\System32\CertLog"
```

**Output received:**

```
Directory: C:\Windows\System32\CertLog

Mode      LastWriteTime         Length   Name
----      -------------         ------   ----
-a----    5/4/2026 1:21 PM      1048576  CVI Issuing CA 1.edb
-a----    5/4/2026 1:21 PM      16384    CVI Issuing CA 1.jfm
-a----    5/4/2026 1:21 PM      8192     edb.chk
-a----    5/4/2026 1:21 PM      1048576  edb.log
-a----    4/25/2026 7:45 PM     1048576  edbres00001.jrs
-a----    4/25/2026 7:45 PM     1048576  edbres00002.jrs
-a----    4/25/2026 7:45 PM     1048576  edbtmp.log
-a----    5/4/2026 1:21 PM      20480    tmp.edb
```

**Files found in CertLog:**

| File Name | Approximate Size |
|-----------|-----------------|
| CVI Issuing CA 1.edb | 1MB |
| CVI Issuing CA 1.jfm | 16KB |
| edb.chk | 8KB  |
| edb.log | 1MB |
| edbres00001.jrs | 1MB | 
| edbres00002.jrs | 1MB |
| edbtmp.log | 1MB |
| tmp.edb | 20 KB |

---

## Reflection

**One thing that went well during this lab:**

```
The network connectivity and CA service verification worked successfully on the first attempt.
The environment was properly configured and all required services and folders were accessible.
```

**One thing that was confusing or unexpected:**

```
One thing that stood out was how important the VM startup order is in a PKI environment.
If DC01 is not fully loaded before PKI-SRV01 starts, authentication and Active Directory communication can fail.
```

**Why must DC01 start before PKI-SRV01? What happens if the order is reversed?**
```
DC01 must start before PKI-SRV01 because PKI-SRV01 is a domain member server that depends on Active Directory and 
DNS services provided by DC01. During startup and login, PKI-SRV01 needs to contact the domain controller to authenticate 
users and communicate with Active Directory Certificate Services. If PKI-SRV01 starts before DC01 is fully available, domain 
authentication may fail and services such as CertSvc may not function correctly. This can result in login errors, service failures, 
or issues connecting the Certification Authority to Active Directory.
```
---

## Submission Checklist

- [ ] All five steps completed
- [ ] All command outputs pasted
- [ ] Reflection section filled in
- [ ] File saved as `lab-01-environment-verification.md`
- [ ] File committed to my portfolio repo under `labs/week-09/`
