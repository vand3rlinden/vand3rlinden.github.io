---
title: "DNSSEC and DANE explained"
date: 2024-01-13T21:51:39+01:00
draft: false
categories: ["DNS"]
cover: 
  image: /images/dnssec-dane-explained/dnssec-dane-explained-front.png
---

> _In this post, you will learn how DNSSEC and DANE work together and how to configure DANE TLSA DNS records._


## DNSSEC (Domain Name System Security Extensions):
The domain name system (DNS) is the phone book of the Internet: it tells computers where to send and retrieve information. Unfortunately, it also accepts any address given to it, no questions asked.

DNSSEC adds a security layer to this phonebook. It uses digital signatures to make sure the information in the phonebook is trustworthy and hasn't been tampered with. It's like putting a lock on the phonebook to prevent fake entries.

## DANE (DNS-based Authentication of Named Entities):
DANE used the DNS infrastructure to store details about the security of a service, such as the public key of a certificate. These details act as a special seal of approval, ensuring that when your computer talks to that service, it's the real thing and it's safe. DANE uses the TLSA _(Transport Layer Security Authentication)_ record type, which allows users to verify the certificate received from a service _(such as a web host on port 443 or a mail server on port 25)_ by querying the DNS for its information. DANE relies on DNSSEC and only works when DNSSEC is enabled.

### Implementation of DANE
You can implement DANE with a ```TLSA``` record in your domain registrar and DNS hosting provider; example:
- ```_port._protocol.yourdomain.com``` IN ```TLSA``` ```usage-selector-matching certificate-fingerprint```

  - ```_443._tcp.example.com``` IN ```TLSA``` ```3 1 1 a2a49838ad...```
      - 3: ```usage```
      - 1: ```selector```
      - 1: ```Matching-Type```

The certificate ```usage``` type can be one of the following:
- 0: PKIX-TA (Trust Anchor)
- 1: PKIX-EE (End Entity)
- 2: DANE-TA (Trust Anchor Assertion: Intermediate / Root certificate)
- 3: DANE-EE (End Entity Assertion: Host certificate / Server certificate such as yourdomain.com)

The ```selector``` specifies how the certificate is presented:
- 0: Full certificate (not recommended)
- 1: SubjectPublicKeyInfo (recommended)

The ```Matching-Type``` specifies how the certificate association is verified:
- 0: No hash used (not recommended)
- 1: SHA-256 hash (recommended)
- 2: SHA-512 hash (not recommended / less supported)

## How DNSSEC and DANE work together on 443
- Imagine you want to visit a secure website, and your browser wants to ensure it's communicating with the right place.

- DNSSEC acts like a lock for the phonebook (DNS), ensuring its trustworthiness by preventing malicious actors from providing fake addresses.

- DANE steps in and says, "Hey, not only the phonebook is secure, but here are some extra details like PublicKeyInfo of both the root and host certificates on the website to guarantee its authenticity". These details are sourced from the values in the ```TLSA``` records such as ```_443._tcp.yourdomain.com``` for both certificates.

    - Host: ```_443._tcp.yourdomain.com``` in ```TSLA```
        - Value: ```2 1 1 ROOT-certificate-fingerprint-PublicKey```
    - Host: ```_443._tcp.yourdomain.com``` in ```TSLA```
        - Value: ```3 1 1 End-ENTITY-certificate-fingerprint-PublicKey```

Example for this site:

![IMAGE](/images/dnssec-dane-explained/dnssec-dane-explained-1.png)

- Your browser then checks these additional details from DANE and verifies the integrity of the phonebook with DNSSEC to guarantee that everything is secure and legitimate.

![IMAGE](/images/dnssec-dane-explained/dnssec-dane-explained-2.png)

[VERIFY A DANE TLSA RECORD](https://check.sidnlabs.nl/dane/)

In simpler terms, DNSSEC secures the phonebook, and DANE adds a special note about a website's security. Together, they ensure that when you visit a website, it's the genuine one, and your connection is secure.

## How DNSSEC and DANE work together on 25 (SMTP DANE)
SMTP DANE is a security protocol that uses DNS to verify the authenticity of the certificates used for securing email communication with TLS and protecting against TLS downgrade attacks. DNSSEC is a set of extensions to DNS that provides cryptographic verification of DNS records, preventing DNS spoofing and adversary-in-the-middle attacks to DNS.

Where SPF, DKIM, and DMARC focus more on the email messages and the sending hosts they come from, DANE focuses more on establishing the TLS connection between mail servers.

### The flow of SMTP DANE
- **Sending Email Server:** fabrikam.com (outbound, requesting DANE record for the ```MX``` record of the domain)
- **Receiving Email Server:** contoso.com (inbound, requires DNSSEC and DANE ```TLSA```records)

1. The sending server at fabrikam.com queries the MX record of the receiving server, contoso.com, from the DNS server, resulting in "mail.contoso.com" (a DNSSEC/DANE-enabled domain).

2. The sending server at fabrikam.com requests the DANE record for "mail.contoso.com" from the DNS server.

3. The DNS server of contoso.com sends DANE records for the Root and End-entity certificates (mail.contoso.com) to the sending server at fabrikam.com:
    - Host: ```_25._tcp.mail.contoso.com``` in ```TSLA```
        - Value: ```2 1 1 ROOT-certificate-fingerprint```
    - Host: ```_25._tcp.mail.contoso.com``` in ```TSLA```
        - Value: ```3 1 1 End-ENTITY-certificate-fingerprint```

    ![IMAGE](tlsa-record.png)

4. The sending server at fabrikam.com establishes a TLS connection with the receiving server, contoso.com.

5. The receiving server at contoso.com sends certificates with fingerprints to the sending server at fabrikam.com.

6. The sending server at fabrikam.com verifies the fingerprints received from the DNS server:
    - If verification is successful, establish the connection.
    - If verification fails, disconnect.

## To Summarize
The use of DNSSEC and DANE is critical to strengthening online security. DNSSEC ensures data integrity and authentication in the DNS, while DANE associates digital certificates with domain names to prevent unauthorized access. Together, they form a powerful defense against evolving cyber threats, promoting trust and improving overall Internet security.