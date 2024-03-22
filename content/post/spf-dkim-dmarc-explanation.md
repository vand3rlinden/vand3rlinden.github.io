---
title: "SPF, DKIM and DMARC explanation"
date: 2023-12-09T11:46:16+01:00
draft: false
categories: ["email"]
cover: 
  image: /images/spf-dkim-dmarc-explanation/spf-dkim-dmarc-explanation-front.png
---

> _In this post you will learn to understand how the DNS protocols SPF, DKIM and DMARC work together to protect your domain from phishers and spammers._

## SPF
### What is SPF
Sender Policy Framework (SPF) is a protocol that aims to reduce spam. SPF can reduce email spoofing and spam by determining if the sender is authorized to send on behalf of the listed sender.

### How SPF works
Imagine that you have an SPF record that looks like: 

```v=spf1 include:_spf.domain.com ip4:11.222.33.444 -all```

If an unauthorized server sends on behalf of your domain, the email will get an ```spf=fail``` in the message header because that IP is not listed in your SPF record.

SPF will pass if the sender’s IP is added to the SPF record for the P1 Sender domain.

### P1 vs P2- sender explanation:
| Postal Letter      | Precise Term                        | Protected by  |
| -----------        | -----------                         | -----------   |
| Sender on envelope | ```RFC5321.MailFrom``` (P1 Sender)  |  SPF          |
| Author on letter   | ```RFC5322.From``` (P2 Sender)      |  DKIM + DMARC |

### Implementation of SPF
When you add your domain to Microsoft 365, Microsoft will ask you to provide an SPF record, such as ```v=spf1 include:spf.protection.outlook.com -all```. If you have more allowed senders, you must include them in the SPF record. For example ```v=spf1 include:spf.protection.outlook.com include:_spf.domain.com ip4:11.222.33.444 -all```

The SPF record will differ for everyone; therefore, it is important to understand the following when implement an SPF- record:

- Can only have 255 characters, but it can be split to multiple strings in a single record, most DNS providers handle this automatically.
    - Example: ```"v=spf1 first string" "second string -all"```
- Can take up to a maximum of 10 DNS Lookups like ```include:spf.protection.outlook.com``` (which currently contains 1 DNS lookup, without child lookups)

### Avoiding the 10 DNS Lookups cap, to:
- Flatten your SPF record ***(not recommended)***
    - Example: ```v=spf1 include:spf.domain.com -all``` can be ```v=spf1 ip4:11.222.33.444 ip4:11.222.33.444 -all```

> The problem with flattening is that email service providers can change or add IP addresses without telling you. Then your SPF record will be inaccurate, leading to email delivery problems.

- Not using entries like ```a``` and ```mx```, these mechanisms are often useless and probably should not be included in your SPF record (and other duplicate SPF mechanisms).

- Where email is incapable of passing DMARC with SPF, configure DKIM for the P2 Sender domain on the sending server or mail provider.
  - ***NOTE:*** In this scenario, you need to set your SPF record to [softfail](https://vand3rlinden.com/post/spf-dkim-dmarc-explanation/#softfail-or-hardfail) ```~all```, so that the DMARC and DKIM evaluation is always performed in the absence of a valid SPF validation.

- Check if providers have the ability to pass SPF on a subdomain on behalf of your primary domain; passing SPF on a subdomain is still DMARC compliant.

- Move to vendor traffic with the use of subdomains for SPF authentication, subdomain segmentation creates a new domain dedicated to a particular mail stream with its own 10 DNS lookups. Organizations that segment their email streams find there is no need for SPF flattening and will end up with better controls, less attack surface, and limit any spillover effects of a potential cyber incident.

- Use an [SPF macro](https://vand3rlinden.com/post/handle-your-spf-record/#use-an-spf-macro-to-restrict-a-third-party-service-to-send-from-a-specific-address) to limit a third-party service to sending from a specific address, since services like Salesforce are most often limited to sending from a single email address, such as ```invoices@yourdomain.com```.

### Softfail or Hardfail
SPF can get a softfail or fail, you determine this at the end of the record.

- With ```~all```: The SPF record has designated the host as NOT allowed to send, but it is in transition (Accept but mark, Softfail).

- With ```-all```: The SPF record has designated the host as NOT being allowed to send (Reject, Hardfail).

Most mailbox providers will treat soft and hard- fails directives similarly, but it is [recommended](https://dmarcian.com/spf-best-practices/) to mirror the DMARC policy as the technology is deployed: use softfail (```~all```) if the DMARC policies are "none" and "quarantine", and use hardfail (```-all```) if you have moved to a "reject" policy. 

***NOTE:*** If an email cannot pass SPF, it can be rejected at the SMTP level with an SPF hardfail (-all), which may prevent DMARC and DKIM evaluation. If you are unsure whether your senders are passing SPF, then [consider using an SPF softfail (```~all```)](https://www.mailhardener.com/blog/why-mailhardener-recommends-spf-softfail-over-fail) along with DMARC set to reject (```p=reject```), so that the DMARC and DKIM evaluation is always performed in the absence of a valid SPF validation. For example, an email that fails SPF due to relaying may be hard rejected with an SPF hardfail (```-all```) before any DKIM or DMARC evaluation takes place.

### Limitations of SPF
Although SPF works relatively well in theory, there are several flaws in the protocol that mean that SPF alone is not enough to protect a sending domain.

The SPF protocol only protects the P1 sender and not the P2 sender of the email. DKIM was created to protect the P2 sender by adding a signature (SHA-256 hash function) to the email. But then there was the problem that not every mail server has an active policy for DKIM, which is why DMARC was created. With DMARC, the sender specifies on behalf of their domain what should be done with email that does not meet the DMARC requirements. DMARC protects the P2 sender as the last line of defense.

## DKIM
### What is DKIM
DomainKeys Identified Mail (DKIM) is an authentication-based technique, by using DKIM, the receiving mail server can check the authenticity of an e-mail with the DKIM-Signature header. This signature is created using the SHA-256 hash function.

### How DKIM works
The receiving server makes a DNS request using the sender’s domain name (P2 Sender). In response to the DNS request, the receiving server obtains the public key from a DNS record in the DNS zone of the sending domain, such as ```selector1._domainkey.yourdomain.com```, and compares it to the private key in the message from the sending server.

DKIM will pass if the sending server’s private key can be confirmed by the receiving server, using the public key in the sending domain’s DNS zone. This public key is published in a ```TXT``` or ```CNAME``` record. For example, when in ```TXT```:

- Hostname: ```key1._domainkey.yourdomain.com``` (in TXT)
- Value: ```v=DKIM1;t=s;p=MIIBIjANBgkqhkiG9w0BA...``` (public key)

### Implementation of DKIM
You can configure DKIM with a ```TXT``` record in your DNS zone for your sending mail servers (such as postfix or sendmail). A DKIM generator can be used to generate a private key for your server and a public key for your DNS.

> _Note: that a DKIM record is required for each sending server or mail provider._

How you set up DKIM can vary depending on your mail provider; setting up DKIM for [Salesforce](https://help.salesforce.com/s/articleView?id=sf.emailadmin_create_secure_dkim.htm&type=5) differs from [Exchange Online](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/email-authentication-dkim-configure?view=o365-worldwide#steps-to-create-enable-and-disable-dkim-from-microsoft-defender-portal) in Microsoft Defender for Office 365.

## DMARC
### What is DMARC
DMARC (Domain-based Message Authentication, Reporting and Conformance) acts as a shield on top of SPF and DKIM. With DMARC, the sender specifies what to do with email on behalf of the domain if it does not meet the requirements of SPF and DKIM.

### How DMARC works
Once your domain has a DMARC record, any receiving email server can verify the incoming email based on the instructions in the DMARC policy. If the email passes the DMARC authentication, it will be delivered and can be trusted. If the email fails the check, depending on the instructions in the DMARC record, the email can be delivered, quarantined, or rejected.

DMARC will pass if the P1 Sender and P2 Sender are equal, and/or SPF and DKIM are passed. If the P1 sender is not equal to the P2 sender, then DKIM must pass for the P2 sender domain in order to get a DMARC pass.

### Implementation of DMARC
Before using DMARC, you need to know how often it will fail. You should consider to start monitor the DMARC fails, before setting the DMARC policy directly on reject.

There are several monitoring tools that convert the DMARC reports into a clear dashboard. One such tool is [Valimail](https://www.valimail.com/blog/office-365-free-dmarc-monitoring/) (free for Microsoft 365 users with an Exchange Online plan). 

> ***NOTE:*** There are two associated report types - aggregate ```RUA``` and forensic ```RUF```. If the receiving mailbox provider supports sending DMARC reports, it can send the DMARC report to the address specified in the ```RUA``` and/or ```RUF``` tags. The differences between the reports are described [below](https://vand3rlinden.com/post/spf-dkim-dmarc-explanation/#rua-vs-ruf-dmarc-reports).

For heavy mail domains, I recommended monitoring the domain for at least three months with the DMARC policy set to ```none```:

- Hostname: ```_dmarc```
- Value: ```v=DMARC1; p=none; sp=none; rua=mailto:dmarc_agg@vali.email;```

After the monitoring phase, we set the DMARC policy to ```reject```:

- Value: ```v=DMARC1; p=reject; sp=reject; rua=mailto:dmarc_agg@vali.email;```

The ```sp=reject``` tag means that subdomains will be included; if you don’t want your subdomains to be included in your domain’s root DMARC policy, you can set this to sp=none and list a separate DMARC policy for each subdomain (not recommended).

If you do not list the ```sp=``` tag, your subdomains will get the policy from the ```p=``` tag.

### DMARC policy explanation
| Policy      | Value               | Meaning |
| ----------- | -----------         | -----------   |
| None        | ```p=none;```       |  This policy is essentially a monitoring or reporting mode. It instructs email receivers not to take any action based on the DMARC authentication results. |
| Quarantine  | ```p=quarantine;``` |  This policy instructs email receivers to quarantine emails that fail DMARC authentication. Instead of outright rejecting the email, it may be placed in a separate quarantine area or flagged as potentially suspicious. The DMARC quarantine policy will not take full advantage of DMARC. |
| Reject      | ```p=reject;```     |  This policy instructs email receivers to reject (not deliver) emails that fail DMARC authentication. |

The table on this [Microsoft Learn page](https://learn.microsoft.com/en-us/archive/blogs/fasttracktips/spf-dkim-dmarc-and-exchange-online#covering-the-basics-of-dmarc) summarizes the options you have when configuring your DMARC policy.

### RUA vs RUF DMARC Reports
```RUA``` reports are the most important and contain information about the authentication status for SPF, DKIM and DMARC. The ```RUA``` report doesn't contain any sensitive information from the email itself; the data is limited to message counts and email authentication attributes.

```RUF``` data was initially meant to give domain owners redacted copies of emails failing DMARC compliance. They use forensic reports to identify legitimate email sources needing remediation. However, due to privacy concerns, most DMARC reporters don't offer ```RUF``` reports due to the potential personally identifiable information (PII) that reports may contain. You don't need ```RUF``` Reporting to get a DMARC compliant domain, ```RUA``` is sufficient.

### SPF, DKIM and DMARC in short
SPF performs verification that the IP address of the sending server matches the entry in the SPF record from the sending domain.
- SPF protects the P1 sender domain (Envelope sender, ```RFC5321.MailFrom```).

DKIM verifies if the public key (DNS record) of a sending domain, matched the private key that came from the sending server. This is a check that the sending domain actually sent the e-mail. DKIM must be configured for ***each*** sending server, such as Exchange Online or any other server/SaaS service.
- DKIM protects the P2 sender domain (Letter sender, ```RFC5322.From```).

DMARC acts as a shield on top of SPF and DKIM. DMARC ensures that emails that fail the SPF and/or DKIM tests do not get through. If the email is unable to pass DMARC with SPF, DKIM can help pass DMARC for the P2 sender domain.
- DMARC protects the P2 sender domain (Letter sender, ```RFC5322.From```).

### Finalizing
To protect all non-sending domains, you should consider:
- a ***deny all SPF*** record:
  - Name: ```@```  
  - Content:```v=spf1 -all``` 
  - Type: ```TXT```
- a ***reject DMARC*** record:
  - Name: ```_dmarc``` 
  - Content: ```v=DMARC1; p=reject;```
  - Type: ```TXT```

This protects all of your domains from phishers and spammers, as bad actors will actively look for unused domains to exploit.

In my opinion, DKIM is unnecessary because SPF, when combined with DMARC, provides sufficient protection. This is due to the configuration of DKIM in both the sending source (private key) and DNS (public key) to ensure DKIM passes.

If desired, you could consider using a wildcard domainkey that covers all possible values for the selector, with an empty public key. This is because there is no public key to bind to.

- Name: ```*._domainkey```
- Content: ```v=DKIM1; p=```
- Type: ```TXT```

## Reference
- [dmarcian SPF best practices](https://dmarcian.com/spf-best-practices/)
- [Why Mailhardener recommends SPF softfail over fail](https://www.mailhardener.com/blog/why-mailhardener-recommends-spf-softfail-over-fail)
- [Concluding the Experiment: SPF Flattening](https://dmarcian.com/spf-flattening/)
- [dmarcian DMARC best practices](https://dmarcian.com/advancing-dmarc-policy/)
- [The Difference in DMARC Reports: RUA and RUF](https://dmarcian.com/rua-vs-ruf/)
- [How to protect domains that do not send email](https://www.cloudflare.com/learning/dns/dns-records/protect-domains-without-email/)