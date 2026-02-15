---
title: "Understanding the role and benefits of ARC Sealing"
date: 2024-05-24T23:37:34+02:00
draft: false
categories: ["email", "DNS"]
cover: 
  image: /images/arc-explained/arc-explained-front.png
---

> _This blog post explains the role and benefits of ARC sealing._

ARC (Authenticated Received Chain) is an email authentication protocol that preserves the authentication results of an email as it travels through multiple intermediaries, such as forwarding services. This allows your recipients to accept the ARC Seal from your relaying or intermediate server. 

Using ARC helps organizations handle the complexities of email authentication, especially when emails are forwarded. ARC involves multiple servers working together based on mutual trust. In this blog post, we will explore the basics of ARC, how it works, and the benefits it provides.

## How ARC works
ARC (Authenticated Received Chain) sealing is a way to help ensure the authenticity of email messages as they pass through various email servers. 

When you send an email message, it goes through your sending server and may be routed through an intermediate server (mailing lists or account forwarding services) before it reaches the receiving server. However, some legitimate mail services may modify messages before they're delivered to the receiving server. Modifying incoming messages in transit can, and likely will, cause the following email authentication failures:
- **SPF** fails because of the new message source (IP address)
    - SPF checks email messages against an authorized list of IP addresses. When email is forwarded, it passes through an intermediate server whose IP may not be on the sender's SPF list. This results in unwanted SPF failures, even for legitimate email. 
- **DKIM** fails because of content modification
    - DKIM adds a signature to an email that is hashed and signed using a private key on the sending server and can be verified with a public key published in DNS. If the email is modified during forwarding, such as by changing the email body or certain headers, the DKIM signature may no longer match, causing the verification to fail.
- **DMARC** fails because of the SPF and DKIM failures
	- DMARC assumes that emails are completely unchanged throughout the delivery process.

![IMAGE](/images/arc-explained/arc-visual.png)

> Want to learn more about SPF, DKIM, and DMARC? You can read my [previous blog post](https://vand3rlinden.com/post/spf-dkim-dmarc-explanation/) about these outbound authentication protocols.

## ARC Headers
ARC helps preserve email authentication results and verifies the identity of intermediate server(s) that forward a message on to its final destination. 

There are three key components to ARC:
- The `ARC-Authentication-Results` header: A header containing email authentication results like SPF, DKIM, and DMARC. 
- The `ARC-Message-Signature` header: This header takes a snapshot of the message header information, including To, From, Subject, and Body.  
- The `ARC-Seal` header: This header includes the `ARC-Message-Signature` and `ARC-Authentication-Results` header information and contains a tag called chain validation `cv=`, which contains the result of the evaluation of the existing ARC chain. The value can be `none`, `fail` or `pass`:
	- `none`: No ARC chain to validate
	- `fail`: ARC chain validation failed
	- `pass`: ARC chain validation succeeded

Each ARC header includes an `i=` tag, which stands for ARC instance. This number indicates the position of the system in the forwarding chain and helps establish the order of trust. 

For example:
- `i=1` is added by the first intermediary (e.g., the first system that receives and forwards the original message).
- `i=2` is added by the next hop, which receives the message from the first forwarder and re-validates the earlier authentication results.
- `i=3` and higher are added by subsequent intermediaries or the final destination mail server.

Each ARC instance adds its own set of headers:
- `ARC-Authentication-Results` (its own assessment of SPF, DKIM, DMARC, and possibly ARC).
- `ARC-Message-Signature` (to cryptographically bind the message content and headers).
- `ARC-Seal` (to seal the entire ARC package and validate the prior chain).

This system creates a verifiable chain of custody that helps the final recipient determine whether the authentication results can still be trusted, even if the message was forwarded or altered en route.

## The ARC trust chain
When emails pass through multiple intermediaries, traditional authentication methods like SPF and DKIM can fail. That’s where ARC comes in, preserving the original authentication results across each hop. Below is an example of how ARC headers build up through multiple relays, including SendGrid, a security gateway, and finally Microsoft 365.

### Message Flow
* **Original Sender:** `shaggy@vand3rlinden.com` (sent from a DMARC-compliant mail server for `vand3rlinden.com`)
* **Hop 1:** SendGrid (SMTP relay service acting as the intermediate server for `vand3rlinden.com`)
* **Hop 2:** Security Gateway
* **Final Recipient:** Microsoft 365 user

### ARC Header Chain Examples
#### ✅ ARC Instance 1 (Outbound authentication passed, ARC Seal added by SendGrid)
```
ARC-Authentication-Results: i=1; d=sendgrid.net;
 spf=pass smtp.mailfrom=vand3rlinden.com;
 dkim=pass header.d=vand3rlinden.com;
 dmarc=pass header.from=vand3rlinden.com;

ARC-Message-Signature: i=1; a=rsa-sha256; d=sendgrid.net;
 s=arc; h=from:to:subject:date;
 bh=[body-hash]; b=[signature]

ARC-Seal: i=1; a=rsa-sha256; d=sendgrid.net; s=arc;
 t=[timestamp]; cv=none; b=[seal-signature]
```

#### ✅ ARC Instance 2 (ARC chain validation succeeded, ARC Seal added by a Security Gateway)
```
ARC-Authentication-Results: i=2; d=securitygateway.com;
 spf=pass; dkim=pass; dmarc=pass;

ARC-Message-Signature: i=2; a=rsa-sha256; d=securitygateway.com;
 s=arc; h=from:to:subject:date;
 bh=[body-hash]; b=[signature]

ARC-Seal: i=2; a=rsa-sha256; d=securitygateway.com; s=arc;
 t=[timestamp]; cv=pass; b=[seal-signature]
```

#### ✅ ARC Instance 3 (ARC chain validation succeeded, ARC Seal added by Microsoft 365)
```
ARC-Authentication-Results: i=3; d=microsoft.com;
 spf=pass; dkim=pass; dmarc=pass;

ARC-Message-Signature: i=3; a=rsa-sha256; d=microsoft.com;
 s=arc; h=from:to:subject:date;
 bh=[body-hash]; b=[signature]

ARC-Seal: i=3; a=rsa-sha256; d=microsoft.com; s=arc;
 t=[timestamp]; cv=pass; b=[seal-signature]
```

Each instance (`i=1`, `i=2`, `i=3`) seals the authentication results of the previous hop. By the time it reaches Microsoft 365 (`i=3`), the entire trust chain can be evaluated, even if the original SPF or DKIM fails due to forwarding. This is the core benefit of **ARC**, preserving authentication integrity across multiple hops.

## Mailbox providers that support ARC Sealers
ARC has already been adopted by major mailbox providers such as Google and Microsoft, and is likely to become a global standard.

## Set ARC seals on your intermediate server
The commercial MTAs Halon and MailerQ include ARC sealing. For open source solutions, [authentication_milter](https://github.com/fastmail/authentication_milter) or [OpenARC](https://github.com/trusteddomainproject/OpenARC) can be used to implement ARC with the Postfix, Oracle Communications Messaging Server, and Sendmail MTAs.

## Accept trusted ARC sealers on your receiving server
An intermediate server's ARC seal can be accepted by an administrator within their mailbox provider (receiving server). To do this, you need to add the trusted signing domain. This domain must match the domain that's shown in the `d` value in the `ARC-Seal` and `ARC-Message-Signature` headers in affected messages.

The steps to accept an ARC seal depend on your mailbox provider; for Microsoft 365, you can use [this](https://learn.microsoft.com/en-us/defender-office-365/email-authentication-arc-configure) documentation.

## To summarize
ARC ensures that email authentication results are preserved as messages pass through multiple hops, such as forwarding services and other intermediate services.
The success of ARC really depends on email receivers trusting each other by applying the protocol.

## Reference
- [ARC Specifications](https://arc-spec.org/)
- [ARC is defined in RFC 8617](https://www.rfc-editor.org/info/rfc8617)
- [Microsoft Defender for Office 365: Configure trusted ARC sealers](https://learn.microsoft.com/en-us/defender-office-365/email-authentication-arc-configure)