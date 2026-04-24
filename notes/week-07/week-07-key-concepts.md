
# Week 7 Lesson Notes — PKI in Enterprise Infrastructure

## 1. Core Concept

Enterprise PKI isn't just about certificates, it's about knowing where certificates live, who owns them, and what happens when they expire. Before managing certificates in an environment, you must first understand where they exist. In enterprise infrastructure, certificates are not stored in one place, they are distributed across multiple systems, services, and teams.

Every HTTPS-enabled endpoint relies on TLS certificates, but those certificates can live in web servers, load balancers, APIs, devices, email systems, and even software signing processes. This creates a layered ecosystem where identity and trust are enforced at different points in the architecture.

In simple terms, PKI in the enterprise is not just about certificates — it’s about understanding where trust is established, how it is maintained, and who is responsible for it.

---

## 2. Why It Matters

In real enterprise environments, certificate management is not centralized. Different teams often manage different parts of the infrastructure:

- Application teams deploy certificates  
- Infrastructure teams manage servers and load balancers  
- Security teams monitor compliance and risk  

This creates **ownership gaps**, where no single team has full visibility.

This matters because:
- Certificates can expire unnoticed  
- Revoked certificates may still be active  
- Shadow certificates can exist without inventory tracking  

From the labs, it became clear that failures are rarely due to “missing certificates”, they are usually due to **misplaced trust, broken chains, or lack of visibility**.


---

## 3. Technical Breakdown

Enterprise PKI is a distributed system that manages identity and trust across multiple infrastructure layers using certificates issued by internal or public Certificate Authorities.

**- Components:**
These layers represent where certificates typically exist in an enterprise environment, each with different ownership and operational risks:

**1. Layer 1: Web Servers (Apache, NGINX)**  
Certificates stored as flat files (`.pem`, keystores) and referenced in configuration files. Typically managed by application or infrastructure teams.

**Microsoft IIS (Windows Certificate Store)**  
Certificates stored centrally in the OS and managed through IIS Manager. Ownership often overlaps between infrastructure and system administrators.

**2. Layer 2: Load Balancers (TLS Termination Point)**  
TLS sessions often terminate here instead of the backend server. Certificates are configured at the load balancer level, meaning backend systems may not even have certificates installed.

**3. Layer 3: Internal APIs and Microservices**  
Internal services use certificates issued by internal CAs, often generated programmatically. In modern environments (service mesh), mutual TLS (mTLS) is used where both client and server authenticate each other.

**4. Email and Remote Access**
- **S/MIME**: Certificates tied to individual users for signing and encrypting email  
- **VPN Authentication**: Certificates used for device and user identity  

A major challenge here is scale, thousands of user certificates may expire or require revocation at any given time.

**5. Device Identity (Zero Trust Layer)**  
Certificates identify devices for network access control and conditional access policies. No certificate = no access.

**6. Code Signing**  
Certificates verify that software comes from a trusted source and has not been altered. Used in:
- Internal deployments  
- Vendor software  
- OS updates  

---

### Flow

This same flow was applied throughout the labs when retrieving, inspecting, and validating certificates from live endpoints. 
Certificate usage in enterprise environments typically follows this pattern:

1. Client connects to endpoint (browser, API, device)  
2. TLS handshake begins  
3. Certificate is presented (server or both sides in mTLS)  
4. Certificate is validated (chain, expiration, revocation)  
5. Trust is established → secure communication begins  

From the labs, this aligns directly with:
**Retrieve → Parse → Validate → Verify**

---

### Trust Implications

Trust in PKI depends on:
- Valid certificate chains  
- Trusted root Certificate Authorities  
- Non-expired certificates  
- Non-revoked status  

In enterprise environments:
- Public CAs provide universal trust  
- Internal CAs provide controlled, private trust  

A failure in any of these breaks secure communication.

---

## 4. Common Misconceptions

- **“Certificates are managed in one place”**  
  In reality, they are distributed across multiple systems and teams.

- **“If a certificate exists,it's good until it expires" Certificates can be revoked before expiration. A revoked certificate is no longer trusted, even if it's still within its validity period. Systems that don't check revocation status (via OCSP or CRL) may continue trusting a revoked certificate.

---

## 5. Where This Shows Up

### Web Security:
- Public-facing websites use public CA certificates
- Load balancers terminate TLS and may re-encrypt to backend servers
- Certificate transparency logs track all public certificates issued for a domain

### Internal Enterprise Systems:
- VPNs use device certificates for authentication
- Internal APIs authenticate with mTLS (mutual TLS)
- Device identity certificates control network access

### Cloud Environments:
- AWS Certificate Manager (ACM): Issues and auto-renews certificates for AWS services (ELB,           CloudFront, API Gateway). Certificates cannot be exported — private keys never leave AWS.
- Azure Key Vault: Manages certificates for both Azure services and external systems. Certificates    can be exported, making it suitable for hybrid deployments.
- GCP Certificate Authority Service: Runs a managed internal CA. GCP operates the infrastructure,     the organization controls CA policy.

### DevOps Workflows:
- Code signing certificates validate deployment artifacts
- Container registries authenticate with certificates
- CI/CD pipelines use certificates for secure access  
  
---

## Mental Model

Enterprise PKI can be understood as:

**Identity + Trust + Verification**

- Identity → Who is communicating (certificate owner)  
- Trust → Who issued and validates that identity (CA)  
- Verification → How that trust is checked (chain, expiry, revocation)  

This became practical through the diagnostic process used in the labs:

- **Retrieve:** Pulled the certificate from a live CDN-backed endpoint
- **Parse:** Identified it as an EV certificate with organizational validation
- **Validate:** Confirmed the complete chain (leaf → intermediate → root)
- **Verify:** Checked TLS configuration, and revocation mechanisms

Three PKI models can coexist in an enterprise environments:

- Public CAs for external-facing services  
- Internal CAs for internal systems and device identity  
- Cloud-managed PKI (ACM, Azure Key Vault, GCP CA) for cloud-native deployments.

The challenge is integrating all three without creating gaps where certificates fall through the cracks
This same model applies across all layers; from web servers to APIs to cloud infrastructure.


