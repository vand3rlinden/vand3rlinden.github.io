---
title: 'Get a handle on your SPF record'
date: 2023-12-17T18:31:45+01:00
draft: false
categories: ["email", "DNS"]
cover: 
  image: /images/handle-your-spf-record/handle-your-spf-record-front.png
---

> _In this post, I will share my best practices for getting a handle on your SPF record._

## Why it makes sense to have a good SPF procedure in place
In a previous [blog post](https://vand3rlinden.com/post/spf-dkim-dmarc-explanation/), I explained the limitations of SPF and how it works with DKIM and DMARC. It's crucial to have a well-structured SPF procedure to avoid future problems, especially since exceeding the DNS lookup limit of 10 can cause issues, such as:

- Your domain is vulnerable to spoofing on the P1 sender domain (`RFC5321.MailFrom`)
- Your domain authentication or validation may fail (`spf=PermError`)
- Your emails will be undeliverable without any warning

Around the globe you will find an answer to bypass the DNS lookup limit with ***SPF flattening***. This approach converts hostnames to IP addresses, which don’t count in the DNS lookup count.

### P1 vs. P2 - Sender explanation table:
In this blog, you will read a lot about the P1 and P2 sender. Please refer to the table below to become familiar with these terms.

| Postal Letter                        | Precise Term                    | Protected by  |
| -----------                          | -----------                     | -----------   |
| Sender on envelope (Envelope sender) | `RFC5321.MailFrom` (P1 Sender)  |  SPF          |
| Author on letter (Header sender)     | `RFC5322.From` (P2 Sender)      |  DKIM + DMARC |

## The dangers of SPF flattening
The problem with SPF flattening is that email service providers can change or add IP addresses without notifying you. As a result, your SPF record becomes inaccurate, leading to complications with email delivery. Yes, there are paid tools available to automate this process. However, SPF flattening also increases the likelihood of forgetting to remove entries that are no longer needed, resulting in over-authorization.

From a security perspective, over-authorization of unnecessary SPF entries creates a potential attack vector. If an adversary gains access to any infrastructure listed in an SPF record, they can bypass your SPF and DMARC to send DMARC-compliant email.

When you segment your email providers using subdomain segmentation, SPF flattening becomes unnecessary. Each subdomain functions as a dedicated domain for a specific email provider, enabling it to use its own set of 10 DNS lookups.

Subdomain segmentation can be implemented in three ways:

1. Using a direct subdomain address, such `news.yourdomain.com`, where both the P1 sender and P2 sender use the subdomain (e.g., `news@news.yourdomain.com`).
2. Setting only the P1 sender to a subdomain (if supported by your email provider, such as SendGrid) allows SPF alignment in relaxed mode (assuming your DMARC policy uses the default `aspf=r` tag). For example, SPF aligns with subdomain `news.yourdomain.com` as the P1 sender, while the email is still sent from your primary domain `yourdomain.com` as the P2 sender (e.g., `news@yourdomain.com`).
   - DMARC policy tag `aspf=r` (SPF relaxed mode): In this mode, the authenticated signing domain and the sender domain can be subdomains of each other and still be considered aligned.

> **NOTE**: If your email provider does not allow setting the P1 sender to a subdomain, they might offer an option to pass SPF using their domain (e.g., `youremailprovider.com`). Keep in mind that this method requires DKIM alignment for your domain as the P2 sender. If DKIM is correctly set up for the P2 sender, the message can still meet DMARC requirements. However, relying only on DKIM is not best practice. If DKIM fails, for example due to slow DNS response, there is no backup. The best practice is to have both SPF and DKIM correctly aligned.

3. Using an SPF macro that points to a subdomain allows you to continue sending from your main domain, but only from a fixed/static sender address (e.g., `news@yourdomain.com`).

These subdomain segmentation options can be combined, as covered in this blog. Adopting SPF segmentation increases control, reduces attack surfaces, and mitigates the impact of potential cyber incidents.

> **NOTE**: SPF always checks the exact domain of the P1 sender. So, if you send from a subdomain, the receiving mail server looks for an SPF record on the subdomain and does not fall back to the rootdomain.

SPF flattening attempts to work around the **too many DNS lookups** problem without addressing its underlying causes. Avoiding SPF record flattening will help you get a handle on your SPF record.

## How to get a handle on your SPF record
#### A quick win on how to handle your SPF record:
- Not using entries like `a` and `mx`, these mechanisms are often useless and probably should not be included in your SPF record (and other duplicate SPF mechanisms).

#### For the long term:
Imagine your organization has an SPF record on `yourdomain.com` with 9 of the 10 allowed DNS lookups, such as:
```
v=spf1 ip4:11.222.33.444 ip4:44.33.222.111 ip4:22.33.444.555 ip4:55.66.777.8 ip4:88.99.999.99 ip4:99.88.777.66 mx include:spf.protection.outlook.com include:_spf.salesforce.com include:mail.zendesk.com include:_spf.app1.com include:_spf.app2.com ~all
```

> **NOTE**: IP addresses do not consume any DNS lookups and therefore they do not require DNS resolution (translating friendly domain names into IP addresses).

Calculation of DNS lookups:

| DNS Lookup                           | Count          |
| -----------                          | -----------    |
| `mx` (your MX record)                | 1 DNS Lookup   |
| `include:spf.protection.outlook.com` | 1 DNS Lookup   |
| `include:_spf.salesforce.com`        | 2 DNS Lookups  |
| `include:mail.zendesk.com`           | 1 DNS Lookup   |
| `include:_spf.app1.com`              | 2 DNS Lookups  |
| `include:_spf.app2.com`              | 2 DNS Lookups  |
| Total:                               | 9 DNS Lookups  |

Effective segmentation of your email providers is essential for maintaining control and clarity over your SPF record.

## Cut your SPF record
Based on the information provided, identify which email provider can be restricted to sending emails from a fixed sender address using an SPF macro. Additionally, determine which email provider, such as Microsoft 365, require sending emails from multiple addresses using your primary domain, and which email providers are not subject to these restrictions and can send through a subdomain, or where the email provider supports SPF alignment in relaxed mode through a subdomain.

A helpful approach is to create a list of your SPF record entries and specify the desired behavior for each entry, as shown in the example below:

| Look up                               | Outcome     |
| -----------                           | ----------- |
| `mx`                                  | Duplicate mechanisms, can be removed. When using Microsoft 365, the `MX` endpoint IP is already listed in `include:spf.protection.outlook.com`|
| `include:spf.protection.outlook.com`  | Uses multiple email addresses and must send through the primary domain `yourdomain.com`|
| `include:_spf.salesforce.com`         | Must send through the primary domain, but can be restricted to send from a fixed sender address `invoices@yourdomain.com` using an SPF macro|
| `include:mail.zendesk.com`            | Must send through the primary domain, but can be restricted to send from a fixed sender address `support@yourdomain.com` using an SPF macro|
| `include:_spf.app1.com`               | Uses multiple addresses and is capable of sending through a subdomain for both the P1 sender and P2 sender, with a new SPF `TXT` record configured for `app1.yourdomain.com`|
| `include:_spf.app2.com`               | Uses multiple addresses, but since the email provider supports setting the P1 sender to a subdomain, a new SPF `TXT` record is created for `app2.yourdomain.com`, but emails can still be sent using the primary domain `yourdomain.com` as the P2 sender|

> **NOTE**: If you encounter unfamiliar email providers or IP addresses, it may be the result of incomplete historical documentation of authorized email services within your organization. In such cases, monitor your SPF record using DMARC monitoring for 1 to 3 months, and only allow email providers that you can confidently verify as legitimate. For more details, see my clarification on DMARC monitoring in [this blog post](https://vand3rlinden.com/post/spf-dkim-dmarc-explanation/#clarification-on-dmarc-monitoring).

If you look at the example above, we have ***2 DNS lookups*** left after cutting the current SPF record for the primary domain:
| Look up                               | Count        |
| -----------                           | -----------  |
| `include:spf.protection.outlook.com`  | 1 DNS Lookup |
| `include:%{l}._spf.yourdomain.com`    | 1 DNS Lookup, using an SPF macro for both Salesforce and Zendesk _(configuration of this SPF macro is covered later in this blog)_|

## IP address management in your SPF record
We successfully reduced the SPF record from ***9 DNS lookups*** to ***2 DNS lookups***, great job!

Now, regarding IP addresses in the SPF record:
IP addresses do not consume any DNS lookups and therefore they do not require DNS resolution. However, including many IPs can make your SPF record lengthy and difficult to manage.

It is important to document each IP address and understand its origin. If you need to include many IPs (as discussed in this blog), consider offloading them to a separate include (e.g., `include:_spf.yourdomain.com`). This approach costs ***1 DNS lookup*** and helps keep your main SPF record more manageable.

> **NOTE**: If you only have a small number of IP addresses to list, such as two or three, it is not strictly necessary to place them in a separate include. You can let them stay into your main SPF record. However, it is important to place the `ip4` mechanism always immediately after the `v=spf1` tag. While this approach is not technically required, it is considered a best practice because it improves SPF efficiency by enabling faster DNS evaluation of straightforward matches before more resource intensive mechanisms are processed.

1. In the SPF record for `yourdomain.com`, add the following DNS lookup:
```
include:_spf.yourdomain.com
```

2. Now we create a new `TXT` record in the DNS zone of `yourdomain.com` listing the IP addresses.

| Host                 | Type   | Value                                                                                                                |
| ---                  | ---    | ---                                                                                                                  |
| `_spf.yourdomain.com` | `TXT` | `v=spf1 ip4:11.222.33.444 ip4:44.33.222.111 ip4:22.33.444.555 ip4:55.66.777.8 ip4:88.99.999.99 ip4:99.88.777.66 ~all`|

> **CAUTION:** When the SPF record for your IP addresses reaches its limit of [two strings of 255 characters](https://vand3rlinden.com/post/spf-dkim-dmarc-explanation/#implementation-of-spf), it becomes inaccurate. You should avoid including too many IP addresses. While you can add another include, such as `_spf1.yourdomain.com`, at the cost of another DNS lookup, it is advisable to start segmenting this into subdomains. Also, get into the habit of documenting all of your IP addresses that send mail on behalf of your domain.

> **NOTE:** There is also an SPF macro for IP addresses, `%{i}`, this macro replace the IP address of the SMTP client that submitted the message. However, using two separate SPF macros _(because this blog already uses the macro `%{l}`)_ is not advisable due to the limit of two allowed void lookups (`NXDomain`). Even if you stay within the limit, there is still a risk of DNS timeouts due to slow DNS responses. Exceeding the limit will result in SPF `permerror`. Publishing an SPF policy that refers to data that does not exist in DNS is a poor practice and raises security concerns (see [RFC7208](https://www.rfc-editor.org/info/rfc7208) Section 4.6.4.).

## Use an SPF macro to restrict a third-party service to send from a specific address
As described above, third-party services like Salesforce and Zendesk are mostly limited to sending from a single email address, such as `invoices@yourdomain.com` and `support@yourdomain.com`.
So it is unnecessary to have `include:_spf.salesforce.com` and `include:mail.zendesk.com` cost ***3 DNS Lookups*** in the SPF record on `yourdomain.com`.

Calculate the DNS Lookups from `include:_spf.salesforce.com` in `yourdomain.com`:
| DNS Lookup                           | Count                      |
| -----------                          | -----------                |
| `include:_spf.salesforce.com`        | 1 DNS Lookup               |
| `exists:%{i}._spf.mta.salesforce.com`| 1 DNS Lookup (child lookup)|
| Total:                               | ***2 DNS Lookups***        |

Calculate the DNS Lookup from `include:mail.zendesk.com` in `yourdomain.com`:
| DNS Lookup                           | Count               |
| -----------                          | -----------         |
| `include:mail.zendesk.com`           | 1 DNS Lookup        |
| Total:                               | ***1 DNS Lookup***  |

To reduce the ***3 DNS Lookups*** to ***1 DNS Lookup***, we will use the SPF macro `%{l}`, which replaces the local-part of the sender’s email address. Follow the two steps below to implement this.

1. In the SPF record for `yourdomain.com`, add the following DNS lookup at a cost of ***1 DNS Lookup***:
```
include:%{l}._spf.yourdomain.com
```

> **CAUTION**: Always place the `{l}` SPF macro at the end of your SPF record, as SPF evaluation processes this macro last during policy validation. Placing it elsewhere can lead to SPF permerrors, regardless of any subsequent includes in your SPF record.

2. Now we will create two new `TXT` records in the DNS zone of `yourdomain.com` to restrict Salesforce to only send from `invoices@yourdomain.com` and Zendesk to only send from `support@yourdomain.com`.

| Host                           | Type   | Value                                    |
| ---                            | ---    | ---                                      |
| `invoices._spf.yourdomain.com` | `TXT`  | `v=spf1 include:_spf.salesforce.com ~all`|
| `support._spf.yourdomain.com`  | `TXT`  | `v=spf1 include:mail.zendesk.com ~all`   |

After setting up the above, Salesfroce's sending servers can only send from `invoices@yourdomain.com` and Zendesk can only send from `support@yourdomain.com`.

## How the SPF macro %{l} works on the receiving mail server
![IMAGE](/images/handle-your-spf-record/spf-macro-visual-l.png)

## To summarize what we have done
1. The main SPF record is cleaned up by deleting ***7 DNS lookups***, this with segmenting your email providers with subdomains and using an SPF macro for fixed sender addresses.
2. We deleted the `mx` DNS lookup because of a duplicate mechanism.
3. We have set all the IP addresses in a separate include.

Instead of ***9 DNS lookups*** before cleaning, the cleaned SPF record has only ***3 DNS lookups*** with SPF macros:
```
v=spf1 include:spf.protection.outlook.com include:_spf.yourdomain.com include:%{l}._spf.yourdomain.com ~all
```

Final computation of DNS lookups:
| DNS Lookup                           | Count                                  |
| -----------                          | -----------                            |
| `include:spf.protection.outlook.com` | 1 DNS Lookup                           |
| `include:_spf.yourdomain.com`        | 1 DNS Lookup for the IP addresses      |
| `include:%{l}._spf.yourdomain.comm`  | 1 DNS Lookup for Salesforce and Zendesk|
| Total:                               | 3 DNS Lookups                          |

## Lastly
For the future of your SPF record, add IP addresses using the separate include, and carefully decide whether a email provider should be sent through a subdomain or a fixed sender address instead of any address from your primary domain. In addition, make it a habit to monitor your SPF record frequently and document each sender you list.

## Reference
- [Dmarcian SPF best practices - Unavailable from the Netherlands](https://dmarcian.com/spf-best-practices/)
- [Dmarcian Concluding the Experiment: SPF Flattening - Unavailable from the Netherlands](https://dmarcian.com/spf-flattening/)
- [Using SPF Macros to Solve the Operational Challenges of SPF](https://www.jamieweb.net/blog/using-spf-macros-to-solve-the-operational-challenges-of-spf/)
- [SPF Macros: Overcoming the 10 DNS Lookup Limit](https://www.uriports.com/blog/spf-macros-max-10-dns-lookups/)
- [SPF Policy Tester](https://vamsoft.com/support/tools/spf-policy-tester)