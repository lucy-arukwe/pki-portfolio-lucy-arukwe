# Lab — Symmetric Encryption (Confidentiality)

## Goal

This lab builds operational understanding of symmetric encryption and the security property of confidentiality.

You will:

- Encrypt a file using AES
- Decrypt it back to its original form
- Observe how ciphertext differs from plaintext
- Understand why symmetric encryption is used in TLS for bulk data protection

---

## Part 1 — Setup

### Prerequisites

- OpenSSL installed
- Access to a local terminal (Mac Terminal, Git Bash, or WSL)
- Your Week 2 portfolio folder created

All commands must be executed locally.  
GitHub’s web interface cannot run OpenSSL commands.

---

## Part 2 — Execution Steps

### Step 1 — Create Artifact Directory
From the root of your directory:
mkdir -p labs/02-week-02-cryptography-fundamentals/submissions/encrypted

### Step 2 — Create a Plaintext File
echo "Week 2 Symmetric Encryption Lab - CVI" > labs/02-week-02-cryptography-fundamentals/submissions/encrypted/plaintext.txt

Open the file and confirm it is readable.

### Step 3 — Encrypt the File
Use AES-256 encryption with password-based key derivation:

openssl enc -aes-256-cbc -salt -pbkdf2 \
  -in labs/02-week-02-cryptography-fundamentals/submissions/encrypted/plaintext.txt \
  -out labs/02-week-02-cryptography-fundamentals/submissions/encrypted/plaintext.txt.enc

You will be prompted for a password.

Observe:
- The encrypted file is unreadable.
- It contains binary ciphertext.

### Step 4 — Decrypt the File
openssl enc -d -aes-256-cbc -pbkdf2 \
  -in labs/02-week-02-cryptography-fundamentals/submissions/encrypted/plaintext.txt.enc \
  -out labs/02-week-02-cryptography-fundamentals/submissions/encrypted/plaintext.decrypted.txt

Enter the same password used during encryption.

### Step 5 — Verify Integrity of Decrypted File
diff labs/02-week-02-cryptography-fundamentals/submissions/encrypted/plaintext.txt \
     labs/02-week-02-cryptography-fundamentals/submissions/encrypted/plaintext.decrypted.txt

If no output appears, the files are identical.

## Part 3 — Observations
Document the following in your Week 2 lab notes:
- Why the encrypted file is unreadable
- What would happen if the wrong password were used
- What security property symmetric encryption provides
- Why TLS uses symmetric encryption for data transfer

### Submission (Portfolio Repo)
Ensure the following files exist:

labs/02-week-02-cryptography-fundamentals/submissions/encrypted/
  plaintext.txt
  plaintext.txt.enc
  plaintext.decrypted.txt

Commit and push your changes.

Do NOT commit any passwords.

## Stretch (Optional)
Try decrypting the file using an incorrect password.

What error message do you receive?
Why does decryption fail?

CVI PKI Career Pathway — Foundations Phase
