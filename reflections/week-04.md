# Week 04 Reflection

---

## 

1. What I learnt this week?
This week focused on understanding how certificates actually work beyond just being files. A certificate is a structured trust statement that contains identity, a public key, and a signature from a Certificate Authority. The most important takeaway was that trust is not inside the certificate itself — it depends on whether the certificate chains up to a trusted root CA in the system.

Different certificate formats such as PEM, DER, and PFX were also explored. These formats do not change the certificate data, only how it is encoded and stored. The labs reinforced this by converting between formats and confirming that the certificate remained identical.

2. What concept was most challenging?
The most challenging concept was understanding the difference between system trust and manual verification. Initially, it seemed that removing a root CA should cause all verification to fail. However, verification still succeeded when a CA file was manually provided.

It became clear that OpenSSL can validate certificates using a local CA file instead of the system trust store. This showed that trust depends on how validation is performed, not just what exists on the system.

3. Where does this concept appear in real-world systems?
This concept appears in many real-world systems such as HTTPS connections, enterprise networks, and internal applications. Organizations install their own root CAs on employee devices so that internal services can be trusted automatically.

It is also relevant in security scenarios. If a malicious root CA is installed on a system, it can allow attackers to create trusted certificates and intercept encrypted traffic without warnings. This is why managing the trust store is a critical security responsibility. 

4. How would you explain this topic to a non-technical audience?
A certificate can be compared to an ID card. It shows who someone is, but it only becomes trusted if it is issued by an authority that is already trusted.

The system keeps a list of trusted authorities. When a website presents a certificate, the system checks if it can trace that certificate back to one of those trusted authorities. If it can, the connection is allowed. If not, a warning is shown.

In simple terms, it is not just about having an ID, but about who signed it.

5. What questions remain?
One question that remains is how organizations manage and rotate root CAs at scale without disrupting trust across systems. Another area of interest is how certificate revocation works in practice and how systems check whether a certificate is still valid after it has been issued.

---

## Professional Growth Check

- [x] I documented my work clearly and in my own words
- [x] I used structured formatting in my submission files
- [x] My commit message was meaningful and descriptive

---

*CVI PKI Career Pathway — Foundations Phase*
