---
title: "Exchange Online: Configure inbound SMTP DANE with DNSSEC"
date: 2024-07-28T01:13:44+02:00
draft: false
categories: ["DNS", "email"]
cover: 
  image: /images/exo-inbound-smtp-dane-dnssec/exo-inbound-smtp-dane-dnssec-front.png
---

> _In this post, you will learn how to enable and use SMTP DANE with DNSSEC in Exchange Online._

While ***outbound*** SMTP DANE with DNSSEC in Exchange Online has been enabled since 2022, Microsoft is has rolling out ***inbound*** SMTP DANE with DNSSEC in Exchange Online since late 2024. In an earlier [blog post](https://vand3rlinden.com/post/dnssec-dane-explained/), I explained how SMTP DANE with DNSSEC works together on a mail and web server.

A short recap:
- **Outbound SMTP DANE with DNSSEC `sending mail server`**: Requests DANE `TLSA` records of the receiving domain's `MX` record.
- **Inbound SMTP DANE with DNSSEC `receiving mail server`**: Requires DNSSEC and DANE `TLSA` records that can be requested by the sending mail server.

## Inbound SMTP DANE with DNSSEC benefits
- **Authentication of TLS Certificates**: SMTP DANE ensures that the TLS certificates used in email exchanges are authenticated.
- **Reduction in Delivery Failures**: By using DANE, the sending mail server can verify that the recipient's server supports and prefers secure TLS connections.
- **Enhance Email Reputation**: Demonstrate that you comply with the latest security standards.
- **Integrity and Authenticity of DNS Records**: DNSSEC adds a layer of security to the DNS system by digitally signing DNS records. 

## Prerequisites
- Before you enable inbound SMTP DANE with DNSSEC in Exchange Online for a domain, you must have added the domain as an Accepted domain and the domain status must be Healthy in the Microsoft 365 Admin Center. The current domain's `MX` record must have a priority of `0` or `10` and must not have a fallback or secondary `MX` record.

- Make sure that DNSSEC is enabled for your domain at your DNS provider.
 - You can use the [DNSSEC Analyzer](https://dnssec-analyzer.verisignlabs.com/) from VeriSign, to check if your DNS provider have DNSSEC enabled for your domain.

- You must be authorized to access Exchange Online PowerShell and to run the cmdlets.

- The domain you want to secure with inbound SMTP DANE with DNSSEC must referenced to an `MX` record such as `yourdomain-com.mail.protection.outlook.com`.

- To configure inbound SMTP DANE with DNSSEC for your Accepted Domain, such as `yourdomain.com`, ensure that:
  - If this domain is referenced in any smarthost configurations, or in any connectors, you need to switch the smarthost name to `yourdomain-com.mail.protection.outlook.com`
  - Your business partners update their connectors to `yourdomain.onmicrosoft.com` or `yourdomain-com.mail.protection.outlook.com` to avoid failures. After enabling DANE, partners can switch to the new `yourdomain-com.j-v1.mx.microsoft` endpoint to restore the original connection.

## Set up inbound SMTP DANE with DNSSEC in Exchange Online
Below is a simplified version of the implementation compared to the official [Microsoft Learn documentation](https://learn.microsoft.com/en-us/purview/how-smtp-dane-works#set-up-inbound-smtp-dane-with-dnssec).

1. Update the `TTL` of your existing `MX` record to the lowest possible value (not lower than `30` seconds). Then, wait for the previous `TTL` to expire before proceeding. For example, if the `TTL` of your existing `MX` record was `3600` seconds or `1` hour before you changed it, you need to wait 1 hour before proceeding.

2. Connect to Exchange Online PowerShell

> If you're using [MTA-STS](https://vand3rlinden.com/post/mta-sts-explained/), you'll need to set your policy mode to `testing` during configuration, and set it back to `enforced` after configuration.

3. Enable DNSSEC on your accepted domain by running the cmdlet in PowerShell: `Enable-DnssecForVerifiedDomain -DomainName yourdomain.com`

4. Take the `DnssecMxValue` value, navigate to the DNS registrar hosting the domain, add a new `MX` record: `20 yourdomain-com.j-v1.mx.microsoft` and set the `TTL` to the lowest possible value (not lower than `30` seconds).

5. Verify that the new `MX` is working via the [Inbound SMTP Email test](https://testconnectivity.microsoft.com/tests/O365InboundSmtp/input)

6. Change the priority of the legacy `MX` record `yourdomain-com.mail.protection.outlook.com` from current priority to `30` (`30 yourdomain-com.mail.protection.outlook.com`)

7. Change the priority of the new `MX` record to `0` (`0 yourdomain-com.j-v1.mx.microsoft`)

8. Delete the legacy `MX` record `yourdomain-com.mail.protection.outlook.com`

9.  Update the `TTL` for the new `MX` record `yourdomain-com.j-v1.mx.microsoft` to `3600` seconds or `1` hour

10. Enable SMTP DANE for that same domain once the DNSSEC enablement is complete by running the cmdlet: `Enable-SmtpDaneInbound -DomainName yourdomain.com`

11. Verify that the TLSA record has been propagated (this can take 15-30 minutes) by using the [DANE Validation in Microsoft Remote Connectivity Analyzer](https://testconnectivity.microsoft.com/tests/O365DaneValidation/input)

Once you have completed DNSSEC and SMTP DANE enablement, a successful output will be displayed in the DANE validation tool.

12. Check the health of your domain's `MX` record in the Microsoft 365 Admin Center under 'DNS Records'.


## Cmdlets to get DNSSEC and SMTP DANE configuration settings in Exchange Online
```PowerShell
# DNSSEC
Get-DnssecStatusForVerifiedDomain -DomainName yourdomain.com | Select-Object DnssecFeatureStatus

$DNSSEC = Get-DnssecStatusForVerifiedDomain -DomainName yourdomain.com
$DNSSEC.ExpectedMxRecord

$DNSSEC = Get-DnssecStatusForVerifiedDomain -DomainName yourdomain.com
$DNSSEC.DnsValidation

$DNSSEC = Get-DnssecStatusForVerifiedDomain -DomainName yourdomain.com
$DNSSEC.MxValidation

# DNSSEC - MTA-STS Policy validation (check the validation only if you use an MTA-STS policy)
$DNSSEC = Get-DnssecStatusForVerifiedDomain -DomainName yourdomain.com
$DNSSEC.MtaStsValidation

# SMTP DANE
Get-SmtpDaneInboundStatus -DomainName yourdomain.com
```

## Check the TLSA record
The `TLSA` records are listed in: ` _25._tcp.yourdomain-com.j-v1.mx.microsoft`

![IMAGE](/images/exo-inbound-smtp-dane-dnssec/exo-inbound-smtp-dane-dnssec1.png)

## Activate TLS Reporting (TLSRPT)
TLS Reporting (TLSRPT) is a standard that provides a way to report when the TLS connection could not be established during email transmission.

### Implementation of TLSRPT
1. Log in to your DNS hosting provider's management console.
2. Add a new TXT record with the following details:

| Host                        | Type | Value                                   |
| ----                        | ---  | ---                                     |
| `_smtp._tls.example.com` | `TXT`| `v=TLSRPTv1; rua=mailto:tlsrpt@example.com`|

### TLSRPT report handling
If a sending mail server is having trouble securely delivering mail to a receiving mail server, the sending mail server can use the receiving mail server's TLSRPT record to find out where to send a report about the problem or to report a successful session.

The reports are received in `.json`, you can look for the `summary` tag to check if the TLS connection was failed or successful:
```
"summary":{"total-successful-session-count":1,"total-failure-session-count":0}
```

## Reference
- [Announcing General Availability of Inbound SMTP DANE with DNSSEC for Exchange Online](https://techcommunity.microsoft.com/blog/exchange/announcing-general-availability-of-inbound-smtp-dane-with-dnssec-for-exchange-on/4281292)
- [How SMTP DNS-based Authentication of Named Entities (DANE) works](https://learn.microsoft.com/en-us/purview/how-smtp-dane-works)
- [DNSSEC Analyzer](https://dnssec-analyzer.verisignlabs.com/)
- [TLSRPT is defined in RFC8460](https://datatracker.ietf.org/doc/html/rfc8460)