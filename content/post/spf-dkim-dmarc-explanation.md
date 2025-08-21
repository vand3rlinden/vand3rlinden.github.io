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

`v=spf1 ip4:11.222.33.444 include:_spf.domain.com ~all`

If an unauthorized server sends on behalf of your domain, the email will get a `spf=fail` in the message header because the IP is not listed in your SPF record.

SPF will pass if the sender’s IP is added to the SPF record for the P1 Sender domain.

### How SPF evaluation works
![IMAGE](/images/spf-dkim-dmarc-explanation/spf-visual.png)

### Implementation of SPF
When you add your domain to Microsoft 365, Microsoft will ask you to provide an SPF record, such as `v=spf1 include:spf.protection.outlook.com ~all`. If you have more allowed senders, you must include them in the SPF record. For example `v=spf1 ip4:11.222.33.444 include:spf.protection.outlook.com include:_spf.domain.com ~all`

The SPF record will vary for each domain; therefore, it is important to understand the following when implementing an SPF record:

- Can only have 255 characters, but it can be split to multiple strings in a single record, most DNS providers handle this automatically.
    - Example: `"v=spf1 first string" "second string ~all"`
- Can take up to a maximum of 10 DNS Lookups, such as `include:spf.protection.outlook.com` (this entry costs 1 DNS lookup, with no additional child lookups).

> **CAUTION**: You should take [steps](https://vand3rlinden.com/post/handle-your-spf-record/) to stay well under 10 DNS lookups. Because if a vendor decides to add another DNS lookup within its SPF include (child lookup). Your SPF record will become inaccurate because it has reached the DNS lookup limit, which may result in email delivery problems.

### Tips to avoid reaching the 10 DNS lookups limit
- Flatten your SPF record.
    - Example: `v=spf1 include:spf.domain.com ~all` can be `v=spf1 ip4:11.222.33.444 ip4:11.222.33.444 ~all`

> **Not recommended**: The [problem with flattening](https://vand3rlinden.com/post/handle-your-spf-record/#the-dangers-of-spf-flattening) is that email service providers can change or add IP addresses without telling you. Then your SPF record will be inaccurate, leading to email delivery problems.

- Where email is incapable of passing DMARC with SPF (e.g. due to relaying), configure DKIM for the P2 Sender domain on the sending server or email provider.
  - In this scenario, you must to set your SPF record to softfail `~all`, instead of harfail `-all`. Softfail ensures that DKIM evaluation is always performed in absence of valid SPF validation due to relaying, so the email can still be DMARC compliant due to DKIM alignment. More details will follow on why using softfail instead of hardfail is considered best practice.

> **Not recommended**: You should work toward having both SPF and DKIM properly aligned. However, in relay scenarios, this is not always possible, even for DKIM. In such cases, you can check if your email provider supports ARC Sealers, which help preserve authentication results when an email passes through multiple servers. This allows your recipients to accept the ARC Seal from your relaying or intermediate server. More information about ARC Sealers can be found in [this](https://vand3rlinden.com/post/arc-explained/) blog.

- Allow SPF to pass by using a different P1 sender, such as your email provider’s domain (if supported), and use DKIM on your own domain as the P2 sender. This method still DMARC compliant because DKIM is aligned with the P2 sender domain.

> **Not recommended**: It is not best practice to rely only on DKIM validation. If DKIM fails, for example due to slow DNS response, there is no fallback. The best practice is to have both SPF and DKIM correctly aligned.

- Not using entries like `a` and `mx`, these mechanisms are often useless and probably should not be included in your SPF record (and other duplicate SPF mechanisms).

- Move to vendor traffic with the use of subdomains for SPF authentication, subdomain segmentation creates a new domain dedicated to a particular email provider with its own 10 DNS lookups. Organizations that segment their email providers find there is no need for SPF flattening and will end up with better controls, less attack surface, and limit any spillover effects of a potential cyber incident. 
  - Subdomain segmentation can be implemented in three ways:
    - Using a direct subdomain addresses.
    - Check if your email providers have the ability to pass SPF on a subdomain on behalf of your primary domain; passing SPF on a subdomain is still DMARC compliant (assuming your DMARC policy uses the default `aspf=r` tag for SPF relaxed mode).
    - Use an [SPF macro](https://vand3rlinden.com/post/handle-your-spf-record/#use-an-spf-macro-to-restrict-a-third-party-service-to-send-from-a-specific-address) to limit a third-party service to sending from a specific address, since services like Salesforce are most often limited to sending from a single email address, such as `invoices@yourdomain.com`.

### Softfail or Hardfail
SPF can get a softfail or a hardfail, you determine that at the end of the record.

- With `~all`: Email originating from a host that matches the term will not pass SPF evaluation, but will still be tested for other mechanisms (like DKIM).

- With `-all`: Email originating from a host that matches the term will be rejected, no further mechanism (like DKIM) may be used.

Most mailbox providers treat SPF softfail (`~all`) and hardfail (`-all`) in a similar way. However, when an email fails SPF, such as in cases involving relaying or delegated senders, it may be hard rejected at SMTP level if an SPF hardfail (`-all`) is used. This can prevent DKIM evaluation on the receiving mail server, which may lead to a DMARC failure. 

The SPF hardfail mechanism (`-all`) is only recommended for domains that do not send email. 

For domains that do send email, it is considered [best practice to use SPF softfail](https://www.mailhardener.com/blog/why-mailhardener-recommends-spf-softfail-over-fail) (`~all`). This allows that the DKIM evaluation still occur even if SPF fails, helping ensure the email remains DMARC-compliant through DKIM alignment.

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

### How DKIM evaluation works
![IMAGE](/images/spf-dkim-dmarc-explanation/dkim-visual.png)

### Implementation of DKIM
You can configure DKIM with a `TXT` record in your DNS zone for your sending mail servers (such as postfix or sendmail). A DKIM generator can be used to generate a private key for your server and a public key for your DNS.

> **NOTE**: A DKIM record is required for each sending server or mail provider.

How you set up DKIM depends on your sending server or mail provider. For example, configuring DKIM for [Salesforce](https://help.salesforce.com/s/articleView?id=sales.emailadmin_create_secure_dkim.htm) differs from [Exchange Online](https://learn.microsoft.com/en-us/defender-office-365/email-authentication-dkim-configure) in Microsoft Defender for Office 365. If you’re managing your own mail server and using MTAs like Sendmail or Postfix, you can implement DKIM using tools such as [OpenDKIM](http://www.opendkim.org/).

### DKIM recommendations
- You must implement DKIM key rotation for each sending server or mail provider to prevent adversaries from intercepting and decrypting your DKIM keys. DKIM Key rotation helps to minimize the risk of having a private key compromised.
  - Recurring: Every six months for a 2048-bit DKIM key.
- The DKIM key length must be at least 2048-bits. 

## DMARC
### What is DMARC
DMARC (Domain-based Message Authentication, Reporting and Conformance) acts as a shield on top of SPF and DKIM. With DMARC, domain owners specify how to handle emails sent on behalf of their domain if those messages fail SPF and/or DKIM checks. This allows them to define whether such emails should be monitored, quarantined, or rejected entirely.

### How DMARC works
Once your domain has a DMARC record, any receiving email server can verify the incoming email based on the instructions in the DMARC policy. If the email passes the DMARC authentication, it will be delivered and can be trusted. If the email fails the check, depending on the instructions in the DMARC record, the email can be delivered, quarantined (flagged as potentially suspicious), or rejected.

DMARC will pass when the P1 and P2 sender domains are aligned, and at least one of SPF or DKIM passes with that aligned domain. If the P1 and P2 sender domains are not aligned, then DKIM must pass and align with the P2 domain for DMARC to pass.

Example of a DMARC pass with the P1 and P2 sender domain aligned:
![IMAGE](/images/spf-dkim-dmarc-explanation/dmarc-pass.png)

### How DMARC evaluation works
![IMAGE](/images/spf-dkim-dmarc-explanation/dmarc-visual.png)

> **IMPORTANT**: DMARC does not directly block or allow mail, it depends on how the receiving server is configured to handle messages that fail DMARC (SPF and/or DKIM alignment validation).

### Implementation of DMARC
Before enforcing a DMARC policy, it’s important to understand how often DMARC checks might fail for your domain. Jumping straight to a reject policy without visibility can cause legitimate email to be blocked.

You should start by monitoring DMARC failures. This can be done in two ways:

1. Manually reviewing DMARC `RUA` reports to identify which email providers are passing or failing DMARC. However, for high-volume domains, this can be overwhelming and difficult to manage.

RUA report example in `.xml`:

![IMAGE](/images/spf-dkim-dmarc-explanation/dmarc-xml.png)

2. Using DMARC monitoring tools allows you to convert `RUA` reports into clear visual dashboards, providing more actionable insights than raw IP address data alone. Most tools can associate sending IP addresses with known email providers, such as Microsoft 365 or Salesforce. One example of such a tool is [Valimail](https://www.valimail.com/blog/office-365-free-dmarc-monitoring/) (free for Microsoft 365 users with an Exchange Online plan). 
   - For personal use, you can use [Mailhardener’s free version](https://www.mailhardener.com/pricing), which supports one domain and offers one month of data retention. Mailhardener also offers good value for corporate use. While it’s not free, it supports hosting BIMI assets and MTA-STS policies, and includes SSO integration with Entra ID.

Overview of the Valimail Dashboard:

![IMAGE](/images/spf-dkim-dmarc-explanation/valimail-monitor.png)

> To sign up for a free Valimail environment, click [here](https://www.valimail.com/sign-up-ms-blog/). Once the setup is complete, you can sign in at [app.valimail.com](https://app.valimail.com/). After logging in, you can proceed to integrate [Microsoft Entra ID SSO with Valimail](https://support.valimail.com/en/articles/8466393-tutorial-how-to-integrate-microsoft-entra-id-sso-with-valimail).

For heavy mail domains, I recommended monitoring the domain for at least three months with the DMARC policy set to `none`:

- Hostname: `_dmarc`
- Value: `v=DMARC1; p=none; sp=none; rua=mailto:dmarc_agg@vali.email;`

During the monitoring phase, you can [adjust your SPF record](https://vand3rlinden.com/post/handle-your-spf-record/) and [set up DKIM for each sending server](https://vand3rlinden.com/post/spf-dkim-dmarc-explanation/#implementation-of-dkim), and then update your DMARC record to `reject` once the monitoring phase is complete:

- Value: `v=DMARC1; p=reject; sp=reject; rua=mailto:dmarc_agg@vali.email;`

The `sp=reject` tag means that subdomains will be included; if you don’t want your subdomains to be included in your domain’s root DMARC policy, you can set this to `sp=none` and list a separate DMARC policy for each subdomain (not recommended).

> **NOTE**: If you do not list the `sp=` tag, your subdomains will get the policy from the `p=` tag.

### Clarification on DMARC Monitoring
DMARC monitoring does **not** provide insights into the total sending volume of your domain. Instead, it **only** shows authentication results (SPF/DKIM/DMARC pass or fail) based on the aggregate (`RUA`) reports received.

There are three primary approaches to managing reporting in your DMARC monitoring strategy:
1. If you see an email provider you don’t recognize, but SPF and/or DKIM passes, and your organization hasn’t historically documented all authorized email providers, review your SPF and `*._domainkey` entries in your public DNS. Remove any unused entries, and establish a practice of documenting all email providers authorized to send on behalf of your domain.
2. If you recognize an email provider, but SPF and/or DKIM fails, update your domain’s email authentication settings to align SPF and DKIM for that email provider. This ensures the email provider is properly configured for outbound email authentication.
3. If the email provider is unrecognized and all authentication checks fail (SPF, DKIM, and DMARC), then DMARC is working as intended. No action is required, as these messages will be rejected by most receiving mail servers once your DMARC policy is set to `reject`.

### Maintenance steps on DMARC Monitoring
- SPF:
  - Perform periodic checks of the SPF record
  - Track and document any changes to the SPF record
    -	Maintain an up-to-date list of authorized senders

- DKIM:
  - Monitor for periodic DKIM key rotations
    - Recurring: Every six months for a 2048-bit DKIM key
  - Ensure timely updates and reconfiguration of rotated keys

- DMARC:
  - Regularly review DMARC reports and data
  - Set up alerting and reporting to detect high-volume email sources where DMARC validation fails due to SPF or DKIM misalignment
  
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
     - Combined data on a group of emails
     - Not real-time, they are sent everyday by default
     - Sent in `XML` format
     - No PII (Personal Identifiable Information)
     - Supported in all DMARC-compliant mailbox providers

- `RUF` (Reporting URI for Forensic reports): 
  - Is designed to send domain owners detailed failure reports when emails don’t pass DMARC checks. These reports may include portions of the original message headers and metadata, sometimes with limited message content, depending on the reporting provider. The goal is to help identify legitimate sources that need to be properly authenticated. However, due to privacy concerns and the potential exposure of Personal Identifiable Information (PII), most providers do not send `RUF` reports.
    - Details of an individual email
    - Sent almost immediately after the failures
    - Plain text format
    - Contains PII (Personal Identifiable Information)
    - Supported in only a handful of mailbox providers, due to the sensitive PII data it may contains

### Best Practices and Insights for RUA and RUF DMARC Reports
1. You don't need `RUF` Reporting to get a DMARC compliant domain, `RUA` is sufficient.

2. Not all receiving mail servers support sending both `RUF` and `RUA` reports. Most servers typically support `RUA`, but this is not guaranteed. If supported, the reports are sent to the addresses specified in the DMARC policy.

3. It’s not prohibited to include multiple email addresses in the `RUA` and `RUF` tags. However, best practice recommends limiting each tag to no more than two addresses, as some providers may ignore additional entries when sending DMARC reports. **My advice** is to limit this to one address whenever possible. If you do need to include more than one, you should format it like this: `rua=mailto:dmarc-reports@example.com,mailto:dmarc@example.com;`

4. DMARC External Validation for `RUA`/`RUF`: If you want to send DMARC reports to a domain different from your own, the receiving domain must explicitly authorize this by configuring a DNS record. This ensures that email providers recognize the recipient as an authorized destination for the reports.
   - For example, if you’re sending reports to `example.com`, that domain must create the following TXT record: `yourdomain.com._report._dmarc.example.com`, with the value: `v=DMARC1;`
   - DMARC monitoring providers, like Valimail, handle this automatically or use wildcard records on their end.

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

**DKIM**: verifies if the public key (DNS record) of a sending domain, matched the private key that came from the sending server. This is a check that the sending domain actually sent the e-mail. DKIM must be configured for **each** sending server, such as Exchange Online or any other email provider.
- **Purpose**: Message authenticity verification
- **Protect**: `RFC5322.From` (P2 Sender)

**DMARC**: ensures that emails failing SPF and/or DKIM validation do not get through, depending on how the receiving server is configured to handle the messages. If the email is unable to pass DMARC with SPF, DKIM can help pass DMARC for the P2 sender domain. Therefore, it is really important to have DKIM enabled for all your sending servers before setting DMARC to `reject`.
- **Purpose**: Final sheld on top of SPF and DKIM
- **Protect**: `RFC5322.From` (P2 Sender)

## Reference
- [Why Mailhardener recommends SPF softfail over fail](https://www.mailhardener.com/blog/why-mailhardener-recommends-spf-softfail-over-fail)
- [How to protect domains that do not send email](https://www.cloudflare.com/learning/dns/dns-records/protect-domains-without-email/)
- [SPF is defined in RFC 7208](https://www.rfc-editor.org/info/rfc7208)
- [DKIM is defined in RFC 6376](https://www.rfc-editor.org/info/rfc6376)
- [DMARC is defined in RFC 7489](https://www.rfc-editor.org/info/rfc7489)
- [SPF Policy Tester](https://vamsoft.com/support/tools/spf-policy-tester)