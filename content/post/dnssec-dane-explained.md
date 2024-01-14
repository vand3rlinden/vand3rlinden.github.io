---
title: "DNSSEC and DANE explained"
date: 2024-01-13T21:51:39+01:00
draft: false
categories: ["DNS"]
cover: 
  image: /images/dnssec-dane-explained/dnssec-dane-explained-front.png
---

> _In this post, you will learn how DNSSEC and DANE work together and how to configure DANE TLSA DNS records._


## DNSSEC (Domain Name System Security Extensions)
The domain name system (DNS) is the phone book of the Internet: it tells computers where to send and retrieve information. Unfortunately, it also accepts any address given to it, no questions asked.

DNSSEC adds a security layer to this phonebook. It uses digital signatures to make sure the information in the phonebook can be trusted and hasn't been tampered with. It's like putting a lock on the phonebook to prevent DNS spoofing.

## DANE (DNS-based Authentication of Named Entities)
DANE used the DNS infrastructure to store details about the security of a service, such as the public key of a certificate. These details act as a special seal of approval, ensuring that when your computer talks to this service, it's real and it's safe. DANE uses the TLSA _(Transport Layer Security Authentication)_ record type, which allows users to verify the security details from a service _(such as a web or mail server)_ by querying the DNS for its information.  

> DANE relies on DNSSEC and only works when DNSSEC is enabled.

### Implementation of DANE
You can implement DANE with a ```TLSA``` record at your DNS hosting provider; example:
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

## How DNSSEC and DANE work together on a webserver (443)
- Imagine you want to visit a secure site, and your browser wants to make sure it's communicating with the right site.

- DNSSEC acts as a lock on the phonebook, ensuring it can be trusted by preventing malicious actors from providing spoofed addresses.

- DANE steps in and says, _"Hey, not only the phonebook is secure, but here are some extra details like the PublicKeyInfo of both the root and host certificates of the site to guarantee its authenticity"_. These details are sourced from the values in the ```TLSA``` records such as ```_443._tcp.yourdomain.com``` for both certificates.

    - Host: ```_443._tcp.yourdomain.com``` in ```TSLA```
        - Value: ```2 1 1 ROOT-certificate-fingerprint-PublicKey```
    - Host: ```_443._tcp.yourdomain.com``` in ```TSLA```
        - Value: ```3 1 1 End-ENTITY-certificate-fingerprint-PublicKey```

Example for this site:

![IMAGE](/images/dnssec-dane-explained/dnssec-dane-explained-1.png)

- Your browser then checks these additional details from DANE and verifies the integrity of the phonebook with DNSSEC to guarantee that everything is secure and legitimate.

![IMAGE](/images/dnssec-dane-explained/dnssec-dane-explained-2.png)
[VERIFY A DANE TLSA RECORD](https://check.sidnlabs.nl/dane/)

## How DNSSEC and DANE work together on a mailserver (25, SMTP DANE)
SMTP DANE is a security protocol that uses DNS to verify the authenticity of the certificates used for securing email communication with TLS and protecting against TLS downgrade attacks. 

Where SPF, DKIM, and DMARC focus more on the email messages and the sending hosts they come from, DANE focuses more on establishing the TLS connection between mail servers.

### The flow of SMTP DANE
- **Sending Email Server:** fabrikam.com (outbound, requesting DANE ```TLSA``` records of the ```MX``` record from the receiving email server)
- **Receiving Email Server:** contoso.com (inbound, requires DNSSEC and DANE ```TLSA```records)

1. The sending server at ***fabrikam.com*** queries the ```MX``` record from the DNS server of ***contoso.com***, resulting in ***mail.contoso.com*** (a DNSSEC/DANE-enabled domain).

2. The sending server at ***fabrikam.com*** requests the DANE ```TLSA``` records for ***mail.contoso.com*** from the DNS server.

3. The DNS server of ***contoso.com*** sends DANE ```TLSA``` records for the Root and End-entity certificates of ***mail.contoso.com*** to the sending server ***fabrikam.com***:
    - Host: ```_25._tcp.mail.contoso.com``` in ```TSLA```
        - Value: ```2 1 1 ROOT-certificate-fingerprint```
    - Host: ```_25._tcp.mail.contoso.com``` in ```TSLA```
        - Value: ```3 1 1 End-ENTITY-certificate-fingerprint```

4. The sending server at ***fabrikam.com*** establishes a TLS connection with the receiving server ***mail.contoso.com***.

5. The receiving server at ***mail.contoso.com*** sends certificates with fingerprints to the sending server ***fabrikam.com***.

6. The sending server at ***fabrikam.com*** verifies the fingerprints received from the DNS server of ***contoso.com***:
    - If verification is successful, establish the connection.
    - If verification fails, disconnect.

## To Summarize
The use of DNSSEC and DANE is critical to strengthening online security. DNSSEC ensures data integrity and authentication in the DNS, while DANE associates digital certificates with domain names to prevent unauthorized access. Together, they form a powerful defense against evolving cyber threats, promoting trust and improving overall Internet security.