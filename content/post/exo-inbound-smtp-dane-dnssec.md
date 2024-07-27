---
title: "Exchange Online: Configure inbound SMTP DANE with DNSSEC"
date: 2024-07-28T01:13:44+02:00
draft: false
categories: ["DNS", "email"]
cover: 
  image: /images/exo-inbound-smtp-dane-dnssec/exo-inbound-smtp-dane-dnssec-front.png
---

> _In this post, you will learn how to enable and use SMTP DANE with DNSSEC in Exchange Online._

While ***outbound*** SMTP DANE with DNSSEC in Exchange Online has been enabled since 2022, Microsoft is currently rolling out ***inbound*** SMTP DANE with DNSSEC in Exchange Online. It is currently in public preview, with General Availability expected in October 2024. In an earlier [blog post](https://vand3rlinden.com/post/dnssec-dane-explained/#how-dnssec-and-dane-work-together-on-a-mailserver-25-smtp-dane), I explained how SMTP DANE with DNSSEC works together.

A short recap:
- **Outbound SMTP DANE with DNSSEC `sending mail server`**: Requests DANE `TLSA` records of the receiving domain's MX record.
- **Inbound SMTP DANE with DNSSEC `receiving mail server`**: Requires DNSSEC and DANE `TLSA` records that can be requested by the sending mail server.

## Inbound SMTP DANE with DNSSEC benefits
- Better protect your email domain from impersonation
- Help ensure your messages are delivered to the intended recipients using encryption and without being altered or redirected
- Enhance your email reputation by demonstrating compliance with the latest security standards

## Prerequisites
- You must have added the domain as an "Accepted Domain," and the domain status must be "Healthy" in the Microsoft 365 Admin Center. The current domain's MX record must have a priority of 0 or 10 and must not have a fallback or secondary MX record.

- Make sure that DNSSEC is enabled for your domain at your DNS provider.
 - You can use the [DNSSEC Analyzer](https://dnssec-analyzer.verisignlabs.com/) from VeriSign, to check if your DNS provider have DNSSEC enabled for your domain.

- You must be authorized to access Exchange Online PowerShell and to run the cmdlets.

- If the domain you want to secure with inbound SMTP DANE with DNSSEC is not referenced to an MX record such as `yourdomain-com.mail.protection.outlook.com`, you need to switch to this smarthost first.

## Set up inbound SMTP DANE with DNSSEC in Exchange Online
Below is a simplified version of the implementation compared to the official [Microsoft Learn documentation](https://learn.microsoft.com/en-us/purview/how-smtp-dane-works#set-up-inbound-smtp-dane-with-dnssec-in-preview).

1. Update the TTL of your existing MX record to the lowest possible value (not lower than 30 seconds). Then, wait for the previous TTL to expire before proceeding. For example, if the TTL of your existing MX record was '3600 seconds' or '1 hour' before you changed it, you need to wait 1 hour before proceeding.

2. Connect to Exchange Online PowerShell

3. Enable DNSSEC on your verified domain by running the cmdlet: `Enable-DnssecForVerifiedDomain -DomainName yourdomain.com`

4. Take the `DnssecMxValue` value, navigate to the DNS registrar hosting the domain, add a new MX record: `20 yourdomain-com.j-v1.mx.microsoft` and set the TTL to the lowest possible value (not lower than 30 seconds).

5. Verify that the new MX is working via the [Inbound SMTP Email test](https://testconnectivity.microsoft.com/tests/O365InboundSmtp/input)

6. Change the priority of the legacy MX record `yourdomain-com.mail.protection.outlook.com` from current priority to `30` (`30 yourdomain-com.mail.protection.outlook.com`)

7. Change the priority of the new MX record to `0` (`0 yourdomain-com.j-v1.mx.microsoft`)

8. Delete the legacy MX record `yourdomain-com.mail.protection.outlook.com`

9. Update the TTL for the new MX record `yourdomain-com.j-v1.mx.microsoft` to '3600 seconds' or '1 hour'

10. Enable SMTP DANE for that same domain once the DNSSEC enablement is complete by running the cmdlet: `Enable-SmtpDaneInbound -DomainName yourdomain.com`

11. Verify that the TLSA record has been propagated (this can take 15-30 minutes) by using the [DANE Validation in Microsoft Remote Connectivity Analyzer](https://testconnectivity.microsoft.com/tests/O365DaneValidation/input)

Once you have completed DNSSEC and SMTP DANE enablement, a successful output will be displayed in the DANE validation tool.

12. Check the health of your domain's MX record in the Microsoft 365 Admin Center under 'DNS Records'.


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

$DNSSEC = Get-DnssecStatusForVerifiedDomain -DomainName yourdomain.com
$DNSSEC.MtaStsValidation

# SMTP DANE
Get-SmtpDaneInboundStatus -DomainName yourdomain.com
```

## Reference
- [Announcing Public Preview of Inbound SMTP DANE with DNSSEC for Exchange Online](https://techcommunity.microsoft.com/t5/exchange-team-blog/announcing-public-preview-of-inbound-smtp-dane-with-dnssec-for/ba-p/4155257)
- [How SMTP DNS-based Authentication of Named Entities (DANE) works](https://learn.microsoft.com/en-us/purview/how-smtp-dane-works)
- [Releasing: Outbound SMTP DANE with DNSSEC in Exchange Online](https://techcommunity.microsoft.com/t5/exchange-team-blog/releasing-outbound-smtp-dane-with-dnssec/ba-p/3100920)
- [Support of DANE and DNSSEC in Office 365 Exchange Online](https://techcommunity.microsoft.com/t5/exchange-team-blog/support-of-dane-and-dnssec-in-office-365-exchange-online/ba-p/1275494)