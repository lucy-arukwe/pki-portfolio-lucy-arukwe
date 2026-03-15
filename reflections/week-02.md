---

## Week 02 Reflection: Cryptography Fundamentals

## 1. What did you learn this week?

This week helped me understand the difference between confidentiality, integrity, and authenticity, and how they work together in security systems.

` Confidentiality ` is about keeping information private so that only authorized people can see it. In the symmetric encryption lab, I saw how a file can be encrypted using a 
password so that it becomes unreadable unless the correct key is used to decrypt it. That showed me how encryption protects data from being exposed.

` Integrity ` is about making sure data has not been changed or tampered with. In the hashing lab, I generated a SHA-256 hash of a file, then modified the file slightly and generated another hash. 
Even though the change was very small, the new hash value was completely different. That helped me understand how hashing can quickly detect if something has been altered.

` Authenticity ` is about proving who created or sent something. In the digital signature lab, I created a key pair, signed a file using the private key, and verified the signature using the public key. 
When the file was unchanged, the verification succeeded. But after I modified the file, the verification failed. That showed how digital signatures can confirm both the source of the file and whether it has been modified.
Overall, this week helped me see how these three ideas work together to secure information.


## 2. What concept was most challenging?

The concept that was most challenging for me was understanding why hashing cannot be reversed.
At first, I thought of hashing like encryption, where something could be decoded back to the original data. But I realized that hashing works differently. It creates a unique fingerprint of the data rather than something meant to be decrypted.
It was also interesting to see why digital signatures fail after tampering. When I first ran the verification command, it worked fine. But after adding just one word to the file, the verification failed. 
That helped me understand that even a tiny change creates a completely different hash, which causes the signature verification to fail.
Seeing it happen in the lab made the concept much clearer than just reading about it.

## 3. Where does this appear in real-world systems?

These concepts appear in many real-world systems. 
One example is HTTPS connections. When you visit a secure website, your browser checks the website’s certificate to confirm it was signed by a trusted Certificate Authority. This uses digital signatures to verify authenticity and integrity.

Another example is software updates. When companies release updates, they often sign the software using digital signatures. Your computer verifies that signature before installing the update to make sure it has not been modified by attackers.

Code signing is another place where this is used. Developers sign their software so that users know the program came from the original developer and has not been changed.

Organizations also use PKI systems internally to manage secure communication, authentication, and access control.

## 4. How would you explain this topic to a non-technical audience?

If I had to explain this to someone without a technical background, I would describe it like this.

`Encryption` is like locking a message in a safe. Only someone with the correct key can open it and read the contents.

`Hashing` is like creating a fingerprint for a file. If the file changes even a little, the fingerprint changes completely. That helps detect tampering.

`Digital` signatures are like signing a document with a unique signature that only you can create. Anyone who has your public key can check that signature and confirm the document really came from you and has not been altered.

Together, these tools help protect information and verify that it is trustworthy.

## 5. What questions remain?

After completing the labs, I still have some questions about how these systems are implemented at a larger scale.

For example, I would like to understand more about key management and how organizations securely store and rotate private keys.

I am also curious about how trust chains work in Certificate Authorities and how browsers decide which certificates to trust.

Another area I would like to explore more is how TLS certificates are issued and how they are verified during secure web connections.

These are topics I hope to learn more about in the upcoming weeks.
