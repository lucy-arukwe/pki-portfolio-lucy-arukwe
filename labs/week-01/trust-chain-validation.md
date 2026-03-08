# Week 01 Mini Lab — Trust Chain Validation

## Screenshot Evidence

Capture a screenshot of the Certification Path (certificate chain) from your browser.

Save it as:

assets/screenshots/week-01/trust-chain-validation.png

Embed the screenshot below:

![Trust Chain Validation](../../assets/screenshots/week-01/trust-chain-validation.png)


## Website Information

**Website inspected:**  
http://www.scotiabank.com

---

## Certificate Chain Breakdown

**Leaf (Server) Certificate**  
http://www.scotiabank.com

**Intermediate Certificate Authority**
DigiCert Global G2 TLS RSA SHA256 2020 CA1

**Root Certificate Authority (Trust Anchor)**
DigiCert Global Root G2

---

## Trust Anchor Verification

Is the Root CA marked as trusted by your system?

Yes

If yes, explain where that trust comes from (OS/browser root store).
The trust comes from the operating system and browser root certificate store, which contains trusted certificate authorities.

If no, explain what warning or behavior occurred.

---

## Observations

Document three observations about the certificate.

### Observation 1
The certificate chain shows how the website certificate links to a trusted root certificate.

### Observation 2
An intermediate certificate authority is used instead of issuing certificates directly from the root CA.

### Observation 3
The browser automatically validates the certificate chain before establishing a secure HTTPS connection, and nd no security warning appears because the certificate is trusted.

---

## Reflection

In 3–5 sentences, explain:
- Why the Root certificate is called a trust anchor
- How validation walks the certificate chain
- What would happen if the Root CA were not trusted

Use your own words.

The root certificate is called a trust anchor because it is the starting point of trust in the certificate validation process. When a browser validates a website certificate, it checks the certificate chain from the server certificate to the intermediate certificate and finally to the trusted root certificate. If the root certificate is trusted by the system, the connection is considered secure. If the root certificate were not trusted, the browser would show a security warning.
