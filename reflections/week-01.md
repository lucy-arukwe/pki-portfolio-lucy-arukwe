---

## Week 01 Reflection
1. what is digital trust?

Digital trust is the confidence that a system, website, or service is who it claims to be and can be interacted with safely. It is what allows users to share sensitive information like passwords, banking details, or personal data without fear. This trust is not based on guesswork but on systems like encryption, certificates, and verification processes working behind the scenes. For example, when a browser shows a padlock icon, it is indicating that a secure connection has been established. Digital trust depends on verifying identity, ensuring data integrity, and protecting confidentiality. Without it, users would have no reliable way to distinguish between legitimate services and malicious ones. In simple terms, digital trust is what makes the internet usable and safe for everyday activities.

2. Difference between identity and authentication?

Identity refers to who or what something claims to be, such as a website, user, or system. Authentication, on the other hand, is the process of verifying that identity is actually true. For example, a user may claim an identity by entering a username, but authentication happens when they provide a correct password or a second factor like a code. In the context of websites, a certificate represents the identity of the server. The browser then authenticates that identity by checking if the certificate is valid and signed by a trusted authority. Identity is simply a claim, while authentication is the proof of that claim. Without authentication, identity cannot be trusted. Both concepts work together to establish secure communication in digital systems.

3. Public key vs Private key

A public key and private key work together as a pair, but they serve different purposes. The public key is meant to be shared with others, while the private key must be kept secret. If someone wants to send secure data, they use the public key to encrypt it. Once encrypted, only the matching private key can decrypt and read the information. This means even if someone intercepts the data, they cannot understand it without the private key. The private key is also used to create digital signatures, proving that something came from a trusted source. The public key can then be used to verify that signature. In simple terms, the public key locks the message, and the private key unlocks it. Keeping the private key secure is critical, because if it is exposed, the whole system is compromised.

4. What does a certificate prove? What does it not prove?

A certificate proves that a public key belongs to a specific identity, such as a website or organization. It also proves that this identity has been verified by a trusted Certificate Authority. When a browser checks a certificate, it confirms that the certificate is valid, not expired, and signed by a trusted authority. This helps ensure that users are communicating with the correct server and not an attacker. However, a certificate does not guarantee that the website itself is safe or free from malicious content. It does not prove that the organization is trustworthy in behavior, only that its identity has been verified. For example, a phishing site can still have a valid certificate. A certificate is about identity and encryption, not reputation or intent. Understanding this difference is important when evaluating online security.

5.PKI in ones daily life

PKI appeared in several everyday activities without being immediately obvious. When visiting websites like GitHub or logging into online accounts, HTTPS connections were used, which rely on certificates and encryption. The padlock icon in the browser is a direct result of PKI working in the background. Email services also use PKI concepts for secure communication, even if it is not always visible to the user. Software updates and app downloads often use code signing certificates to verify authenticity. Even Wi-Fi networks and VPN connections can use certificates for secure access. These systems rely on PKI to ensure data is protected and identities are verified. This week made it clear that PKI is deeply integrated into daily digital interactions.

6. What concept is still unclear?

One concept that is still unclear is how trust is initially established between Certificate Authorities and operating systems or browsers. It raises the question of how certain root certificates are chosen to be trusted by default. Another area that needs more understanding is certificate revocation and how quickly revoked certificates are recognized across different systems. The role of Certificate Transparency logs and how they help detect misissued certificates is also something worth exploring further. Additionally, the differences between public CA certificates and internal enterprise PKI systems are not yet fully clear. Understanding when and why an organization would use one over the other is important. These areas will require deeper study to fully understand how trust is maintained at scale.
