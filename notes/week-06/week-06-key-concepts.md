# Week 6 Lesson Notes — Revocation, Expiration & Risk

## 1. Core Concept

PKI failures are rarely random. Most issues come from a small set of causes: expiration, misconfiguration, trust gaps, or revocation problems.

A structured approach is required to diagnose them properly. Guessing, restarting services, or searching for quick fixes may work temporarily, but often leaves the real issue unresolved. Over time, this leads to repeated incidents and loss of trust in the system.

To address this, the PKI Diagnostic Framework provides a consistent method for identifying the real cause of TLS failures.

---

## 2. Why It Matters
In real environments, TLS failures can disrupt critical systems such as internal portals, APIs, or clinical applications. These failures often appear suddenly, even when nothing seems to have changed.

Without a structured approach, teams may fix symptoms instead of causes. This leads to recurring outages, especially during high-pressure moments like late-day incidents or production changes.
The engineer who resolves them fastest follows a framework and documents findings clearly.
Using a framework ensures that issues are diagnosed correctly the first time, reducing downtime and preventing repeat failures. 

---

## 3. Technical Breakdown

- Definition:
  The PKI Diagnostic Framework is a step-by-step method used to identify the root cause of certificate-   related failures.
- Components:
Certificate (leaf)
Intermediate CA
Root CA (trust anchor)
Client trust store
Revocation systems (OCSP / CRL)

- Flow
  The framework follows four steps, in order:

Step 1 — Retrieve

Connect to the live system and extract the actual certificate:

openssl s_client -connect hostname:443 -showcerts

This ensures the certificate being analyzed is the one actually presented by the server, not what is expected or configured.

Step 2 — Parse

Inspect the certificate fields:

Validity (Not Before / Not After)
Subject (hostname)
SAN (Subject Alternative Name)
Issuer (who signed it)

This step identifies issues such as:

Expired certificates
Hostname mismatches
Incorrect certificate deployment
Wrong issuing CA
Step 3 — Validate the Chain

Confirm the full trust chain exists:

Leaf → Intermediate → Root
openssl verify cert.pem

This step identifies:

Missing intermediate certificates
Incomplete or broken chains
Step 4 — Check Revocation and Trust

Verify certificate status and client trust:

Check OCSP / CRL
Confirm root CA exists in client trust store
openssl x509 -in cert.pem -noout -text | grep -A2 "OCSP"
This step identifies:

Revoked certificates
Missing root certificates
Trust store inconsistencies
OCSP failures or availability issues

- Trust implications
  
Trust is not defined by the certificate alone. It depends on:

A complete chain
A trusted root CA
A valid certificate (time + identity)
A reachable and working revocation system

A failure in any of these areas breaks trust.
---

## 4. Common Misconceptions

**Misconception 1:** The certificate looks fine, so it must not be the problem.
A certificate can appear valid but still fail due to trust or configuration issues.

Misconception 2:
** Restarting the service fixed it, so the issue is resolved.”
Temporary fixes often hide the real cause and allow the issue to return later.

Misconception 3:
“If it works on one machine, it should work everywhere.”
Differences in client trust stores can cause inconsistent behavior across environments.

“DNS/CNAME changes can fix certificate issues.”
DNS does not change certificate identity. The hostname must match the certificate SAN.
---

## 5. Where This Shows Up

- Web security:
  Public-facing websites with TLS certificates
  Certificate expiration causing browser warnings
  SAN mismatches after domain migrations
  
- Internal enterprise systems:
  Trust store gaps on new network subnets
  Missing intermediate CAs after certificate renewals
  Self-signed certificates accidentally deployed to production
  
- Cloud environments:
  Load balancers presenting wrong certificates
  Auto-scaling groups with inconsistent certificate deployment
  Certificate rotation failures in containerized environments
  
- DevOps workflows
  CI/CD pipelines failing due to expired certificates
  Staging environments promoted to production without proper certificates
  OCSP stapling configuration errors


---

## Mental Model

PKI troubleshooting follows a simple model:

Identity + Trust + Verification

Identity: Does the certificate match the hostname?
Trust: Does the client trust the issuing CA?
Verification: Is the certificate valid and properly chained?

If any one of these fails, the connection fails.
