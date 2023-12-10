+++
title = 'SPF, DKIM and DMARC explanation'
date = 2023-12-09T11:46:16+01:00
draft = false
+++

> _In this post you will learn to understand SPF, DKIM, DMARC and how these DNS protocols work together to protect your domain from phishers and spammers._

## SPF
### What is SPF
Sender Policy Framework (SPF) is a protocol that aims to reduce spam. SPF can reduce email spoofing and spam by determining if the sender is authorized to send on behalf of the listed sender.

### How SPF works
Imagine that you have an SPF record that looks like:

“v=spf1 include:_spf.domain.com ip4:11.222.33.444 -all”

If an unauthorized server sends on behalf of your domain, the email will get an spf=fail in the message header because that IP is not listed in your SPF record.

SPF will pass if the sender’s IP is added to the SPF record for the P1 Sender domain.

### P1 vs P2- sender explanation:
| Postal Letter      | Precise Term                  | Protected by  |
| -----------        | -----------                   | -----------   |
| Sender on envelope | RFC5321.MailFrom (P1 Sender)  |  SPF          |
| Author on letter   | RFC5322.From (P2 Sender)      |  DKIM + DMARC |

### Implementation of SPF
When you add your domain to Microsoft 365, Microsoft will ask you to provide an SPF record, such as “v=spf1 include:spf.protection.outlook.com -all”. If you have more allowed senders, you must include them in the SPF record. For example: “v=spf1 include:spf.protection.outlook.com include:_spf.domain.com ip4:11.222.33.444 -all”

The SPF record will differ for everyone; therefore, it is important to understand the following when implement an SPF- record:

- Can only have 255 characters, but it can be split to multiple strings in a single record, most DNS providers handle this automatically.
    - Example: “v=spf1 first string” “second string -all”
- Can take up to a maximum of 10 DNS Look Ups like: “include:spf.protection.outlook.com” (which already include two DNS lookups)

### Avoiding the 10 DNS Look Ups cap, to:
- Flatten your SPF record (not recommended)
    - Example: “v=spf1 include:spf.domain.com -all” can be “v=spf1 ip4:11.222.33.444 ip4:11.222.33.444 -all”.

Problem with flattening is that email service providers may change or add IP addresses without telling you. Then your SPF record is inaccurate, which leads into email delivery problems

- Not using entries like “a” and “mx”, these mechanisms are often useless and probably should not be included in your SPF record (and other duplicate SPF mechanisms).

- Move to vendor traffic with using subdomains for SPF authentication. Subdomain segmentation creates a new domain dedicated to a particular mail stream with its own 10 DNS Look Ups. Learn more about this in this post.

### Softfail or Hardfail
SPF can get a softfail or fail, you determine this at the end of the record.

- With ~all: If an email is sent from a host or IP that is not in the SPF record, the message will still pass, but it can be flagged as spam (Softfail).

- With -all: If an email is sent from a host or IP that is not in the SPF record, it can be rejected by the receiving server (HardFail).

### Limitations of SPF
Although SPF works relatively well in theory, there are several flaws in the protocol that mean that SPF alone is not enough to protect a sending domain.

The SPF protocol only protects the P1 sender and not the content of the email, which is the P2 sender of the email. DKIM was set up to protect the P2 sender by adding a signature (SHA-256 hash function) to the email.

Then there is the problem that not every receiving mail server has an active policy for SPF and DKIM, which is why DMARC was created. With DMARC, the sender specifies on behalf of their domain what should be done with email that does not meet the requirements. DMARC can be used to protect the P2 sender as a last line of defense.

## DKIM
### What is DKIM
DomainKeys Identified Mail (DKIM) is an authentication-based technique, by using DKIM, the receiving mail server can check the authenticity of a e-mail with the DKIM-Signature header. This signature is created using the SHA-256 hash function.

### How DKIM works
The receiving server makes a DNS request using the sender’s domain name (P2 Sender). In response to the DNS request, the receiving server obtains the public key from a DNS record in the DNS zone of the sending domain, such as selector1._domainkey.yourdomain.com, and compares it to the private key in the message from the sending server.

DKIM will pass if the sending server’s private key can be confirmed by the receiving server, using the public key in the sending domain’s DNS zone. This public key is published in a TXT/CNAME record. For example, when in TXT:

- Hostname: key1._domainkey.yourdomain.com (in TXT)
- Value: v=DKIM1;t=s;p=MIIBIjANBgkqhkiG9w0BA... (public key)

### Implementation of DKIM
You can configure DKIM with a TXT record in your DNS zone for your sending mail servers (such as postfix or sendmail). A DKIM generator can be used to generate a private key for your server and a public key for your DNS.

> _Note: that a DKIM record is required for each sending server or mail provider._

How you set up DKIM can vary depending on your mail provider; setting up DKIM for [Google Workplace](https://support.google.com/a/answer/180504?hl=en) differs from [Exchange Online](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/email-authentication-dkim-configure?view=o365-worldwide#steps-to-create-enable-and-disable-dkim-from-microsoft-defender-portal) in Microsoft Defender for Office 365.
