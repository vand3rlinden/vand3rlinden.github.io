---
title: 'Exchange Online: Reject Direct Send'
date: 2025-08-07T15:03:37+02:00
draft: false
categories: ["email"]
cover: 
  image: /images/exo-reject-direct-send/exo-reject-direct-send-front.png
---

> _In this post, you will learn what Direct send is and how to reject Direct Send._

Direct Send is a method used to send emails directly to Exchange Online hosted mailboxes from on-premises devices, applications, or third-party cloud services, using the MX record endpoint of your accepted domain in Exchange Online.

This method assumes that SPF, DKIM, and DMARC are properly configured for your accepted domain. Any sender using Direct Send without being included in the SPF record of the accepted domain will already struggle to deliver messages successfully to your internal inboxes.

However, there are situations where a potentially malicious email can still be successfully delivered to internal mailboxes when Direct Send is enabled. For example:
- If SPF, DKIM, and DMARC validation may fail, but the email can still be accepted if [implicit email authentication](https://vand3rlinden.com/post/mdo-handling-false-positives-false-negatives/#how-inbound-email-works-in-microsoft-365) passes. This mechanism goes beyond traditional authentication methods by using additional signals to determine the final verdict for inbound messages.
- If additional protections, such as Spoof Intelligence and the ‘Honor DMARC policy’ settings (if a message is detected as Spoof by Spoof Intelligence), are not enabled in your anti-phishing policy.

Since most tenants do not rely on Direct Send, Microsoft has introduced a setting **(in Public Preview)** to disable Direct Send. Turning off this feature helps block bad actors from spoofing your accepted domains in Exchange Online and sending emails to your internal mailboxes.

## Direct Send and third-party services
Direct Send traffic may include third-party services that you have authorized to use your domain, or email applications hosted on-premises. To ensure these messages are not rejected when Direct Send is disabled, they must be properly authenticated.

This can be achieved using SMTP relay, along with a partner mail flow connector that matches either the certificate used to send the messages (recommended) or the source IP addresses.

To configure SMTP relay, you can follow the guidance provided in the following Microsoft Learn articles:

- [Configure a TLS certificate-based connector for SMTP relay (recommended)](https://learn.microsoft.com/en-us/exchange/mail-flow-best-practices/how-to-set-up-a-multifunction-device-or-application-to-send-email-using-microsoft-365-or-office-365#configure-a-tls-certificate-based-connector-for-smtp-relay) 

- [Configure an IP address-based connector for SMTP relay](https://learn.microsoft.com/en-us/exchange/mail-flow-best-practices/how-to-set-up-a-multifunction-device-or-application-to-send-email-using-microsoft-365-or-office-365#configure-an-ip-address-based-connector-for-smtp-relay)


## Reject Direct Send with Exchange Online Powershell
1. Connect to Exchange Online Powershell: `Connect-ExchangeOnline`
2. Block Direct Send: `Set-OrganizationConfig -RejectDirectSend $true`
3. Validate the status: `Get-OrganizationConfig | Select-Object Identity, RejectDirectSend`

If needed, you can roll back the change to allow Direct Send again: `Set-OrganizationConfig -RejectDirectSend $false`

## How an adversary can abuse DirectSend for Phishing
For this test, I set up a new demo tenant with the following configuration:

- **Anti-phishing policy**: Spoof Intelligence is enabled
- **Applied action**: If a message is detected as spoofed by Spoof Intelligence > Quarantine the message
- **Quarantine policy**: DefaultFullAccessWithNotificationPolicy
- **RejectDirectSend**: `$false`
- The domain of the victum is not DMARC-compliant

### DirectSend Allowed (default for existing tenants)
In this scenario, an attacker attempts to phish a user in the demo tenant by trying sending an email directly to the inbox from a malicious server, where port 25 is open for outbound traffic. The attacker follows these steps:

1. The attacker first obtains the MX record of the target domain. They notice that the MX record points to Microsoft 365. Since they cannot tell whether DirectSend is enabled, they decide to attempt delivery.

2. Using the **outdated** `Send-MailMessage` cmdlet, the attacker sends a malicious attachment to the target. This is done with the following PowerShell script:

```
$PSEmailServer = "yourdomain-com.mail.protection.outlook.com"
$SMTPPort = 25
$MailTo = "scooby.doo@yourdomain.com"
$MailFrom = "Shaggy Rogers <shaggy@yourdomain.com>" #This email address does not require a mailbox, can be any value before the @ symbol
$MailSubject = "Test email"
$MailBody = "Hi Scooby, please help me out with this test."
$Attachment = "C:/temp/test.docx"
Send-MailMessage -From $MailFrom -To $MailTo -Attachments $Attachment -Subject $MailSubject -Body $MailBody -Port $SMTPPort -WarningAction "SilentlyContinue"
```

3. The malicious server from the attacker is not authorized to send email for `yourdomain.com`, because its public IP is missing from the SPF record. However, since the domain is not DMARC-compliant, there is still a chance the email will be accepted (based on the tenant configuration or implicit email authentication). Fortunately, the receiving tenant has Spoof Intelligence enabled and the email has been detected as `SPOOF` by Spoof Intelligence. As a result, the message is quarantined because of the configured applied action. 

[!IMAGE](/images/exo-reject-direct-send/exo-reject-direct-send-1.png)

4. However, this tenant does not have **request to release from quarantine** restrictions enabled for the applied action. This means the user can release the message themselves and open the malicious attachment.

[!IMAGE](/images/exo-reject-direct-send/exo-reject-direct-send-2.png)

### Reject DirectSend
For this test, I used the same demo tenant, but this time with RejectDirectSend set to `$true`. The attacker followed the same steps, but now encountered the following error when attempting to send a message:

[!IMAGE](/images/exo-reject-direct-send/exo-reject-direct-send-3.png)

## Summary
You may be hesitant to enable the Reject Direct Send feature due to a lack of visibility into whether Direct Send is currently used in your tenant. To help with this, Microsoft is working on a Direct Send traffic report, which will allow admins to identify any existing Direct Send usage and assess the potential impact of enabling the feature. Since Direct Send requires the outbound IP of the sender or range to be included in the SPF record, admins should already be in the habit of documenting each sender in their SPF configuration. If you are confident that Direct Send is not being used in your environment, you can safely enable the Reject Direct Send feature already.

## References
- [Microsoft Learn - Direct Send Setup](https://learn.microsoft.com/en-us/exchange/mail-flow-best-practices/how-to-set-up-a-multifunction-device-or-application-to-send-email-using-microsoft-365-or-office-365#direct-send-send-mail-directly-from-your-device-or-application-to-microsoft-365-or-office-365)
- [Microsoft Learn - Inbound email authentication in Microsoft 365](https://learn.microsoft.com/en-us/defender-office-365/email-authentication-about#inbound-email-authentication-for-mail-sent-to-microsoft-365)
- [Microsoft Tech Community - More control over Direct Send](https://techcommunity.microsoft.com/blog/exchange/introducing-more-control-over-direct-send-in-exchange-online/4408790)



