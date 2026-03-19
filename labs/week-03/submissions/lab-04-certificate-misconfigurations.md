# Lab 04 — Detect Certificate Misconfigurations

## Overview
Briefly describe what this lab was about in your own words.
What PKI concept were you investigating?

---

## Scenario 1 — Missing Subject Alternative Name

**Would modern browsers trust this certificate?**
[Your answer]

**Analysis:**
[Explain why SAN is required, why CN is not sufficient, and what error users would see]

---

## Scenario 2 — Incorrect Extended Key Usage

**Would a browser accept this certificate for a web server?**
[Your answer]

**Analysis:**
[Explain what EKU defines, what value is required for HTTPS, and what error users would see]

---

## Scenario 3 — Expired Certificate

**What happens if this certificate is used today?**
[Your answer]

**Analysis:**
[Explain why expiration fails validation, why lifecycle management matters, and what users would see]

---

## Scenario 4 — Missing Intermediate Certificate

**What error would a browser likely display?**
[Your answer]

**Analysis:**
[Explain how chains establish trust, why servers must include intermediates, and why browser behavior varies]

---

## Key Takeaway
In 2-3 sentences, explain why certificate misconfiguration is one of the most common causes of PKI outages.
