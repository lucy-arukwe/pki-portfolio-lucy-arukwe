# Lab —  Hashing & Integrity

## Overview
This lab focused on understanding how cryptographic hashing works and why it is used to verify `integrity.` A SHA-256 hash of a file was generated, the file was modified, and then hashed again to observe how even a small change produces a completely different output. The core concept explored is how hashing detects tampering and ensure data has not been altered.

---

## Environment
Document the environment used to complete the lab.

- Operating System: Windows 11 
- Terminal Used: Git Bash (MINGW64)
- OpenSSL Version: OpenSSL 3.5.5 27 Jan 2026  

---

## Steps Performed
1. Created the `submissions/hashes/` directory inside the Week 2 lab folder.
2. Created a plaintext file containing "Week 2 Hashing Lab - CVI" as the test message.
3. Generated a SHA-256 hash of the file using OpenSSL and saved the output to `message.sha256.txt`.
4. Appended the word "tampered" to the original file to simulate tampering.
5. Generated a new SHA-256 hash of the modified file and saved it to `message_tampered.sha256.txt`.
6. Compared both hash outputs to observe how drastically different they were.

---

## Results
Original hash (before tampering):

SHA2-256(message.txt)= c628ac1a6cdb7ef27fd7e62c6e3208626ca24f76052d182a77e45e19c7546b93


Hash after adding the word "tampered":

SHA2-256(message.txt)= be2d7fb9708376211adb5b6034965a4b94c78ff42b14e7cbaf63fa02adbb21aa

The two hashes share no visible similarity, even though the file content is almost identical.
This confirmed that even a small change in input results in a completely different hash value.  

Evidence stored in `assets/screenshots/` 

**How to embed an image:**

**Option A — Terminal / Local Editor**

Save your screenshot to `assets/screenshots/` in your repo, then reference it using a relative path from your submission file:

```markdown
![Description of your screenshot](../../../assets/screenshots/your-filename.png)
```

> The `../../../` moves up three levels: `submissions/` → `week-03/` → `labs/` → repo root, then into `assets/screenshots/`.

**Option B — GitHub Web (Easiest)**

Open your `.md` file on GitHub, click the pencil icon to edit, then **drag and drop your image directly into the text editor**. GitHub will upload it automatically and insert the correct link for you.

Example of what an embedded image looks like:

```markdown
![Certificate output showing SAN field](../../../assets/screenshots/san-field.png)
```

---

## Key Findings
- A small change to the file produced a completely different hash value, demonstrating the avalanche effect.
- Hash outputs are always fixed in length regardless of the size of the input.
- Hashing is a one-way process and cannot be reversed to recover the original data.


## Explanation
- Hashing provides integrity by allowing verification that data has not been altered. If two hash values match, the content is identical. If they differ, even slightly, it indicates that the data has been modified.
- Hashing does not provide confidentiality because the original data remains readable. It ensures data integrity, not secrecy (Mainly used for verification)
- In PKI systems, hashing is used in digital signatures, certificate validation, and code signing, where software integrity is verified using hashes.
A certificate is signed based on a hash of its contents, and any change to the certificate will result in a different hash, causing signature verification to fail.

---

## Challenges / Troubleshooting

- Attempted to view the file using `cat` before creating it, which resulted in a "no such file or directory" error. This was resolved by creating the file first using the `echo` command.
- No major issues were encountered during this lab. The main focus was ensuring correct file paths were used when generating and saving the hash outputs.

---

## Artifacts

- `labs/week-02/02-week-02-cryptography-fundamentals/submissions/hashes/message.txt` — original test message
- `labs/week-02/02-week-02-cryptography-fundamentals/submissions/hashes/message.sha256.txt` — SHA-256 hash of the original file
- `labs/week-02/02-week-02-cryptography-fundamentals/submissions/hashes/message_tampered.sha256.txt` — SHA-256 hash after tampering
- `labs/week-02/02-week-02-cryptography-fundamentals/submissions/hashes/lab-02-hashing-integrity.md` — this write-up


---

*CVI PKI Career Pathway — Foundations Phase*
