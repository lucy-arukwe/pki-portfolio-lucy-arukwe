# Lab 03: HSM Key Ceremony — Observation Report

**Student Name:**  Lucy Arukwe
**Date Completed:** May 17, 2026 
**Phase:** 2 | **Week:** 10  
**Submission Path:** `labs/week-10/lab-03-hsm-observation.md`

> **Note:** This is an instructor-led demonstration. You observe and document — no hands-on configuration required. The instructor runs SoftHSM2 on a dedicated demo machine. Your deliverable is a structured observation report.

---

## Overview

A Hardware Security Module (HSM) is a physical device that stores cryptographic keys in tamper-resistant hardware. Enterprise CAs use HSMs to protect CA private keys — the most sensitive material in the PKI environment. In this demonstration, the instructor runs a **PKCS#11 key ceremony** using **SoftHSM2**, a software-based HSM simulator that exposes the same interface as a physical HSM like the Thales Luna.

Your task: observe each step, document what the instructor ran, what it produced, and what the equivalent step would look like with a real HSM in an enterprise environment.

---

## Part A — Setup Context

**What is SoftHSM2?**

In your own words, describe what SoftHSM2 is and why it is being used for this demonstration instead of a physical HSM:

```
SoftHSM2 is a software-based HSM simulator that mimics how a real Hardware Security Module works.
It allows engineers to practice PKCS#11 operations, token initialization, and key ceremonies without needing expensive enterprise HSM hardware.
In this demonstration, it was used to teach the workflow and concepts behind enterprise CA key protection while still using the same command structure and interfaces found in production environments.
```

**What is PKCS#11?**

In your own words, describe what the PKCS#11 interface is:

```
PKCS#11 is a standard interface used for communicating with cryptographic devices like HSMs.
It acts as the bridge between applications and the hardware or software token storing the keys. Instead of the application directly touching the private key,
the application sends requests through the PKCS#11 library, and the signing operation happens inside the HSM boundary.
```

**HSM being simulated in this demo:**

| Item                                        | Value                                    |
|---------------------------------------------|------------------------------------------|
| Software used                               | SoftHSM2 Version 2.7.0                   |
| PKCS#11 library path                        | /opt/homebrew/lib/softhsm/libsofthsm2.so |                   
| Companion tool used (key ceremony commands) |  pkcs11-tool                             |

---

## Part B — Step-by-Step Observation

Document each step of the ceremony as the instructor runs it. Record the command, what it did, and what you observed in the output.

---

### Step 1 — Token Slot Initialization

**Command run by instructor:**

```
softhsm2-util --init-token --slot 0 --label "CVI-CAKeyStore" --pin 1234 --so-pin 5678
```

**What this command does:**

```
This command initializes a new SoftHSM token inside slot 0.
It creates the logical keystore container and assigns both a User PIN and a Security Officer PIN.
The User PIN is used for operational access while the SO-PIN is used for administrative control over the token.
```

**Output or confirmation observed:**

```
The token has been initialized and is reassigned to slot 2076269524
```

**Why this step is necessary:**

```
This step creates the secure container where the CA private key will live.
Without initializing the token first, the HSM has nowhere to securely store cryptographic objects or enforce access controls.
```

---

### Step 2 — Confirm Slot/Token State

**Command run:**

```
softhsm2-util --show-slots
```

**Output:**

```
The output displayed the initialized token with:

Token present: yes
Initialized: yes
User PIN initialized: yes
Label: CVI-CAKeyStore
Assigned slot ID
```

**What did the output confirm?**

```
The output confirmed that the token was successfully initialized and active.
It also confirmed that the token label, serial number, and PIN initialization were properly configured before key generation began.
```

---

### Step 3 — Key Generation

**Command run:**

```
pkcs11-tool --module /opt/homebrew/lib/softhsm/libsofthsm2.so \
--login --pin 1234 \
--keypairgen --key-type rsa:2048 \
--label "CVI-IssCA-Key" \
--id 01
```

**Key parameters used (record what you observed):**

| Parameter | Value                             |
|-----------|-----------------------------------|
| Key type  | RSA                               |
| Key size  | 2048 bits                         |
| Key label | CVI-IssCA-Key                     |
| Token     | CVI-CAKeyStore (slot 2076269526) |

**Output:**

```
Using slot 0 with a present token (0x7bc15bd6)
Key pair generated:
Private Key Object; RSA 2048 bits
label:      CVI-IssCA-Key
ID:         01
Usage:      decrypt, sign, signRecover, unwrap
Access:     sensitive, always sensitive, never extractable, local
Public Key Object; RSA 2048 bits
label:      CVI-IssCA-Key
ID:         01
Usage:      encrypt, verify, verifyRecover, wrap
Access:     local
```

**What this step accomplished:**

```
This step generated the CA key pair directly inside the token.
The important concept demonstrated was that the private key was created within the
HSM boundary instead of being generated as a normal file on disk. This reduces the risk of the private key being copied or exported.
The critical attributes "never extractable" and "local" confirm this: the key cannot be exported, and it was generated locally within the token rather than being imported from an external source.
The key is now available for signing operations, but only by providing the correct user PIN. The public key is accessible without authentication, which is normal since public keys are meant to be shared.
```

---

### Step 4 — Confirm Key is Token-Resident

**Command run:**

```
pkcs11-tool --module /opt/homebrew/lib/softhsm/libsofthsm2.so --login --pin 1234 --list-objects
```

**Output:**

```
Using slot 0 with a present token (0x7bc15bd6)
Private Key Object; RSA 2048 bits
label:      CVI-IssCA-Key
ID:         01
Usage:      decrypt, sign, signRecover, unwrap
Access:     sensitive, always sensitive, never extractable, local
Public Key Object; RSA 2048 bits
label:      CVI-IssCA-Key
ID:         01
Usage:      encrypt, verify, verifyRecover, wrap
Access:     local
```

**What the output proves:**

```
The output proves that the key pair is stored as permanent objects inside the token, not in temporary session memory or on the filesystem. The presence of both private and public key objects with matching labels and IDs confirms that the keypair is intact and accessible via the PKCS#11 interface. Most importantly, the private key attributes "sensitive, always sensitive, never extractable, local" demonstrate that this key cannot be copied or exported — it exists only within the HSM boundary.
This is the fundamental protection that makes HSM storage superior to software key storage.
```

---

### Step 5 — Any Additional Steps Demonstrated

*(Use this section if the instructor ran additional commands — e.g., key attribute inspection, PKCS#11 URI display, CA configuration reference)*

**Command:**

```
softhsm2-util --show-slots
```

**What it showed:**

```
This command displayed the current state of the available SoftHSM2 slots and tokens after the key generation process.
The output confirmed that the initialized token was still present, properly labeled as CVI-CAKeyStore, and remained accessible after the RSA key pair was generated.

It also showed details such as:

Slot ID
Token label
Serial number
Whether the token was initialized
Whether the User PIN was initialized

This step helped verify that the token configuration remained intact and operational throughout the ceremony workflow.
```

---

## Part C — SoftHSM2 vs. a Physical HSM

**Physical HSM reference: Thales Luna Network HSM**

Fill in the table based on the lesson and demonstration:

| Dimension                     | SoftHSM2 (Demo)       | Thales Luna (Enterprise)      |
|-------------------------------|-----------------------|-------------------------------|
| Key storage location          | Software (filesystem) | Tamper-resistant hardware     |
| Tamper resistance             | None                  | Physical tamper protection    |
| PKCS#11 interface             | Same                  | Same                          |
| FIPS validation               | Not applicable        | FIPS 140-2/140- Level3 validated    |
| Used in production CAs        | No — simulation only  | Yes                           |
| Cost                          | Free / open source    | Expensive enterprise hardware |
| Required for a CA private key | No                    | Common in enterprise PKI      |

**In your own words: what does a physical HSM protect against that SoftHSM2 cannot?**

```
A physical HSM protects against physical extraction and hardware tampering. Even if an attacker gains server access,
the private key still cannot be copied out of the device because cryptographic operations happen inside the hardware boundary.
SoftHSM2 teaches the workflow, but its keys are still ultimately stored in software on the operating system.
```

---

## Part D — The Key Ceremony Concept

**What is a key ceremony?**

In your own words, describe what a key ceremony is and why enterprise PKI deployments conduct them with multiple administrators present:

```
A key ceremony is a formal and audited process used to generate, protect, and document highly sensitive CA keys.
Enterprise organizations conduct ceremonies with multiple administrators present so that no single person has complete control over the CA private key.
The process usually includes documented procedures, assigned roles, witness verification, and secured backup handling.
The purpose is not only technical security, but also accountability and auditability.
```

**What would be different about this ceremony if CVI Issuing CA 1 were using a physical HSM?**

```
If a real HSM were used, the ceremony would likely happen in a secure datacenter or restricted room with multiple administrators physically present.
Hardware tokens, smart cards, or quorum authentication would be required before the HSM could be unlocked.
The entire process would be documented in audit records, including who attended, what commands were executed, backup key handling procedures, and security controls followed during the ceremony.
```

---

## Part E — Connection to Phase 2

**Software-protected CA key vs. HSM-protected CA key**

CVI Issuing CA 1 in your lab environment stores its private key in software (the Windows CNG Key Storage Provider). Based on what you observed in this demonstration:

**What is the operational risk of a software-stored CA private key vs. an HSM-protected key?**

```
The main operational risk of a software-protected CA key is that the key may be copied if the server is compromised. An attacker with administrative access could potentially export or steal the CA private
key and use it to issue fraudulent certificates. With an HSM-protected key, the private key remains inside protected hardware and signing operations happen without exposing the key itself.
```

**When you back up the CA in Week 13, what specific step is more sensitive because the private key is software-protected?**

```
The sensitive step is the CA backup export process using certutil -backup because the private key can be exported into backup files.
If those backup files are stolen or improperly secured, the CA private key could be compromised. In enterprise environments using HSMs, backups are usually handled through secure HSM-to-HSM cloning procedures instead.
```

---

## Reflection

**The most important thing you took away from this demonstration:**

```
The biggest takeaway from this demonstration was understanding how PKCS#11 acts as the connection between CA software and the HSM. It made the whole process feel more practical and less mysterious because every step,
from token initialization to key generation; was done through clear commands with visible output.

One thing that stood out was seeing attributes like never extractable and local in the key output. That was proof that the private key was created inside the token and could not simply be copied out like a normal file.

The demonstration also showed that protecting a CA key is not only about the technology itself. The documentation, multi-person control, and ceremony procedures are just as important because they help prevent mistakes,
misuse, or insider threats in enterprise environments.
```

**One question the demonstration raised that you want to understand better:**

```
One thing I want to understand better is how backup and recovery work in a real enterprise HSM environment. As explained in the Week 10 Lesson 4 notes,
the private key is supposed to remain inside the HSM and never be extractable.
That made me curious about how organizations recover those keys if hardware fails or disaster recovery is needed.

I would like to understand whether enterprises use secure HSM-to-HSM transfers, encrypted backup shares, or quorum-based recovery processes.
Learning how organizations handle recovery while still keeping the key protected would help me better understand what “never extractable” means in practice.
```

---

## Submission Checklist

- [x] Part A: SoftHSM2 and PKCS#11 described in own words
- [x] Part B: All ceremony steps documented — commands, outputs, explanations
- [x] Part C: SoftHSM2 vs. Thales Luna comparison table completed
- [x] Part C: Physical HSM protection question answered
- [x] Part D: Key ceremony concept explained in own words
- [x] Part D: Enterprise ceremony differences described
- [x] Part E: Software key vs. HSM key risk addressed
- [x] Part E: Connection to Week 13 backup noted
- [x] Reflection completed
- [x] File saved as `lab-03-hsm-observation.md`
- [x] File committed to portfolio repo under `labs/week-10/`
