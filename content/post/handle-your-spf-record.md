---
title: 'Get a handle on your SPF record'
date: 2023-12-17T18:31:45+01:00
draft: false
categories: ["email"]
cover: 
  image: /images/handle-your-spf-record/handle-your-spf-record-front.png
---

> _In this post, I will share my best practices for getting a handle on your SPF record._

## Why it makes sense to have a good SPF procedure in place
In a previous [blog post](https://vand3rlinden.com/post/spf-dkim-dmarc-explanation/), I explained the limitations of SPF, best practices and how it works with DKIM and DMARC. It is useful to have a good SPF procedure to avoid future problems, because if you exceed the DNS lookup limit of 10:

- Your domain is vulnerable to spoofing on the P1 sender domain (```RFC5321.MailFrom```)
- Your domain authentication or validation may fail (```spf=PermError```)
- Your emails will be undeliverable without any warning

Around the globe you will find an answer to bypass the DNS lookup limit with ***SPF flattening***. This approach converts hostnames to IP addresses, which don’t count in the DNS lookup count.

## The dangers of SPF flattening
The problem with SPF flattening is that email service providers can change or add IP addresses without notifying you. As a result, your SPF record becomes inaccurate, leading to complications with email delivery. Yes, there are paid tools available to automate this process. However, SPF flattening also increases the likelihood of forgetting to remove entries that are no longer needed, resulting in over-authorization.

From a security perspective, over-authorization of unnecessary SPF entries creates a potential attack vector. If an adversary gains access to any infrastructure listed in an SPF record, they can bypass your SPF and DMARC to send DMARC-compliant email.

When you segment your vendors and email streams, you will find that there is no need for SPF flattening because subdomain segmentation creates dedicated domains for specific email streams, each with its own 10 DNS lookups. Adopting SPF segmentation increases control, reduces attack surfaces, and mitigates the impact of potential cyber incidents.

SPF flattening attempts to work around the "too many DNS lookups" problem without addressing its underlying causes. Avoiding SPF record flattening will help you get a handle on your SPF record.

## How to get a handle on your SPF record
#### Quick wins on how to handle your SPF record:
- Not using entries like ```a``` and ```mx```, these mechanisms are often useless and probably should not be included in your SPF record (and other duplicate SPF mechanisms).

- Where email is incapable of passing DMARC with SPF, configure DKIM for the P2 Sender domain on the sending email stream.

#### For the long term:
Imagine your organization has an SPF record with 9 of the 10 allowed DNS lookups, such as:
```
v=spf1 ip4:11.222.33.444 ip4:44.33.222.111 ip4:22.33.444.555 ip4:55.66.777.8 ip4:88.99.999.99 ip4:99.88.777.66 mx include:spf.protection.outlook.com include:_spf.app1.com include:_spf.app2.com include:_spf.app3.com include:_spf.app4.com -all
```

Calculation of DNS lookups:

| Look up                                  | Count           |
| -----------                              | -----------     |
| ```include:spf.protection.outlook.com``` | 1 DNS Look Ups  |
| ```include:_spf.app1.com```              | 2 DNS Look Ups  |
| ```include:_spf.app2.com```              | 1 DNS Look Ups  |
| ```include:_spf.app3.com```              | 2 DNS Look Ups  |
| ```include:_spf.app4.com```              | 2 DNS Look Ups  |
| ```mx``` (your MX record)                | 1 DNS Look Ups  |
| Total:                                   | 9 DNS Look Ups  |

Effective segmentation of your email streams is essential to have a handle on the above.

In today's world, we're surrounded by numerous SaaS applications that use our primary domain for email correspondence. But is it really necessary for these applications to send through your primary domain when they could just as easily use a separate subdomain with its own SPF (TXT) record?

Consider the newsletter scenario. Does it really need to send emails on behalf of your primary domain, or could it just as effectively send them from an address like noreply@news.yourdomain.com?

> ***NOTE***: Some SaaS applications, such as SendGrid, allow SPF to pass through a subdomain as the P1 sender, while still allowing you to use your primary domain as the P2 sender. Similarly, when utilizing sendmail in Unix, you have the flexibility to configure a subdomain as the P1 sender while maintaining the ability to send emails using your primary domain as P2 sender. This approach complies with DMARC standards, and in this setup, DKIM validation for the P2 sender is not necessary, although it is always wise to set up DKIM for enhanced security measures.

## Cut your SPF record
With the above in mind, determine which SaaS applications ***need*** to send on behalf of your primary domain, such as Microsoft 365, and which ***don’t***. Like your newsletter service or an internal application, for example:

| Look up                                   | Outcome                           |
| -----------                               | -----------                       |
| ```include:spf.protection.outlook.com```  | Need to send over yourdomain.com  |
| ```include:_spf.app1.com```               | Need to send over yourdomain.com  |
| ```include:_spf.app2.com``` (delete)      | Can send over app2.yourdomain.com |
| ```include:_spf.app3.com``` (delete)      | Can send over app3.yourdomain.com |
| ```include:_spf.app4.com``` (delete)      | Can send over news.yourdomain.com |
| ```mx``` (delete)*                        | Duplicate mechanisms              |
> *when using Microsoft 365, the MX endpoint IP is already listed in _include:spf.protection.outlook.com_

If you look at the example above, we have ***3 DNS lookups*** left, so we cleaned ***6 DNS lookups***, good job! But what about the IP addresses in the SPF record? IP addresses don’t cost any DNS lookups because we’re not talking to the DNS. One disadvantage of using IP addresses in your SPF record is that it will result in an unmanageable and too long record. Yes, we can add a new include with the cost of 1 DNS lookup, such as ```include:_spf.yourdomain.com``` with a new SPF (TXT) record, for example:

SPF for ```yourdomain.com```:
```
v=spf1 include:_spf.yourdomain.com -all 
```
SPF for ```_spf.yourdomain.com```:
```
v=spf1 ip4:11.222.33.444 ip4:44.33.222.111 ip4:22.33.444.555 ip4:55.66.777.8 ip4:88.99.999.99 ip4:99.88.777.66 -all
```

But without being afraid of needing another DNS lookup when the ```_spf.yourdomain.com``` include reaches its limit of two strings of 255 characters. You can start by using a SPF macro instead. This macro also costs 1 DNS lookup, but you can add an unlimited number of IP addresses _(not recommended, but you get my point)_ to it by creating a separate A record for each ```/32``` IP address.

## Using an SPF macro for your IP addresses
1. In the SPF record for ```yourdomain.com```, add the following DNS lookup:
```
exists:%{i}._spf.yourdomain.com
```

2. Now, for each IP address, such as ```11.222.33.444```, we will create an ```A``` record that is not publicly routable (such as to 127.0.0.2) and a ```TXT``` record. The ```TXT``` record is set to list the source of the IP address and can have any value, which is ***optional*** to use.

| Host                                    | Type       | Value           |
| ---                                     | ---        | ---             |
| ```11.222.33.444._spf.yourdomain.com``` | ```A```    | ```127.0.0.2``` |
| ```11.222.33.444._spf.yourdomain.com``` | ```TXT```  | ```Internal```  |

You can repeat this process for each IP address you need to add.

If you’re not comfortable with setting a source in public DNS, this option is ***optional*** for the above SPF macro. In my opinion, there should be no security concerns if you use a minimal listed source, such as _vendor name_ or _internal_. For the internal value (if you are using an Azure VM), you can check in Azure which PIP is bound to a NIC for example. Also, the IP addresses are not directly visible using this SPF macro, as you can see in the SPF record from step 1. So the direct hostname is not easy to guess, and the purpose of these systems is probably already easy to identify in other ways, such as an Nmap scan for open ports.

## In summary
1. The main SPF record is cleaned up by deleting 6 DNS lookups, this with segmenting your vendors and email streams with subdomains.
2. We deleted the ```mx``` DNS lookup because of a duplicate mechanism.
3. We have a good and secure way to add an unlimited number of IP addresses to the SPF record using an SPF macro.

Instead of 9 DNS lookups before cleaning, the cleaned SPF record has only 4 DNS lookups with an SPF macro:
```
v=spf1 exists:%{i}._spf.yourdomain.com include:spf.protection.outlook.com include:_spf.app1.com -all
```

## Lastly
For the future of your SPF record, add IP addresses to your main SPF record using the SPF macro we set up, and choose carefully if a SaaS application should be sent through a subdomain instead of your main domain. Also, make it a habit to monitor your SPF record frequently.

## Reference
- [dmarcian SPF best practices](https://dmarcian.com/spf-best-practices/)
- [Concluding the Experiment: SPF Flattening](https://dmarcian.com/spf-flattening/)