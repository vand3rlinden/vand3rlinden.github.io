---
title: "DNSSEC and DANE explained"
date: 2024-01-13T21:51:39+01:00
draft: false
categories: ["DNS"]
cover: 
  image: /images/dnssec-dane-explained/dnssec-dane-explained-front.png
---

> _In this post, you find out how DNSSEC and DANE cooperate, and learn how to set up DANE TLSA DNS records._

## DNSSEC (Domain Name System Security Extensions)
The domain name system (DNS) is the phone book of the Internet: it tells computers where to send and retrieve information. Unfortunately, it also accepts any address given to it, no questions asked.

DNSSEC adds a security layer to this phonebook. It uses digital signatures to make sure the information in the phonebook can be trusted and hasn't been tampered with. It's like putting a lock on the phonebook to prevent DNS spoofing.

> DNSSEC can be enabled with your DNS provider if they support it.

## DANE (DNS-based Authentication of Named Entities)
DANE used the DNS infrastructure to store details about the security of a service, such as the public key (fingerprint) of a certificate. These details act as a special seal of approval, ensuring that when your computer talks to this service, it's real and it's safe. DANE uses the TLSA _(Transport Layer Security Authentication)_ record type, which allows users to verify the security details from a service _(such as a web or mail server)_ by querying the DNS for its information.  

> DANE relies on DNSSEC and only works when DNSSEC is enabled.

## Implementation of DANE
You can implement DANE with a `TLSA` record at your DNS hosting provider:
- `_port._protocol.yourdomain.com` IN `TLSA` `usage-selector-matching certificate-fingerprint`

For example, on port 443 (your web server):
  - `_443._tcp.yourdomain.com` IN `TLSA` `2 1 1 0B9FA5A59EED...` (Root certificate)
  - `_443._tcp.yourdomain.com` IN `TLSA` `3 1 1 0D073B9B73B4...` (Host certificate)
      - 3: `usage`
      - 1: `selector`
      - 1: `Matching-Type`

The certificate `usage` type can be one of the following:
- 0: PKIX-TA (Trust Anchor)
- 1: PKIX-EE (End Entity)
- 2: DANE-TA (Trust Anchor Assertion: Intermediate / Root certificate)
- 3: DANE-EE (End Entity Assertion: Host certificate / Server certificate such as yourdomain.com)

The `selector` specifies how the certificate is presented:
- 0: Full certificate (not recommended)
- 1: SubjectPublicKeyInfo (recommended)

The `Matching-Type` specifies how the certificate association is verified:
- 0: No hash used (not recommended)
- 1: SHA-256 hash (recommended)
- 2: SHA-512 hash (not recommended / less supported)

## DNSSEC and DANE on a webserver
By combining DNSSEC and DANE, the integrity and authenticity of both the DNS responses and the TLS certificates are ensured, providing a robust mechanism to prevent man-in-the-middle attacks and other security threats.

## The flow of DNSSEC and DANE on a webserver
![IMAGE](/images/dnssec-dane-explained/webserverdane-visual.png)

Example for this site, `TLSA` records:
![IMAGE](/images/dnssec-dane-explained/dnssec-dane-explained-1.png)

[Verify DANE TLSA records:](https://check.sidnlabs.nl/dane/)
![IMAGE](/images/dnssec-dane-explained/dnssec-dane-explained-2.png)


## DNSSEC and DANE on a mailserver
SMTP DANE is a security protocol that uses DNS to verify the authenticity of the certificates used for securing email communication with TLS and protecting against TLS downgrade attacks. 

Where SPF, DKIM, and DMARC focus more on the email messages and the sending hosts they come from, DANE focuses more on establishing the TLS connection between mail servers.

## The flow of SMTP DANE on a mailserver
- **Outbound SMTP DANE with DNSSEC `sending mail server`**: Requests DANE `TLSA` records of the receiving domain's MX record.
- **Inbound SMTP DANE with DNSSEC `receiving mail server`**: Requires DNSSEC and DANE `TLSA` records that can be requested by the sending mail server.

![IMAGE](/images/dnssec-dane-explained/smtpdane-visual.png)

## To Summarize
The use of DNSSEC and DANE is critical to strengthening online security. DNSSEC ensures data integrity and authentication in the DNS, while DANE associates digital certificates with domain names to prevent unauthorized access. Together, they form a powerful defense against evolving cyber threats, promoting trust and improving overall Internet security.