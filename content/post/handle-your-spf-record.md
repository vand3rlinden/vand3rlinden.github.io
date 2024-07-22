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

## The dangers of SPF flattening
The problem with SPF flattening is that email service providers can change or add IP addresses without notifying you. As a result, your SPF record becomes inaccurate, leading to complications with email delivery. Yes, there are paid tools available to automate this process. However, SPF flattening also increases the likelihood of forgetting to remove entries that are no longer needed, resulting in over-authorization.

From a security perspective, over-authorization of unnecessary SPF entries creates a potential attack vector. If an adversary gains access to any infrastructure listed in an SPF record, they can bypass your SPF and DMARC to send DMARC-compliant email.

When you segment your vendors and email streams, you will find that there is no need for SPF flattening because subdomain segmentation creates dedicated domains for specific email streams, each with its own 10 DNS lookups. Adopting SPF segmentation increases control, reduces attack surfaces, and mitigates the impact of potential cyber incidents.

SPF flattening attempts to work around the "too many DNS lookups" problem without addressing its underlying causes. Avoiding SPF record flattening will help you get a handle on your SPF record.

## How to get a handle on your SPF record
#### A quick win on how to handle your SPF record:
- Not using entries like `a` and `mx`, these mechanisms are often useless and probably should not be included in your SPF record (and other duplicate SPF mechanisms).

#### For the long term:
Imagine your organization has an SPF record on `yourdomain.com` with 9 of the 10 allowed DNS lookups, such as:
```
v=spf1 ip4:11.222.33.444 ip4:44.33.222.111 ip4:22.33.444.555 ip4:55.66.777.8 ip4:88.99.999.99 ip4:99.88.777.66 mx include:spf.protection.outlook.com include:_spf.salesforce.com include:mail.zendesk.com include:_spf.app1.com include:_spf.app2.com -all
```

Calculation of DNS lookups:

| DNS Lookup                           | Count          |
| -----------                          | -----------    |
| `include:spf.protection.outlook.com` | 1 DNS Lookup   |
| `include:_spf.salesforce.com`        | 2 DNS Lookups  |
| `include:mail.zendesk.com`           | 1 DNS Lookup   |
| `include:_spf.app1.com`              | 2 DNS Lookups  |
| `include:_spf.app2.com`              | 2 DNS Lookups  |
| `mx` (your MX record)                | 1 DNS Lookup   |
| Total:                               | 9 DNS Lookups  |

Effective segmentation of your email streams is essential to have a handle on the above.

In today's world, we're surrounded by numerous SaaS applications that use our primary domain for email correspondence. But is it really necessary for these applications to send through your primary domain when they could just as easily use a separate subdomain with its own SPF `TXT` record?

## Cut your SPF record
Given the information provided, determine which SaaS applications can be restricted to sending email from a specific address using an [SPF macro](https://vand3rlinden.com/post/handle-your-spf-record/#use-an-spf-macro-to-restrict-a-third-party-service-to-send-from-a-specific-address). Also, determine which services are required to send email through your primary domain from multiple addresses, such as Microsoft 365, and which services are not subject to such restrictions and can send through a subdomain. A helpful approach is to create a list of your SPF record entries and specify the desired behavior for each entry, as shown in the example below:

| Look up                               | Outcome     |
| -----------                           | ----------- |
| `include:spf.protection.outlook.com`  | Used multiple email addresses and must send through the primary domain `yourdomain.com`|
| `include:_spf.salesforce.com`         | Must send through the primary domain, but can be restricted to send from a specific address `invoices@yourdomain.com` using an SPF macro|
| `include:mail.zendesk.com`            | Must send through the primary domain, but can be restricted to send from a specific address `support@yourdomain.com` using an SPF macro|
| `include:_spf.app1.com`               | Used multiple addresses and can send through a subdomain with a new SPF `TXT` record for `app1.yourdomain.com`|
| `include:_spf.app2.com`               | Used multiple addresses and can send through a subdomain with a new SPF `TXT` record for `app2.yourdomain.com`|
| `mx`                                  | Duplicate mechanisms, can be removed. When using Microsoft 365, the `MX` endpoint IP is already listed in `include:spf.protection.outlook.com`|

If you look at the example above, we have ***2 DNS lookups*** left after cutting the current SPF record for the primary domain:
| Look up                               | Count        |
| -----------                           | -----------  |
| `include:spf.protection.outlook.com`  | 1 DNS Lookup |
| `include:%{l}._spf.yourdomain.com`    | 1 DNS Lookup, using an [SPF macro](https://vand3rlinden.com/post/handle-your-spf-record/#use-an-spf-macro-to-restrict-a-third-party-service-to-send-from-a-specific-address) for both Salesforce and Zendesk |

## IP address management in your SPF record
So we cleaned up ***7 DNS lookups*** from the previous ***9 DNS Lookups***, good job! But what about the IP addresses in the SPF record? IP addresses don’t cost any DNS lookups because we’re not talking to the DNS. One disadvantage of using IP addresses in your SPF record is that it will result in an unmanageable and too long record. So, we can add a new include with the cost of ***1 DNS lookup***, such as `include:_spf.yourdomain.com`.

1. In the SPF record for `yourdomain.com`, add the following DNS lookup:
```
include:_spf.yourdomain.com
```

2. Now we create a new `TXT` record in the DNS zone of `yourdomain.com` listing the IP addresses.

| Host                 | Type   | Value                                                                                                                |
| ---                  | ---    | ---                                                                                                                  |
| `_spf.yourdomain.com` | `TXT` | `v=spf1 ip4:11.222.33.444 ip4:44.33.222.111 ip4:22.33.444.555 ip4:55.66.777.8 ip4:88.99.999.99 ip4:99.88.777.66 -all`|


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

In order to bring this ***3 DNS Lookups*** to ***1 DNS Lookup***, you can follow the steps below.

1. In the SPF record for `yourdomain.com`, add the following DNS lookup at a cost of ***1 DNS Lookup***:
```
include:%{l}._spf.yourdomain.com
```

> The SPF macro `%{l}` replaced with the local-part of the sender's email address.

2. Now we will create two new `TXT` records in the DNS zone of `yourdomain.com` to restrict Salesforce to only send from `invoices@yourdomain.com` and Zendesk to only send from `support@yourdomain.com`.

| Host                           | Type   | Value                                    |
| ---                            | ---    | ---                                      |
| `invoices._spf.yourdomain.com` | `TXT`  | `v=spf1 include:_spf.salesforce.com -all`|
| `support._spf.yourdomain.com`  | `TXT`  | `v=spf1 include:mail.zendesk.com -all`   |

After setting up the above, Salesfroce's sending servers can only send from `invoices@yourdomain.com` and Zendesk can only send from `support@yourdomain.com`.

## How the SPF macro %{l} works on the receiving mail server
![IMAGE](/images/handle-your-spf-record/spf-macro-visual-l.png)

## To summarize what we have done
1. The main SPF record is cleaned up by deleting ***7 DNS lookups***, this with segmenting your email streams with subdomains and using SPF macros.
2. We deleted the `mx` DNS lookup because of a duplicate mechanism.
3. We have set all the IP addresses in a separate include.
4. We have restricted a third-party service to send from a specific address using an SPF macro.

Instead of ***9 DNS lookups*** before cleaning, the cleaned SPF record has only ***3 DNS lookups*** with SPF macros:
```
v=spf1 include:spf.protection.outlook.com include:_spf.yourdomain.com include:%{l}._spf.yourdomain.com -all
```

Final computation of DNS lookups:
| DNS Lookup                           | Count                                  |
| -----------                          | -----------                            |
| `include:spf.protection.outlook.com` | 1 DNS Lookup                           |
| `include:_spf.yourdomain.com`        | 1 DNS Lookup for the IP addresses      |
| `include:%{l}._spf.yourdomain.comm`  | 1 DNS Lookup for Salesforce and Zendesk|
| Total:                               | 3 DNS Lookups                          |

## Lastly
For the future of your SPF record, add IP addresses using the separate include, and carefully decide whether a SaaS application should be sent through a subdomain or a static address instead of any address from your primary domain. In addition, make it a habit to monitor your SPF record frequently and document each sender you list.

## Reference
- [Dmarcian SPF best practices - Unavailable from the Netherlands](https://dmarcian.com/spf-best-practices/)
- [Dmarcian Concluding the Experiment: SPF Flattening - Unavailable from the Netherlands](https://dmarcian.com/spf-flattening/)
- [Using SPF Macros to Solve the Operational Challenges of SPF](https://www.jamieweb.net/blog/using-spf-macros-to-solve-the-operational-challenges-of-spf/)
- [SPF Macros: Overcoming the 10 DNS Lookup Limit](https://www.uriports.com/blog/spf-macros-max-10-dns-lookups/)
- [SPF Policy Tester](https://vamsoft.com/support/tools/spf-policy-tester)