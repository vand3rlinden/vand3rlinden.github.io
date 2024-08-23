---
title: "Understanding the Role and Benefits of ARC Sealing"
date: 2024-05-24T23:37:34+02:00
draft: false
categories: ["email", "DNS"]
cover: 
  image: /images/arc-explained/arc-explained-front.png
---

> _This blog post explains the role and benefits of ARC sealing._

## Introduction
ARC (Authenticated Received Chain) is an email authentication protocol that preserves the authentication results of an email as it travels through multiple intermediaries, such as forwarding services.

By using ARC, organizations can better manage the complexities of email authentication, especially when email is forwarded, but ARC is a collaboration of multiple servers that trust each other. 

ARC ensures that legitimate emails are less likely to be marked as spam or rejected, while fraudulent emails are more easily identified and filtered out. In this blog post, we will explore the basics of ARC, how it works, and the benefits it provides.

## How ARC works
ARC (Authenticated Received Chain) sealing is a way to help ensure the authenticity of email messages as they pass through various email servers. 

When you send an email message, it goes through your sending server and may be routed through an intermediate server (mailing lists or account forwarding services) before it reaches the receiving server. However, some legitimate mail services may modify messages before they're delivered to the receiving server. Modifying incoming messages in transit can, and likely will, cause the following email authentication failures:
- **SPF** fails because of the new message source (IP address)
    - SPF checks email messages against an authorized list of IP addresses. When email is forwarded, it passes through an intermediate server whose IP may not be on the sender's SPF list. This results in unwanted SPF failures, even for legitimate email. 
- **DKIM** fails because of content modification
    - DKIM adds digital signatures to your emails that can be encrypted using a public key to verify the source and authenticity of the message. To do this, DKIM uses a hash value generated from the email header and body. However, during email forwarding scenarios, additional elements such as custom footers or extended subject lines may be added to the email, which can invalidate DKIM. 
- **DMARC** fails because of the SPF and DKIM failures
	- DMARC assumes that emails are completely unchanged throughout the delivery process.

![IMAGE](/images/arc-explained/arc-visual.png)

> Want to learn more about SPF, DKIM, and DMARC? You can read my [previous blog post](https://vand3rlinden.com/post/spf-dkim-dmarc-explanation/) about these outbound authentication protocols.

## ARC Headers
ARC helps preserve email authentication results and verifies the identity of intermediate server(s) that forward a message on to its final destination. There are three key components to ARC:

- The `ARC-Authentication-Results` header: A header containing email authentication results like SPF, DKIM, and DMARC. 

- The `ARC-Message-Signature` header: This header takes a snapshot of the message header information, including To, From, Subject, and Body.  

- The `ARC-Seal` header: This header includes the `ARC-Message-Signature` and `ARC-Authentication-Results` header information and contains a tag called chain validation `cv=`, which contains the result of the evaluation of the existing ARC chain. The value can be `none`, `fail` or `pass`.
	- `none`: No ARC chain to validate
	- `fail`: ARC chain validation failed
	- `pass`: ARC chain validation succeeded

## What ARC can’t do
ARC has limitations and is not a replacement for DMARC. For example, ARC doesn't provide any information about the reputation or trustworthiness of the sender or the intermediate server, because an intermediate server can still add bad content or remove some (or even all) ARC headers.

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