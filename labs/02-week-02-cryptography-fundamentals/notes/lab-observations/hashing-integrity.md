## Part 3 — Observations

## Why the Hash Changed After a Small Modification

The hash changed completely after adding the word “tampered” to the file because cryptographic hash functions are designed with the avalanche effect. 
This means that even a very small change to the input data produces a completely different hash value. 
Hash functions process the entire content of a file, so any modification alters the resulting digest.

## Why Hashing Does NOT Provide Confidentiality

Hashing does not provide confidentiality because it does not encrypt or hide the original data. 
A hash function simply converts data into a fixed-length digest that represents the content. The original file remains readable unless encryption is also applied. 
Hashing is mainly used for verification, not for protecting data from being viewed.

## What Security Property Hashing Provides

Hashing provides the security property of integrity. By comparing hash values, users can verify whether a file has been modified. 
If even a single character in the file changes, the resulting hash will also change, indicating that the data has been altered.

## Where Hashing is Used in PKI Systems

Hashing plays an important role in Public Key Infrastructure (PKI). It is used in several areas, including:

Certificate signatures – Certificate Authorities hash certificate data before signing it with their private key.

File integrity validation – Hash values are used to verify that files have not been tampered with during storage or transmission.

Code signing – Software publishers hash program files before signing them to ensure that users receive authentic and unmodified software.
