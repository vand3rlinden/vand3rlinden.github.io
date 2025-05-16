---
title: "SPF, DKIM and DMARC explanation"
date: 2023-12-09T11:46:16+01:00
draft: false
categories: ["email", "DNS"]
cover: 
  image: /images/spf-dkim-dmarc-explanation/spf-dkim-dmarc-explanation-front.png
---

> _In this post you will learn to understand how the DNS protocols SPF, DKIM and DMARC work together to protect your domain from phishers and spammers._

## Why deploy SPF, DKIM, and DMARC?
SPF, DKIM, and DMARC are critical email authentication protocols that help prevent email spoofing, phishing attacks, and domain impersonation for ***outbound email***. Enabling these protocols together strengthens your domain's email authentication mechanisms, improves security, and builds trust with your recipients by ensuring that email sent from your domain is legitimate and trustworthy.

### SPF, DKIM and DMARC - Comparison table:
| Protocol | Purpose                                                                |
|----------|--------                                                                |
| SPF      | Verifies sender authorization by checking the sender's IP address      |
| DKIM     | Confirms message authenticity through cryptographic signature matching |
| DMARC    | Enforces a policy by requiring alignment between SPF and/or DKIM       |

### P1 vs. P2 - Sender explanation table:
In this blog, you will read a lot about the P1 and P2 sender. Please refer to the table below to become familiar with these terms.

| Postal Letter                        | Precise Term                    | Protected by  |
| -----------                          | -----------                     | -----------   |
| Sender on envelope (Envelope sender) | `RFC5321.MailFrom` (P1 Sender)  |  SPF          |
| Author on letter (Header sender)     | `RFC5322.From` (P2 Sender)      |  DKIM + DMARC |

## SPF
### What is SPF
Sender Policy Framework (SPF) is a protocol that aims to reduce spam. SPF can reduce email spoofing and spam by determining if the sender is authorized to send on behalf of the listed sender.

### How SPF works
Imagine that you have an SPF record that looks like: 

`v=spf1 include:_spf.domain.com ip4:11.222.33.444 -all`

If an unauthorized server sends on behalf of your domain, the email will get a `spf=fail` in the message header because the IP is not listed in your SPF record.

SPF will pass if the sender’s IP is added to the SPF record for the P1 Sender domain.

![IMAGE](/images/spf-dkim-dmarc-explanation/spf-visual.png)

### Implementation of SPF
When you add your domain to Microsoft 365, Microsoft will ask you to provide an SPF record, such as `v=spf1 include:spf.protection.outlook.com -all`. If you have more allowed senders, you must include them in the SPF record. For example `v=spf1 include:spf.protection.outlook.com include:_spf.domain.com ip4:11.222.33.444 -all`

The SPF record will vary for each domain; therefore, it is important to understand the following when implementing an SPF record:

- Can only have 255 characters, but it can be split to multiple strings in a single record, most DNS providers handle this automatically.
    - Example: `"v=spf1 first string" "second string -all"`
- Can take up to a maximum of 10 DNS Lookups, such as `include:spf.protection.outlook.com` (which currently contains 1 DNS lookup, without child lookups).

> **CAUTION**: If your DNS lookups are going to be around 10/10, you should take [steps](https://vand3rlinden.com/post/handle-your-spf-record/) to stay well under 10 DNS lookups. Because if a vendor decides to add another DNS lookup within its SPF include (child lookup). Your SPF record will become inaccurate because it has reached the DNS lookup limit, which may result in email delivery problems.

### Avoiding the 10 DNS Lookups cap, to:
- Flatten your SPF record ***(not recommended)***
    - Example: `v=spf1 include:spf.domain.com -all` can be `v=spf1 ip4:11.222.33.444 ip4:11.222.33.444 -all`

> The [problem with flattening](https://vand3rlinden.com/post/handle-your-spf-record/#the-dangers-of-spf-flattening) is that email service providers can change or add IP addresses without telling you. Then your SPF record will be inaccurate, leading to email delivery problems.

- Not using entries like `a` and `mx`, these mechanisms are often useless and probably should not be included in your SPF record (and other duplicate SPF mechanisms).

- Where email is incapable of passing DMARC with SPF, configure DKIM for the P2 Sender domain on the sending server or mail provider.
  - ***NOTE:*** In this scenario, you must to set your SPF record to [softfail](https://vand3rlinden.com/post/spf-dkim-dmarc-explanation/#softfail-or-hardfail) `~all`, this to ensure that the DMARC and DKIM evaluation will always be performed in the absence of a valid SPF validation.


- Move to vendor traffic with the use of subdomains for SPF authentication, subdomain segmentation creates a new domain dedicated to a particular mail stream with its own 10 DNS lookups. Organizations that segment their email streams find there is no need for SPF flattening and will end up with better controls, less attack surface, and limit any spillover effects of a potential cyber incident. 
  - Subdomain segmentation can be implemented in three ways:
    - Using a direct subdomain addresses.
    - Check if your email streams and providors have the ability to pass SPF on a subdomain on behalf of your primary domain; passing SPF on a subdomain is still DMARC compliant (assuming your DMARC policy uses the default `aspf=r` tag for SPF relaxed mode).
    - Use an [SPF macro](https://vand3rlinden.com/post/handle-your-spf-record/#use-an-spf-macro-to-restrict-a-third-party-service-to-send-from-a-specific-address) to limit a third-party service to sending from a specific address, since services like Salesforce are most often limited to sending from a single email address, such as `invoices@yourdomain.com`.

### Softfail or Hardfail
SPF can get a softfail or a hardfail, you determine that at the end of the record.

- With `~all`: The SPF record identifies the host as NOT authorized to send emails on behalf of the domain; however, it instructs email receivers to accept the email but flag it as potentially suspicious.

- With `-all`: The SPF record specifies that the host is NOT authorized to send emails and instructs email receivers to reject them.

Most mailbox providers will treat soft and hard- fails directives similarly, but it is [recommended](https://dmarcian.com/spf-best-practices/) to mirror the DMARC policy as the technology is deployed: use softfail (`~all`) if the DMARC policies are "none" and "quarantine", and use hardfail (`-all`) if you have moved to a "reject" policy. 

> **NOTE**: If an email cannot pass SPF (e.g. due to relaying), it can be hard rejected at the SMTP level with an SPF hardfail (`-all`), which may prevent DMARC and DKIM evaluation. If you are unsure whether your senders are passing SPF, then [consider using an SPF softfail (`~all`)](https://www.mailhardener.com/blog/why-mailhardener-recommends-spf-softfail-over-fail) along with DMARC set to reject (`p=reject`). This ensures that the DMARC and DKIM evaluation is always performed in the absence of a valid SPF validation.

### Limitations of SPF
Although SPF performs reasonably well in theory, it has several limitations that make it insufficient on its own to fully protect a sending domain.

SPF only validates the P1 sender, not the P2 sender. To address this gap, DKIM was introduced. DKIM helps protect the P2 sender by attaching a cryptographic signature to the message, allowing the receiving mail server to verify its authenticity by matching the private key used to sign the message with the public key published in DNS. However, not all sending mail servers had support to add DKIM signatures. This limitation led to the development of DMARC. DMARC allows domain owners to publish policies specifying how to handle messages that fail SPF and/or DKIM checks, particularly when the P1 and P2 identities are not aligned. Acting as the final layer of defense, DMARC ensures protection for the P2 sender by enforcing alignment and providing clear instructions for handling authentication failures.

## DKIM
### What is DKIM
DomainKeys Identified Mail (DKIM) is an authentication-based technique that allows the receiving mail server to verify the authenticity of an email by comparing the received private key with the public key if they match.

### How DKIM works
The receiving server makes a DNS request using the sender’s domain name (P2 Sender). In response to the DNS request, the receiving server obtains the public key from a DNS record in the DNS zone of the sending domain, such as `selector1._domainkey.yourdomain.com`, and compares it to the private key in the message from the sending server.

DKIM will pass if the sending server’s private key can be confirmed by the receiving server, using the public key in the sending domain’s DNS zone. This public key is published in a `TXT` or within a `CNAME` record. For example, if in `TXT`:

- Hostname: `key1._domainkey.yourdomain.com` (in `TXT`)
- Value: `v=DKIM1; k=rsa; p=MIIBIjANBgkqhkiG9….;` (public key)

![IMAGE](/images/spf-dkim-dmarc-explanation/dkim-visual.png)

### Implementation of DKIM
You can configure DKIM with a `TXT` record in your DNS zone for your sending mail servers (such as postfix or sendmail). A DKIM generator can be used to generate a private key for your server and a public key for your DNS.

> **NOTE**: A DKIM record is required for each sending server or mail provider.

How you set up DKIM depends on your sending server or mail provider. For example, configuring DKIM for [Salesforce](https://help.salesforce.com/s/articleView?id=sales.emailadmin_create_secure_dkim.htm) differs from [Exchange Online](https://learn.microsoft.com/en-us/defender-office-365/email-authentication-dkim-configure) in Microsoft Defender for Office 365. If you’re managing your own mail server and using MTAs like Sendmail or Postfix, you can implement DKIM using tools such as [OpenDKIM](http://www.opendkim.org/).

### DKIM recommendations
- You must implement DKIM key rotation for each sending service to prevent adversaries from intercepting and decrypting your DKIM keys. DKIM Key rotation helps to minimize the risk of having a private key compromised.
  - Recurring: Every six months for a 2048-bit DKIM key.
- The DKIM key length must be at least 2048-bits. 

## DMARC
### What is DMARC
DMARC (Domain-based Message Authentication, Reporting and Conformance) acts as a shield on top of SPF and DKIM. With DMARC, domain owners specify how to handle emails sent on behalf of their domain if those messages fail SPF and/or DKIM checks. This allows them to define whether such emails should be monitored, quarantined, or rejected entirely.

### How DMARC works
Once your domain has a DMARC record, any receiving email server can verify the incoming email based on the instructions in the DMARC policy. If the email passes the DMARC authentication, it will be delivered and can be trusted. If the email fails the check, depending on the instructions in the DMARC record, the email can be delivered, quarantined (flagged as potentially suspicious), or rejected.

DMARC will pass if the P1 Sender and P2 Sender are equal, and/or SPF and DKIM are passed. If the P1 sender is not equal to the P2 sender, then DKIM must pass for the P2 sender domain in order to get a DMARC pass.

![IMAGE](/images/spf-dkim-dmarc-explanation/dmarc-visual.png)

### Implementation of DMARC
Before using DMARC, you need to know how often it will fail. You should consider monitoring DMARC failures before setting the DMARC policy directly to reject.

There are several monitoring tools that convert the DMARC reports into a clear dashboard. One such tool is [Valimail](https://www.valimail.com/blog/office-365-free-dmarc-monitoring/) (free for Microsoft 365 users with an Exchange Online plan).

Overview of the Valimail Dashboard:

![IMAGE](/images/spf-dkim-dmarc-explanation/valimail-monitor.png)

> To sign up for a free Valimail environment, click [here](https://www.valimail.com/sign-up-ms-blog/). Once the setup is complete, you can sign in at [app.valimail.com](https://app.valimail.com/). After logging in, you can proceed to integrate [Microsoft Entra ID SSO with Valimail](https://support.valimail.com/en/articles/8466393-tutorial-how-to-integrate-microsoft-entra-id-sso-with-valimail).

For heavy mail domains, I recommended monitoring the domain for at least three months with the DMARC policy set to `none`:

- Hostname: `_dmarc`
- Value: `v=DMARC1; p=none; sp=none; rua=mailto:dmarc_agg@vali.email;`

During the monitoring phase, you can [adjust your SPF record](https://vand3rlinden.com/post/handle-your-spf-record/) and [set up DKIM for each sender](https://vand3rlinden.com/post/spf-dkim-dmarc-explanation/#implementation-of-dkim), and then update your DMARC record to `reject` once the monitoring phase is complete:

- Value: `v=DMARC1; p=reject; sp=reject; rua=mailto:dmarc_agg@vali.email;`

The `sp=reject` tag means that subdomains will be included; if you don’t want your subdomains to be included in your domain’s root DMARC policy, you can set this to `sp=none` and list a separate DMARC policy for each subdomain (not recommended).

> **NOTE**: If you do not list the `sp=` tag, your subdomains will get the policy from the `p=` tag.

### DMARC policy explanation
| Policy      | Value           | Meaning       |
| ----------- | -----------     | -----------   |
| None        | `p=none;`       |  This policy is essentially a monitoring or reporting mode. It instructs email receivers not to take any action based on the DMARC authentication results. |
| Quarantine  | `p=quarantine;` |  This policy instructs email receivers to mark emails that fail DMARC authentication as potentially suspicious, typically delivering them to the recipient’s SPAM folder. As a result, the DMARC quarantine policy does not fully utilize the protective capabilities of DMARC. |
| Reject      | `p=reject;`     |  This policy instructs email receivers to reject (not deliver) emails that fail DMARC authentication. |

The table on this [Microsoft Learn page](https://learn.microsoft.com/en-us/archive/blogs/fasttracktips/spf-dkim-dmarc-and-exchange-online#covering-the-basics-of-dmarc) summarizes the options you have when configuring your DMARC policy.

### RUA vs. RUF DMARC Reports
- `RUA` (Reporting URI for Aggregate reports):
  -  Is designed to send domain owners data about email authentication results for SPF, DKIM, and DMARC. These reports are essential for monitoring how a domain is being used and contain only authentication outcomes and message counts, without including any sensitive email content.
     - Combined data on a group of emails.
     - Not real-time, they are sent everyday by default.
     - Sent in `XML` format.
     - No PII (Personal Identifiable Information).
     - Supported in all DMARC-compliant mailbox providers.

- `RUF` (Reporting URI for Forensic reports): 
  - Is designed to send domain owners detailed failure reports when emails don’t pass DMARC checks. These reports may include portions of the original message headers and metadata, sometimes with limited message content, depending on the reporting provider. The goal is to help identify legitimate sources that need to be properly authenticated. However, due to privacy concerns and the potential exposure of Personal Identifiable Information (PII), most providers do not send `RUF` reports.
    - Details of an individual email.
    - Sent almost immediately after the failures.
    - Plain text format.
    - Contains PII (Personal Identifiable Information).
    - Supported in only a handful of mailbox providers.

> **NOTE**: You don't need `RUF` Reporting to get a DMARC compliant domain, `RUA` is sufficient.

## Protect all non-sending domains
To protect all non-sending domains, you should consider:
- A ***deny all SPF*** record:
  - Name: `@` 
  - Content:`v=spf1 -all` 
  - Type: `TXT`
- A ***reject DMARC*** record:
  - Name: `_dmarc` 
  - Content: `v=DMARC1; p=reject;`
  - Type: `TXT`
- An ***unassociated public key as a DKIM*** wildcard record:
  - Name: `*._domainkey`
  - Content: `v=DKIM1; p=`
  - Type: `TXT`

This protects all of your domains from phishers and spammers, as bad actors will actively look for unused domains to exploit.

## To Summarize
**SPF**: performs verification that the IP address of the sending server matches the entry in the SPF record from the sending domain.
- **Purpose**: Sender authorization check
- **Protect**: `RFC5321.MailFrom` (P1 Sender)

**DKIM**: verifies if the public key (DNS record) of a sending domain, matched the private key that came from the sending server. This is a check that the sending domain actually sent the e-mail. DKIM must be configured for **each** sending server, such as Exchange Online or any other server/SaaS service.
- **Purpose**: Message authenticity verification
- **Protect**: `RFC5322.From` (P2 Sender)

**DMARC**: ensures that emails that fail the SPF and/or DKIM tests do not get through. If the email is unable to pass DMARC with SPF, DKIM can help pass DMARC for the P2 sender domain. Therefore, it is really important to have DKIM enabled for all your sending servers before setting DMARC to `reject`.
- **Purpose**: Final sheld on top of SPF and DKIM
- **Protect**: `RFC5322.From` (P2 Sender)

## Reference
- [Dmarcian SPF best practices - Unavailable from the Netherlands](https://dmarcian.com/spf-best-practices/)
- [Dmarcian Concluding the Experiment: SPF Flattening - Unavailable from the Netherlands](https://dmarcian.com/spf-flattening/)
- [Dmarcian DMARC best practices - Unavailable from the Netherlands](https://dmarcian.com/advancing-dmarc-policy/)
- [Dmarcian The Difference in DMARC Reports: RUA and RUF - Unavailable from the Netherlands](https://dmarcian.com/rua-vs-ruf/)
- [Why Mailhardener recommends SPF softfail over fail](https://www.mailhardener.com/blog/why-mailhardener-recommends-spf-softfail-over-fail)
- [How to protect domains that do not send email](https://www.cloudflare.com/learning/dns/dns-records/protect-domains-without-email/)
- [SPF is defined in RFC 7208](https://www.rfc-editor.org/info/rfc7208)
- [DKIM is defined in RFC 6376](https://www.rfc-editor.org/info/rfc6376)
- [DMARC is defined in RFC 7489](https://www.rfc-editor.org/info/rfc7489)
- [SPF Policy Tester](https://vamsoft.com/support/tools/spf-policy-tester)