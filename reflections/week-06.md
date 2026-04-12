# Week 6 Reflection

---

## Prompts

1. What was learned this week?
This week focused on diagnosing certificate failures using a structured approach instead of guessing. The PKI Diagnostic Framework made it clear that most TLS issues fall into a few categories, such as expiration, chain problems, hostname mismatches, trust issues, or revocation. Working through each step in order helped show how to identify the exact cause instead of relying on trial and error.

2. What concept was most challenging?
The most challenging part was understanding the difference between a valid certificate and a trusted connection. A certificate can look completely fine on its own but still fail if the chain is incomplete or if the client does not trust the root CA. It took some careful thinking to separate what belongs to the certificate itself and what depends on the client environment.

3. Where does this concept appear in real-world systems?
This shows up in many real systems, especially internal enterprise applications. Examples include internal portals, healthcare systems, and company tools that rely on internal Certificate Authorities. It is also common in cloud environments and load-balanced systems, where certificate deployment or trust configuration can differ across environments.

5. How would you explain this topic to a non-technical audience?
Secure connections depend on three things: identity, trust, and verification. The system needs to prove who it is, the device needs to trust that proof, and everything must still be valid at the time of connection. If any one of these is missing, the connection is blocked to protect the user.

Skipping steps is like a security guard checking that a badge looks real but not scanning it to confirm it’s valid. It might look fine, but it could still be the wrong badge. The framework ensures the check goes all the way, so the real issue is found.
5. What questions remain?

---

## Professional Growth Check

- [x] I documented my work clearly and in my own words
- [x] I used structured formatting in my submission files
- [x] My commit message was meaningful and descriptive

---

*CVI PKI Career Pathway — Foundations Phase*
