
## Why verification succeeds before tampering

Verification succeeds because the file is still exactly the same as it was when it was signed. 
The public key can confirm that the signature matches the file content.

## Why verification fails after modification

Verification fails because the file was changed after signing. Even a small change produces a different hash, 
so the original signature no longer matches the modified file.

## Why digital signatures require both hashing and asymmetric cryptography

Digital signatures use hashing to create a fixed-size digest of the file and asymmetric cryptography to sign that digest with the private key. 
The public key is then used to verify that signature.

## How this relates to certificate signing in PKI

In PKI, Certificate Authorities sign certificate data using their private key. 
Systems then use the CA’s public key to verify that the certificate is authentic and has not been altered.
