# Week 01 — Key Concepts

## Public Key Infrastructure (PKI) is a system that helps establish trust and secure communication on the internet. It solves the problem of verifying identities and protecting data when users connect to websites or exchange information online. PKI uses a pair of cryptographic keys; a public key that can be shared and a private key that must remain secret—to encrypt data and verify identities. Certificates link a public key to a verified identity and are issued by trusted Certificate Authorities. Trust is established through a chain of certificates that ultimately leads to a trusted root authority stored in the operating system or browser.
# Week 1 Lesson Notes — Digital Trust & Identity

## 1. What is Digital Trust?
Digital trust is the confidence that:
- a system is talking to the right system/person (**identity**),
- the information hasn’t been altered (**integrity**),
- and sensitive data is protected (**confidentiality**).

In real environments, digital trust is established through **cryptography + policy + verification**.

---

## 2. Identity vs Authentication vs Authorization
- **Identity**: who/what an entity claims to be (user, server, device, service).
- **Authentication**: proving that identity (password, certificate, MFA, key).
- **Authorization**: what that authenticated entity is allowed to do (permissions, access).

PKI is most commonly used for **authentication** and **secure key exchange** at scale.

---

## 3. Public Keys vs Private Keys
A key pair has two mathematically related keys:

### Private Key
- Must be protected
- Used to **sign** (prove possession) or **decrypt** (in some schemes)
- If compromised, trust is compromised

### Public Key
- Safe to distribute
- Used to **verify signatures** or **encrypt to the holder**

Key idea:
- **Private key = capability**
- **Public key = verification/encryption to you**

---

## 4. What is a Certificate?
A certificate is an identity document for a key.

It binds:
- a public key
- to an identity (subject / DNS name / organization)
- and is vouched for by a trusted authority (issuer / CA)

A certificate answers:
> “Who does this public key belong to, and should I trust that claim?”

---

## 5. Certificates vs Encryption (Common Confusion)
Certificates are not “encryption.”

Certificates enable secure encryption at scale by:
- making identity verifiable
- enabling trust decisions
- supporting secure key exchange (e.g., TLS)

Encryption is the *mechanism* that protects data.
Certificates are part of the *trust framework* that makes encryption usable in real systems.

---

## 6. Where PKI Shows Up
You interact with PKI constantly:
- HTTPS websites (TLS certificates)
- Mobile device management (device certs)
- Corporate Wi-Fi (802.1X cert auth)
- Email security (S/MIME)
- Code signing (signed executables)
- APIs and microservices (mTLS)

---

## 7. Mental Model to Keep
PKI is a system for:
**Identity + Trust + Verification at scale**

If you keep that model, the technical pieces later will make sense.
