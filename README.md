---
## Week 4 — Certificate Formats & Trust Stores
## Focus

This week explores how digital certificates are structured, how they are stored across different 
systems, and how trust is actually established.

A key realization is that certificates do not create trust on their own. Trust depends on whether a 
certificate can be linked back to a root Certificate Authority (CA) that already exists in a system’s trust store.

Different formats such as PEM, DER, and PFX were examined to understand how the same 
certificate can appear in different forms without changing its identity.

The labs in this week demonstrate that trust is not fixed — it can be added, 
removed, or manually controlled depending on how validation is performed.

--- 

## Outcomes

By the end of this week, the following skills were developed:

Convert certificates between PEM, DER, and PFX formats using OpenSSL
Inspect certificate contents and identify key fields such as Subject, Issuer, and validity
Locate and analyze trusted root CAs within a system trust store
Understand how certificate chains are built and validated
Install and remove root CAs to control trust on a system
Differentiate between system trust and manual trust during verification
