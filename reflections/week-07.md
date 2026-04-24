# Week 7 Reflection

---

**1. What did you learn this week?**

This week helped me understand that PKI in enterprise environments is not just about certificates themselves, but about where they exist, how they are managed, and who is responsible for them. I learned that certificates are spread across multiple layers such as web servers, load balancers, APIs, devices, and even code signing systems. 

The biggest takeaway for me was that managing PKI at scale starts with **mapping the environment**, not just fixing individual issues. I also saw how the same tools and steps we used in earlier labs (retrieve, parse, validate, verify) apply directly to real-world infrastructure, just at a much larger scale.

The Week 7 lab reinforced this by analyzing a live enterprise deployment. Using the same diagnostic process, I retrieved a certificate from a live endpoint, parsed its details, validated the certificate chain, and reviewed how TLS was implemented in a production environment. This showed that the tools remain the same — but the focus shifts from troubleshooting problems to understanding how systems are designed.

---

**2. What concept was most challenging?**

The most challenging concept was understanding how certificate ownership works across different teams. It is not always clear who is responsible for a certificate, especially when multiple teams are involved (application, infrastructure, and security).

It was also a bit challenging to fully grasp how TLS termination works at load balancers and how certificates may not even exist on backend servers. This changed how I think about where certificates actually live in an environment.

--- 

**3. Where does this concept appear in real-world systems?**

This appears in almost every modern system that uses secure communication. Public websites use certificates for HTTPS, internal systems use certificates for APIs and device authentication, and cloud platforms manage certificates automatically for services.

It is especially important in environments like healthcare, finance, and cloud infrastructure, where certificate failures can stop communication between systems, block user access, or create security risks. The idea of “shadow certificates” also made me realize how easy it is for unmanaged certificates to exist in production without being noticed.

---

**4. How would you explain this topic to a non-technical audience?**

PKI is a system that helps computers trust each other. A certificate is like an ID card that proves who a system or user is. 

In a simple way, when you visit a website, your browser checks that ID card to make sure it is valid, not expired, and issued by someone it trusts. In large organizations, there are thousands of these “ID cards” across different systems, and managing them becomes complex because they are stored in many different places.

---

**5. What questions remain?**

One question I still have is how organizations keep track of all certificates at scale without missing any, especially the “shadow certificates” that are not part of any inventory.

I am also curious about how automation tools are used to manage certificate lifecycles, especially in environments with microservices and high certificate rotation.

Another question is how organizations decide when to use public CAs versus internal CAs, and how they manage trust between those systems.

What is the career path for someone specializing in PKI? Week 7 introduced the idea of PKI as a career focus, but what does that role actually look like day-to-day? Is it part of a broader infrastructure or security role, or do large organizations have dedicated PKI engineers? What skills beyond OpenSSL and certificate troubleshooting are required — automation, scripting, vendor management, compliance frameworks?

---

## Professional Growth Check

- [x] I documented my work clearly and in my own words
- [x] I used structured formatting in my submission files
- [x] My commit message was meaningful and descriptive

---

*CVI PKI Career Pathway — Foundations Phase*
