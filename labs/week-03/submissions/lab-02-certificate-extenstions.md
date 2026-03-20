# Lab 02 — Investigate Certificate Extensions

## Overview

This lab involved inspecting the X.509v3 extensions of the Google 
certificate retrieved in Lab 01. Extensions define how a certificate 
can be used, what domains it covers, and whether it has the authority 
to issue other certificates. This exercise builds understanding of how 
extensions control certificate behavior and impact TLS validation.
---

## Environment
- OS: Windows 11
- Terminal used: Git Bash (MINGW64) 
- OpenSSL version: ` OpenSSL 3.5.5 27 Jan 2026 `

---

## Extensions Found

### Subject Alternative Name (SAN)
DNS:*.google.com, DNS:*.appengine.google.com, DNS:*.bdn.dev,
DNS:*.origin-test.bdn.dev, DNS:*.cloud.google.com,
DNS:*.crowdsource.google.com, DNS:*.datacompute.google.com,
DNS:*.google.ca, DNS:*.google.cl, DNS:*.google.co.in,
DNS:*.google.co.jp, DNS:*.google.co.uk, DNS:*.google.com.au,
DNS:*.google.com.br, DNS:*.google.com.co, DNS:*.google.com.mx,
DNS:*.google.com.tr, DNS:*.google.com.vn, DNS:*.google.de,
DNS:*.google.es, DNS:*.google.fr, DNS:*.google.hu,
DNS:*.google.it, DNS:*.google.nl, DNS:*.google.pl,
DNS:*.google.pt, DNS:*.googleapis.cn, DNS:*.gstatic.cn,
DNS:*.gstatic-cn.com, DNS:google.com

### Key Usage:
Digital Signature

### Extended Key Usage (EKU):
TLS Web Server Authentication

### Basic Constraints:
CA:FALSE

---

## Observations

1. What domains appear in the SAN field?
The SAN field contains a large number of Google-related domains. It includes domains like `.google.com, ` `.youtube.com, ` `.google.ca, ` and many other country-specific and service-based domains.
This shows that the certificate can be used for multiple Google services, not just one website.

2. What operations are permitted by Key Usage?
The Key Usage shows that the certificate is allowed to perform Digital Signature.
This means it can be used to verify identity and ensure that data has not been tampered with during communication.
 
3. What applications are authorized by EKU?
The certificate is authorized for TLS Web Server Authentication, meaning it is used to verify the identity of a web server during a secure TLS connection.

4. Can this certificate issue other certificates? How do you know?
No, this certificate cannot issue other certificates.
This is because the Basic Constraints field says CA:FALSE, which means it is not a Certificate Authority. which means it is a regular (end-entity/leaf) certificate and does not have permission to sign or issue other certificates.

5. Why are these extensions important for TLS validation?
Certificate extensions are important because they provide additional instructions to systems on how a certificate should be used. They define what cryptographic operations the key can perform, what services the certificate is valid for, what identity it represents, and whether it can issue other certificates. These extensions help enforce security by ensuring certificates are restricted and used only for their intended purposes during TLS validation.
