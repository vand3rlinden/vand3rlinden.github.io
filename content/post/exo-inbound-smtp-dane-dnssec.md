---
title: "Exchange Online: Configure inbound SMTP DANE with DNSSEC"
date: 2024-07-28T01:13:44+02:00
draft: false
categories: ["DNS", "email"]
cover: 
  image: /images/exo-inbound-smtp-dane-dnssec/exo-inbound-smtp-dane-dnssec-front.png
---

> _In this post, you will learn how to enable and use SMTP DANE with DNSSEC in Exchange Online._

While ***outbound*** SMTP DANE with DNSSEC in Exchange Online has been enabled since 2022, Microsoft is has rolling out ***inbound*** SMTP DANE with DNSSEC in Exchange Online since late 2024.

> For a deeper understanding of DNSSEC and DANE, take a look at my earlier [blog post](https://vand3rlinden.com/post/dnssec-dane-explained/).

## How SMTP DANE with DNSSEC works
SMTP DANE is a security protocol that uses DNSSEC to verify the authenticity of TLS certificates used for securing email communication. It helps protect against attacks such as TLS downgrade and man-in-the-middle attacks by ensuring that the certificates and encryption settings used in mail server communications are authentic and trustworthy.

While SPF, DKIM, and DMARC focus on verifying the authenticity of email messages and ensuring they are sent from authorized domains, SMTP DANE focuses specifically on securely establishing TLS connections between mail servers. By leveraging DNSSEC to publish certificate information directly in DNS, SMTP DANE ensures that the sending mail server connects to the intended receiving mail server with verified encryption, enhancing the overall security of email transport.

## The flow of SMTP DANE on a mailserver
- **Outbound SMTP DANE with DNSSEC `sending mail server`**: Requests DANE `TLSA` records of the receiving domain's `MX` record.
- **Inbound SMTP DANE with DNSSEC `receiving mail server`**: Requires DNSSEC and DANE `TLSA` records that can be requested by the sending mail server.
- **TLS Reporting (TLSRPT)**: If the sending mail server encounters issues delivering an email, it can use the receiving server’s `TLSRPT` record to report the problem or confirm that the TLS session was successfully established.

![IMAGE](/images/exo-inbound-smtp-dane-dnssec/smtpdane-tlsrpt-visual.png)

## Prerequisites
- Before you enable inbound SMTP DANE with DNSSEC in Exchange Online for a domain, you must have added the domain as an Accepted domain and the domain status must be Healthy in the Microsoft 365 Admin Center. The current domain's `MX` record must have a priority of `0` or `10` and must not have a fallback or secondary `MX` record.

- Make sure that DNSSEC is enabled for your domain at your DNS provider.
  - You can use the [DNSSEC Analyzer](https://dnssec-analyzer.verisignlabs.com/) from VeriSign, to check if your DNS provider have DNSSEC enabled for your domain.

- You must be authorized to access Exchange Online PowerShell and to run the cmdlets.

- The domain you want to secure with inbound SMTP DANE with DNSSEC must referenced to an `MX` record such as `yourdomain-com.mail.protection.outlook.com`.

- To configure inbound SMTP DANE with DNSSEC for your Accepted Domain, such as `yourdomain.com`, ensure that:
  - If this domain is referenced in any smarthost configurations, or in any connectors, you need to switch the smarthost name to `tenantname.mail.protection.outlook.com`
  - Your business partners update their connectors to `tenantname.onmicrosoft.com` or `tenantname.mail.protection.outlook.com` to avoid failures. After enabling DANE, partners can switch to the new `yourdomain-com.<random>.mx.microsoft` endpoint to restore the original connection.

## Set up inbound SMTP DANE with DNSSEC in Exchange Online
Below is a simplified version of the implementation compared to the official [Microsoft Learn documentation](https://learn.microsoft.com/en-us/purview/how-smtp-dane-works#set-up-inbound-smtp-dane-with-dnssec).

1. Update the `TTL` of your existing `MX` record to the lowest possible value (not lower than `30` seconds). Then, wait for the previous `TTL` to expire before proceeding. For example, if the `TTL` of your existing `MX` record was `3600` seconds or `1` hour before you changed it, you need to wait 1 hour before proceeding.

![IMAGE](/images/exo-inbound-smtp-dane-dnssec/exo-inbound-smtp-dane-dnssec1.png)

2. Connect to Exchange Online PowerShell

> If you're using [MTA-STS](https://vand3rlinden.com/post/mta-sts-explained/), you'll need to set your policy mode to `testing` during configuration, and set it back to `enforced` after configuration.

3. Enable DNSSEC on your accepted domain by running the cmdlet in PowerShell: `Enable-DnssecForVerifiedDomain -DomainName yourdomain.com`
  
![IMAGE](/images/exo-inbound-smtp-dane-dnssec/exo-inbound-smtp-dane-dnssec2.png)

4. Take the `DnssecMxValue` value, navigate to the DNS registrar hosting the domain, add a new `MX` record: `20 yourdomain-com.<random>.mx.microsoft` and set the `TTL` to the lowest possible value (not lower than `30` seconds).

![IMAGE](/images/exo-inbound-smtp-dane-dnssec/exo-inbound-smtp-dane-dnssec3.png)

5. Verify that the new `MX` is working via the [Inbound SMTP Email test](https://testconnectivity.microsoft.com/tests/O365InboundSmtp/input)

![IMAGE](/images/exo-inbound-smtp-dane-dnssec/exo-inbound-smtp-dane-dnssec4.png)

6. Change the priority of the legacy `MX` record `yourdomain-com.mail.protection.outlook.com` from current priority to `30` (`30 yourdomain-com.mail.protection.outlook.com`)

![IMAGE](/images/exo-inbound-smtp-dane-dnssec/exo-inbound-smtp-dane-dnssec5.png)

> Wait a minute

7. Change the priority of the new `MX` record to `0` (`0 yourdomain-com.<random>.mx.microsoft`)

![IMAGE](/images/exo-inbound-smtp-dane-dnssec/exo-inbound-smtp-dane-dnssec6.png)

> Wait a minute

8. Delete the legacy `MX` record `yourdomain-com.mail.protection.outlook.com`

![IMAGE](/images/exo-inbound-smtp-dane-dnssec/exo-inbound-smtp-dane-dnssec7.png)

> Wait a minute

9.  Update the `TTL` for the new `MX` record `yourdomain-com.<random>.mx.microsoft` to `3600` seconds or `1` hour

![IMAGE](/images/exo-inbound-smtp-dane-dnssec/exo-inbound-smtp-dane-dnssec8.png)

10. Enable SMTP DANE for that same domain once the DNSSEC enablement is complete by running the cmdlet: `Enable-SmtpDaneInbound -DomainName yourdomain.com`

![IMAGE](/images/exo-inbound-smtp-dane-dnssec/exo-inbound-smtp-dane-dnssec9.png)

11. Verify that the TLSA record has been propagated (this can take 15-30 minutes) by using the [DANE Validation in Microsoft Remote Connectivity Analyzer](https://testconnectivity.microsoft.com/tests/O365DaneValidation/input)

![IMAGE](/images/exo-inbound-smtp-dane-dnssec/exo-inbound-smtp-dane-dnssec10.png)

12. Check the health of your domain's `MX` record in the Microsoft 365 Admin Center under 'DNS Records'.

![IMAGE](/images/exo-inbound-smtp-dane-dnssec/exo-inbound-smtp-dane-dnssec11.png)

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
The `TLSA` records are listed in: `_25._tcp.yourdomain-com.<random>.mx.microsoft`

![IMAGE](/images/exo-inbound-smtp-dane-dnssec/exo-inbound-smtp-dane-dnssec12.png)

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
- [TLS-RPT Record Checker](https://easydmarc.com/tools/tls-rpt-check)