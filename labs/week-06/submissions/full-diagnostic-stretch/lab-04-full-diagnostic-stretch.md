# Lab 04 — Full Diagnostic Scenario: Incident Report

**Week 6 · PKI Incident Diagnosis & Troubleshooting**
**CVI PKI Career Pathway — Phase 1 Foundations**

---

## PKI Incident Report

**System:** Metro General EHR portal (ehr.metrogeneral.org)
**Reported:** Friday, April 3, 2026 at 4:47 PM 
**Author:** Lucy Arukwe
**Status:** Diagnosis complete — pending remediation 

---

### Executive Summary

Clinical staff on the new clinical subnet cannot access the EHR system because their computers are missing a critical security credential that tells them to trust our internal IT department's digital certificates. The certificate itself is valid and works fine for staff on the main office network, but the new clinical computers were added after we distributed this security credential six months ago, so they never received it. We need to immediately install this missing credential on all clinical subnet computers so they can verify the EHR system's identity, and we also need to disable the old EHR certificate that was replaced last week to prevent potential security risks.

---

### Technical Findings

#### Finding 1 — Missing Root CA in Clinical Subnet Trust Stores

**Type:** Trust Store 
**Severity:** Critical 

**Detail:**
The primary cause of the incident is a trust failure on devices in the clinical subnet. The renewed EHR certificate was issued by Metro General’s internal CA, which depends on the internal root certificate being present in the client trust store. Devices on the main office network trust that root and connect successfully. Devices on the new clinical subnet do not, which causes TLS validation to fail even though the server certificate itself is valid.

**Evidence:**
- Internal CA root certificate was distributed via Group Policy 6 months ago
- Clinical subnet was added to the network 2 weeks ago — after the Group Policy deployment
- TLS connections succeed from main office network (10.10.0.0/24) but fail from clinical subnet (10.22.0.0/24)
- Same certificate and chain are presented to both networks, indicating the failure is client-side (trust store) rather than server-side (certificate/chain configuration)


---

#### Step 2 — Parse

**Commands to parse the certificate:**
```bash
openssl x509 -in ehr_cert.pem -text -noout
openssl x509 -in ehr_cert.pem -noout -dates
openssl x509 -in ehr_cert.pem -noout -pubkey
```

**What the certificate would confirm:**

- **New key confirmation:** The `openssl x509 -noout -pubkey` command would display the public key. Comparing this to the public key from the old certificate (if available) would confirm that a new key pair was generated. The scenario explicitly states the renewal replaced both the certificate and private key, so this field would show a different modulus/public key than the previous certificate.

- **Issuer field:** The Issuer would show `CN=Metro General Internal CA - G2, O=Metro General, ...` or similar. This confirms the certificate was issued by the correct internal CA.

- **Validity dates:** The certificate was issued last week with a 1-year validity window, so the Not Before date would be approximately 7 days ago and the Not After date would be approximately 358 days in the future. There is no indication of a validity date problem in the scenario — the certificate is not expired, not yet valid, or misissued with incorrect dates. The infrastructure team confirmed it was "working fine" on the main office network, which would not be possible if there were a validity date issue.

The certificate parsing step confirms that the certificate itself is valid and properly constructed. The failure must be elsewhere in the chain validation process.

---

#### Step 3 — Validate the Chain

**Command to validate the chain:**
```bash
openssl verify -CAfile metro_general_root.pem ehr_cert.pem
```

**Expected result and reasoning:**

On a device **with** the root CA installed (main office network), this command would return: `ehr_cert.pem: OK`

On a device **without** the root CA installed (clinical subnet), this command would return: `error 20 at 0 depth lookup: unable to get local issuer certificate`

This step confirms that the certificate chain is structurally valid — the signatures are correct, the chain is complete (leaf → intermediate → root), and the root CA certificate itself is trustworthy. The problem is purely that the root CA is not present in the trust stores of devices on the clinical subnet.

**Critical insight:** The EHR system works on the main office network but fails on the clinical subnet, yet the certificate and chain presented by the server are identical in both cases. This definitively localizes the failure to the client side (trust store configuration) rather than the server side (certificate or chain configuration). If this were a certificate problem (expired, wrong hostname, broken chain), it would fail on both networks. The network-specific failure pattern is diagnostic of a trust store distribution gap.

**Type of problem:** This is a **trust store problem**, not a certificate problem or chain problem. The certificate is valid. The chain is complete and properly constructed. The root CA is legitimate. The only issue is that the trust anchor (root CA certificate) is missing from the client devices' trust stores on the clinical subnet.

---

#### Step 4 — Check Revocation and Trust

**Commands to check revocation:**
```bash
openssl x509 -in ehr_cert.pem -noout -text | grep -A3 "CRL Distribution"
openssl x509 -in ehr_cert.pem -noout -text | grep -A3 "OCSP"
```

**Revocation considerations:**

The certificate was renewed with a full replacement (new private key). According to PKI lifecycle best practices, when a certificate is replaced — not just renewed with the same key — the old certificate should be revoked even if it has not been compromised. The operational risk of not revoking the old certificate is that two valid certificates exist for `ehr.metrogeneral.org`: the old one (which may still be within its validity period) and the new one. If an attacker obtained the old certificate and private key (through backup media, logs, or other means), they could use it to impersonate the EHR system until it naturally expires.

**How to verify whether the old certificate has been revoked:**

If the certificate contains a CRL Distribution Point, download the CRL and check if the old certificate's serial number is listed. If the certificate contains an OCSP responder URL, query the OCSP responder with the old certificate's serial number to check its revocation status. The scenario does not provide information suggesting the old certificate was revoked, and the infrastructure team's note ("The renewal shouldn't have changed anything") implies a lack of attention to lifecycle management details like revocation.

**Relevance to current incident:**

The revocation status of the old certificate is **not relevant** to the current TLS failures on the clinical subnet. The clinical subnet devices are failing to validate the **new** certificate due to the missing root CA in their trust stores. Even if the old certificate were properly revoked, it would not resolve the trust store gap affecting the new certificate. However, failure to revoke the old certificate is a secondary security issue that should be addressed as part of incident remediation to prevent future operational or security problems.

---

### Failures in Diagnostic Order

1. **Missing Root CA Certificate in Clinical Subnet Trust Stores**
   - **Type:** Trust Store
   - **Evidence:** Clinical subnet added 2 weeks ago, after Group Policy push 6 months ago. Same certificate chain validates successfully on main office network but fails on clinical subnet, indicating client-side trust store gap rather than server-side certificate/chain problem.

2. **Old Certificate Not Revoked After Full Replacement**
   - **Type:** Revocation / Lifecycle Management
   - **Evidence:** Scenario explicitly states renewal involved new private key ("full replacement, not a renewal"). Old certificate may still be within 1-year validity period. No indication in scenario that revocation was performed.

3. **Process Gap: New Network Segments Deployed Without Security Baseline Validation**
   - **Type:** Configuration / Process
   - **Evidence:** Clinical subnet went into production use without verification that internal CA root certificate was deployed. No pre-production TLS connectivity testing to critical services. Incident discovered by end users rather than validation procedures.

---

### Root Cause

The root cause of this incident is a process gap in the organization's network expansion procedures. When the clinical subnet was added to the network two weeks ago, there was no requirement or checklist to verify that critical security infrastructure — specifically, internal Certificate Authority root certificates — was deployed to all devices on the new segment before production use. The internal CA root had been distributed via Group Policy six months earlier, but the new subnet was not included in that deployment scope. Without a pre-production validation process that includes testing TLS connectivity to internal services and verifying trust store configuration against established security baselines, the subnet went live with incomplete PKI infrastructure. This allowed an operational incident to occur that directly impacted clinical staff's ability to access patient records, rather than being caught and remediated during a controlled deployment phase.

---

### Remediation Steps

1. **Immediate — Deploy Root CA Certificate to Clinical Subnet**
   - Create a targeted Group Policy Object (GPO) or Mobile Device Management (MDM) policy that includes the clinical subnet scope (10.22.0.0/24)
   - Deploy the Metro General Internal CA - G2 Root certificate to the Trusted Root Certification Authorities store on all devices
   - For devices that cannot wait for automatic policy refresh, manually distribute and install the root certificate or force a Group Policy update with `gpsh /force`
   - **Timeline:** Within 2–4 hours

2. **Short-term — Verify Full Remediation and Test Connectivity**
   - After root CA deployment completes, test TLS connectivity to ehr.metrogeneral.org from multiple devices on the clinical subnet
   - Use `openssl s_client` or browser testing to confirm verify return code is 0 (ok) and no certificate warnings appear
   - Work with helpdesk to confirm clinical staff can access EHR system without errors
   - Document which devices received the root CA via automated policy vs. manual installation
   - **Timeline:** Within 24 hours

3. **Secondary — Revoke Old Certificate**
   - Locate the serial number and details of the old ehr.metrogeneral.org certificate (from backup records or infrastructure team documentation)
   - Submit a revocation request to Metro General Internal CA - G2 with reason code "superseded" (since this was a planned replacement, not a compromise)
   - Verify the old certificate appears on the CA's Certificate Revocation List (CRL) or returns "revoked" status from the OCSP responder
   - Update incident documentation with old certificate serial number and revocation timestamp
   - **Timeline:** Within 48 hours

---

### Prevention Recommendations

1. **Implement Network Segment Pre-Production Validation Checklist**
   - Create a mandatory checklist for all new network segments that includes: Group Policy scope verification (confirm all security-related GPOs apply to new subnet), trust store validation (verify internal CA roots are present on test devices), TLS connectivity testing (validate connections to critical internal services like EHR, file shares, intranet portals), and sign-off from security team before production cutover. New subnets should not be opened to production traffic until all checklist items are verified.

2. **Automate Trust Store Monitoring and Alerting**
   - Deploy monitoring that periodically checks a sample of devices from each subnet to verify the presence of required internal CA root certificates. Configure alerts when devices are detected without the expected trust anchors, especially in newly deployed network segments. This creates a safety net that catches trust store distribution gaps before they cause user-facing incidents.

3. **Establish Certificate Lifecycle Procedures with Revocation Requirements**
   - Document a formal certificate lifecycle policy that explicitly requires revocation of old certificates when performing full replacements (new key generation). Include revocation verification as a mandatory step in post-renewal checklists. Train infrastructure team members on the distinction between key reuse renewals (old certificate can remain valid) and full replacements (old certificate must be revoked). This prevents security gaps from unrevoked certificates and ensures the team understands PKI lifecycle obligations.

---

### Lessons Learned

This incident revealed a significant gap in how Metro General manages PKI infrastructure during network expansion. While the organization has established procedures for distributing internal CA root certificates (via Group Policy), there is no corresponding process to ensure new network segments receive this critical security infrastructure before going into production. The clinical subnet deployment proceeded without verifying that devices could validate certificates for internal services, allowing a predictable and preventable failure to impact clinical operations. Going forward, the organization should treat trust store configuration as a mandatory component of network security baselines and implement validation checkpoints before new infrastructure goes live. Additionally, the infrastructure team's initial assessment ("The certificate looks fine") exposed a knowledge gap around distinguishing between certificate validity (which was correct) and trust store configuration (which was incomplete). This suggests a need for additional PKI training focused on systematic diagnostic approaches rather than surface-level certificate inspection.

---

## Reflection

The hardest part of this multi-failure investigation was distinguishing between the primary failure (missing root CA in trust stores) and the secondary issue (unrevoked old certificate). Initially, I was tempted to focus on the certificate renewal itself because the scenario emphasized it heavily, but following the framework forced me to recognize that the certificate was valid and the network-specific failure pattern was diagnostic of a client-side trust store problem, not a server-side certificate problem. In a real production incident, I would prioritize communication with affected users much more heavily — providing an estimated time to resolution within the first hour and updating them as remediation progresses. I would also coordinate with the infrastructure team immediately to understand their Group Policy deployment capabilities and whether they could push the root certificate to the clinical subnet faster than the standard policy refresh cycle.


---

*CVI PKI Career Pathway — Foundations Phase*
