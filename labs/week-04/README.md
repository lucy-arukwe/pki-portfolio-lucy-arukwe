# Week 4 — Certificate Formats & Trust Stores

## Focus
This week explores how digital certificates are structured, how they are stored across different systems, and how trust is actually established. It connects to the PKI mental model by showing that identity is defined in the certificate, trust is controlled by the trust store, and verification determines whether that trust is valid.

A key realization is that certificates do not create trust on their own. Trust depends on whether a certificate can be linked back to a root Certificate Authority (CA) that already exists in a system’s trust store.

Different formats such as PEM, DER, and PFX were examined to understand how the same certificate can appear in different forms without changing its identity.

The labs in this week demonstrate that trust is not fixed — it can be added, removed, or manually controlled depending on how validation is performed.

---

## Outcomes

By the end of this week, the following capabilities were developed:

- Interpret the structure of an X.509 certificate and identify key fields such as Subject, Issuer, and validity period
- Differentiate between PEM, DER, and PFX formats and understand when each is used across systems
- Convert certificates between formats using OpenSSL without altering the underlying data
- Inspect and analyze trusted root Certificate Authorities within a system trust store
- Understand how certificate chains are built and validated against trusted roots
- Install and remove root CAs to observe how trust is established and revoked
- Distinguish between system-level trust and manual trust during certificate verification

---

CVI PKI Career Pathway — Foundations Phase
