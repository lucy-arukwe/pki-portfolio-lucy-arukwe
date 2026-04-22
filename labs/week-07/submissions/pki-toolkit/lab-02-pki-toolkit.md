# My PKI Toolkit
**CVI PKI Career Pathway — Phase 1 Foundations**
Completed: April, 2026

---

## About This Document

This toolkit documents every command-line tool, utility, and service used across seven weeks of hands-on PKI labs. Each entry includes a description of what the tool does, when to reach for it, and a real command example from completed lab work. This document serves as both a personal reference and a portfolio artifact demonstrating practical PKI operations experience.

---

## Core Command-Line Tools

### openssl x509
Parse and Inspect X.509 Certificates
# What it does:
Reads X.509 certificate files and displays certificate fields in a human-readable format. In simple terms, it lets you “open” a certificate and see important details like who issued it, who it belongs to, and when it expires. It can also perform operations like format conversion and field extraction. This is Step 2 of the PKI Diagnostic Framework.

# When to use it:
After retrieving a certificate from a server, when validating certificate fields, checking validity periods, extracting Subject Alternative Names, or converting certificate formats.

Example command from my labs:
# Parse certificate in human-readable format
`openssl x509 -in [leaf_certificate.pem] -text -noout`

# Extract specific fields (OCSP URL)
`openssl x509 -in [leaf_certificate.pem] -text -noout | grep -A4 "Authority Information Access"`

# Convert PEM to DER format
`openssl x509 -in [leaf_certificate.pem] -outform DER -out [certificate.der]`

# Verify DER certificate
`openssl x509 -in [certificate.der] -inform DER -text -noout`

What the output tells you:
Displays validity dates (Not Before/Not After), Subject, Issuer, Subject Alternative Names, Key Usage, and signature algorithm. These details help confirm whether the certificate is valid, matches the expected hostname, and was issued by a trusted authority.

Phase 1 source: Week 3 — Certificate Anatomy; Week 4 — Certificate Formats; Week 5 — OCSP Revocation; Week 6 — PKI Incident Diagnosis (all labs); Week 7 — Enterprise Certificate Analysis

---

### openssl s_client
What it does:
Establishes a TLS connection to a server and displays the TLS handshake details, including the certificates presented by the server. In simple terms, it lets you “connect to a website from the command line” and see exactly what certificate it is using. This is Step 1 of the PKI Diagnostic Framework.

When to use it:
Use this whenever you need to retrieve a certificate from a live system for inspection, diagnosis, or validation. It is typically the first step in any PKI troubleshooting process.

Example command from my labs:
# Basic connection and certificate retrieval
openssl s_client -connect [hostname]:443 -showcerts

# Retrieve and save the leaf certificate
echo | openssl s_client -connect [hostname]:443 2>/dev/null | openssl x509 -out [leaf_certificate.pem]

# Retrieve from test endpoint
openssl s_client -connect [test_endpoint]:443 -showcerts </dev/null 2>/dev/null | openssl x509 -outform PEM > [certificate.pem]

What the output tells you:
Shows the full certificate chain presented by the server, the TLS version used, the cipher suite selected, and the verification status. The -showcerts option ensures all certificates in the chain are displayed, not just the main one.

Phase 1 source: Week 3 — Certificate Anatomy; Week 5 — OCSP Revocation; Week 6 — PKI Incident Diagnosis; Week 7 — Enterprise Certificate Analysis

### openssl req
openssl req  
Generate Certificate Signing Requests (CSRs)

What it does:
Creates a Certificate Signing Request (CSR) that includes a public key and identity information (Subject DN). In simple terms, it prepares a request that asks a Certificate Authority (CA) to issue a certificate for you, while proving that you own the private key.

When to use it:
When requesting a new certificate from a CA for testing or production use. It is also used when generating self-signed certificates for lab environments. The CSR proves ownership of the private key without exposing it.

Example command from my labs:

openssl req -new -x509 -key [private_key.pem] -out [certificate.pem] -days 365 -subj "/CN=[common_name]/O=[organization]/C=[country]"

What the output tells you:
Produces either a CSR file or a self-signed certificate (when using the -x509 flag). The Subject fields identify who the certificate belongs to.

Phase 1 source: Week 4 — Certificate Formats (PFX Bundle Creation)

### openssl verify
Validate Certificate Chains

What it does:
Verifies that a certificate chain is valid by checking signatures from the leaf certificate up to the root certificate, ensuring each certificate is properly signed by its issuer. It confirms whether a certificate can be trusted by tracing it back to a known and trusted authority. This is Step 3 of the PKI Diagnostic Framework.

When to use it:
After retrieving a certificate and its chain, when diagnosing trust failures, or when confirming that a certificate was issued by a trusted Certificate Authority.

Example command from my labs:

openssl verify -CAfile [root_certificate.pem] -untrusted [intermediate_certificate.pem] [server_certificate.pem]

What the output tells you:
Returns "OK" if the chain is valid. If not, it provides an error showing where validation fails (for example: expired certificate, untrusted root, or missing intermediate).

Phase 1 source: Week 3 — Certificate Chain Validation

### openssl ocsp
Query OCSP Responders for Revocation Status

What it does:
Sends an OCSP (Online Certificate Status Protocol) request to an OCSP responder to check whether a certificate has been revoked. In simple terms, it asks the issuing authority, “Is this certificate still valid, or has it been cancelled?” This is Step 4 of the PKI Diagnostic Framework.

When to use it:
When checking the revocation status of a certificate, diagnosing trust issues related to revoked certificates, or confirming that a certificate is still trusted by the issuing Certificate Authority.

Example command from my labs:

openssl ocsp \
  -issuer [issuer_certificate.pem] \
  -cert [leaf_certificate.pem] \
  -url [OCSP_URL] \
  -resp_text \
  -noverify \
  2>/dev/null | tee [ocsp_response.txt]

What the output tells you:
Returns the certificate status as "good" (valid), "revoked" (no longer trusted), or "unknown" (not recognized by the responder). It may also include the time of revocation if applicable.

Phase 1 source: Week 5 — OCSP Revocation Status; Week 6 — PKI Incident Diagnosis (Revocation Checks)

### openssl pkcs12
Create and Inspect PFX/PKCS#12 Bundles

What it does:
Bundles a private key and certificate (and optionally intermediate certificates) into a single encrypted PKCS#12 file (.pfx or .p12). It packages everything needed for a certificate—identity and secret key—into one secure file that can be moved between systems.

When to use it:
When creating a portable certificate bundle for import into Windows systems, browsers, or applications that require both the certificate and private key in a single file.

Example command from my labs:

# Create PFX bundle
openssl pkcs12 -export -out [bundle.pfx] -inkey [private_key.pem] -in [certificate.pem]

# Verify PFX bundle
openssl pkcs12 -info -in [bundle.pfx] -noout

What the output tells you:
The export command creates an encrypted bundle protected by a password. The info command displays the contents of the bundle (certificates and keys included) without exposing the private key.

Phase 1 source: Week 4 — Certificate Format Conversion (PFX Creation)


### openssl genrsa
What it does:
Generates an RSA private key of a specified bit length. In simple terms, this creates a secure “secret key” that is used to prove identity or encrypt data. This private key is the foundation of asymmetric cryptography and must be kept secure.

When to use it:
When creating a new key pair for certificate signing requests, digital signatures, or encryption operations. It is commonly used during initial PKI setup or when generating test certificates.

Example command from my labs:

openssl genrsa -out [private_key.pem] 2048

What the output tells you:
Produces a private key file. The key size (for example, 2048 or 4096 bits) determines the strength of the cryptographic operations performed with this key.

Phase 1 source: Week 2 — Cryptography Fundamentals (Key Generation)

openssl rsa — Extract Public Key from Private Key

What it does:
Reads an RSA private key and extracts the corresponding public key. In simple terms, it creates a version of the key that can be safely shared with others, while keeping the private key secure.

When to use it:
After generating a private key, when the public key needs to be shared for signature verification or encryption purposes.

Example command from my labs:

openssl rsa -in [private_key.pem] -pubout -out [public_key.pem]

What the output tells you:
Produces the public key that corresponds to the private key. This public key can be used to verify signatures created with the private key or to encrypt data.

Phase 1 source: Week 2 — Cryptography Fundamentals (Key Generation)

### openssl enc — Symmetric Encryption and Decryption

What it does:
Performs symmetric encryption or decryption using algorithms such as AES. In simple terms, it locks a file using a password and can later unlock it using the same password. This is used to keep data confidential.

When to use it:
When encrypting sensitive data at rest or testing encryption workflows. It is used for protecting data, not for verifying identity.

Example command from my labs:

# Encryption
openssl enc -aes-256-cbc -salt -pbkdf2 -in [plaintext.txt] -out [encrypted.bin]

# Decryption
openssl enc -d -aes-256-cbc -pbkdf2 -in [encrypted.bin] -out [decrypted.txt]

What the output tells you:
Encryption produces unreadable data (ciphertext). Decryption either restores the original file (if the correct password is used) or fails if the password is incorrect or the data is corrupted.

Phase 1 source: Week 2 — Cryptography Fundamentals (Symmetric Encryption)

### openssl dgst — Hash Files and Create/Verify Digital Signatures

What it does:
Computes cryptographic hashes (such as SHA-256 or SHA-512) of files and can also create or verify digital signatures using a private/public key pair. In simple terms, it creates a unique “fingerprint” of a file or proves who created it and that it hasn’t been changed.

When to use it:
When verifying file integrity (hashing), creating signatures to prove authenticity, or confirming that a file was signed by a trusted source.

Example command from my labs:

# Generate hash
openssl dgst -sha256 [input_file]

# Create signature
openssl dgst -sha256 -sign [private_key.pem] -out [signature.sig] [input_file]

# Verify signature
openssl dgst -sha256 -verify [public_key.pem] -signature [signature.sig] [input_file]

What the output tells you:
The hash output is a fixed-length value that changes if the file is modified. Signature verification returns "Verified OK" if the file is authentic, or "Verification Failure" if it has been altered or signed with the wrong key.

Phase 1 source: Week 2 — Cryptography Fundamentals (Hashing and Digital Signatures)

---

## Network and Inspection Tools

### curl
Retrieve HTTP Headers and Test TLS Endpoints

What it does:
Sends HTTP or HTTPS requests to a server and displays the response headers. It lets you “ask” a website how it is configured and what technologies or security settings it is using, eg Used to inspect server behavior, check for security headers, and identify CDN or load balancer indicators.

When to use it:
When determining where TLS terminates (such as a CDN, load balancer, or application server), checking for security headers like HSTS, or analyzing server responses during troubleshooting.

Example command from my labs:

# Check for CDN and security headers
curl -sI https://[hostname] | grep -i "server\|cf-ray\|x-cache\|via\|x-amz"

# Check HSTS configuration
curl -sI https://[hostname] | grep -i "strict-transport-security"

# Download intermediate certificate from CA Issuers extension
curl -s "[CA_ISSUERS_URL]" -o [intermediate_certificate.der]

What the output tells you:
Response headers reveal server type, CDN presence (for example CloudFront or Cloudflare), and security policies such as HSTS. This helps identify how traffic is handled and where TLS is being terminated.

Phase 1 source: Week 6 — PKI Incident Diagnosis (Untrusted Root, Broken Chain); Week 7 — Enterprise Certificate Analysis (TLS Termination)

### diff — Compare File Contents for Integrity Verification

What it does:
Compares two files byte-by-byte and reports any differences between them. In simple terms, it answers the question: “Are these two files exactly the same?”

When to use it:
After encryption and decryption workflows, to confirm that the decrypted output matches the original file exactly.

Example command from my labs:

diff [original_file.txt] [decrypted_file.txt]

What the output tells you:
No output means the files are identical (successful encryption/decryption process). Any output indicates differences, which may mean corruption or a failed decryption.

Phase 1 source: Week 2 — Cryptography Fundamentals (Symmetric Encryption)

Browser Certificate Inspector — Inspect Certificates in the Browser

What it does:
Displays certificate details for HTTPS websites directly in the browser. In simple terms, it lets you quickly view a website’s certificate by clicking the padlock icon in the address bar.

When to use it:
For quick visual inspection of a website’s certificate without using command-line tools. It is useful for initial checks before performing deeper analysis with tools like OpenSSL.

Example workflow from my labs:
Click padlock icon → View certificate → Open Details tab → Review Subject, Issuer, Validity, and Certificate Hierarchy

What the output tells you:
Shows the certificate chain, validity dates, Subject Alternative Names (SANs), and whether the browser trusts the certificate. It provides a quick overview, but does not show as much detail as command-line inspection.

Phase 1 source: Week 1, Week 2, Week 3 — Certificate Anatomy; Week 7 — Enterprise Certificate Analysis (Initial Reconnaissance)

---

## Reference and Analysis Services

SSL Labs SSL Server Test 
External TLS Configuration Analysis

What it does:
Performs a comprehensive analysis of a website’s TLS/SSL configuration. In simple terms, it scans a public website and gives a detailed report on how securely it is set up.

When to use it:
When assessing the overall TLS security posture of a website, identifying weak configurations, or validating that security best practices (such as HSTS and modern TLS versions) are properly implemented.

Example usage from my labs:
Go to https://www.ssllabs.com/ssltest/
 → Enter [hostname] → Run analysis → Review the grade and detailed results

What the output tells you:
Provides a letter grade (such as A+ or B) based on security strength. It also shows supported TLS versions, preferred cipher suites, certificate chain validity, HSTS configuration, and OCSP stapling status.

Phase 1 source: Week 7 — Enterprise Certificate Analysis (attempted; target opted out of testing)

### crt.sh — Certificate Transparency Search

What it does:
Searches public Certificate Transparency logs to find all certificates issued for a given domain. In simple terms, it lets you see the full history of certificates that have been created for a website.

When to use it:
When investigating certificate issuance patterns, detecting unauthorized or unexpected certificates, identifying changes in Certificate Authorities, or monitoring for phishing domains that may be impersonating a brand.

Example usage from my labs:
Go to https://crt.sh
 → Search for [domain_name] → Review historical certificate issuances and issuer patterns

What the output tells you:
Shows a complete list of certificates issued for the domain, including issuer names, validity periods, and Subject Alternative Names. This can reveal changes in issuing authorities or highlight suspicious or unexpected certificates.

Phase 1 source: Week 7 — Enterprise Certificate Analysis (Certificate Transparency Analysis)

---

## Phase 1 Skills Summary

Phase 1 labs built hands-on experience with the complete PKI operations toolkit — from foundational cryptography (key generation, encryption, hashing, and digital signatures) through certificate lifecycle management (creation, inspection, format conversion, and chain validation) to real-world incident diagnosis (expired certificates, trust failures, and revocation checking).

A key outcome was developing a structured diagnostic approach rather than relying on guesswork. By consistently following the retrieve → parse → validate → verify process, it became possible to break down TLS and certificate issues in a clear and repeatable way.

By Week 7, the focus shifted from fixing issues to analyzing enterprise environments, using the same tools to assess certificate deployments, identify risks, and understand how secure communication is implemented at scale.

All tools documented in this toolkit were used in real lab scenarios, not theoretical exercises. This reflects practical PKI troubleshooting skills that directly translate to real-world environments where secure communication depends on properly configured and trusted certificates.


