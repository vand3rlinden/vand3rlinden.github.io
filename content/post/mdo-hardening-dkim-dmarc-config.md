---
title: 'Microsoft Defender for Office 365: Hardening DKIM and DMARC configuration'
date: 2024-04-21T15:31:27+02:00
draft: false
categories: ["Microsoft Defender for Office 365"]
cover: 
  image: /images/mdo-hardening-dkim-dmarc-config/mdo-hardening-dkim-dmarc-config-front.png
---

> Improve email security in Microsoft 365: Fine-tuning DKIM, configuring DMARC for the MOERA domain, and blocking inbound DMARC failures from reaching user inboxes.

## Fine-tune DKIM by frequently rotating the DKIM keys
After [setting up DKIM](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/email-authentication-dkim-configure) in Microsoft Defender for Office 365, it is also important to set up frequent rotation of these DKIM keys to prevent adversaries from intercepting and decrypting your cryptographic keys. Key rotation helps to minimize the risk of compromising the private keys. In Microsoft 365, you can rotate the DKIM keys for your domains to increase security. The recurrence must be every 3 months because rotating the DKIM keys every 3 months ensures a complete rotation of both selectors every 6 months. You can rotate the DKIM keys manually using the [Defender portal](https://learn.microsoft.com/en-us/defender-office-365/email-authentication-dkim-configure#use-the-defender-portal-to-rotate-dkim-keys-for-a-custom-domain) or [Exchange Online PowerShell](https://learn.microsoft.com/en-us/defender-office-365/email-authentication-dkim-configure#use-exchange-online-powershell-to-rotate-the-dkim-keys-for-a-domain-and-change-the-bit-depth), but it is easy to forget if you do it manually. So you should delegate this to Azure Automation by using the runbook below:

```
#Connect EXO
Connect-ExchangeOnline -ManagedIdentity -Organization yourorg.onmicrosoft.com

#Rotate the DKIM keys for all enabled domains
$Domains = (Get-DkimSigningConfig | Where-Object {$_.Enabled -eq 'True'}).domain

foreach ($domain in $domains){
    Rotate-DkimSigningConfig -KeySize 2048 -Identity $domain
}
```

To configure the runbook, you can access my [MDO Azure Automation](https://github.com/vand3rlinden/MDO-Azure-Automation) repository on Github.

## Setup DMARC for the MOERA domain
Your `onmicrosoft.com` domain, also known as the Microsoft Online Email Routing Address (MOERA) domain, does not have a DMARC policy set up by default. This means that this domain is vulnerable to abuse and can be used to spoof your `onmicrosoft.com` domain for ***outbound messages***. 

Example of a spoofed ***outbound message*** from an `onmicrosoft.com` that doesn't have a DMARC reject policy:
![IMAGE](/images/mdo-hardening-dkim-dmarc-config/mdo-hardening-dkim-dmarc-config1.png)

> **NOTE:** Spoofed ***inbound messages*** from  your `onmicrosoft.com` domain will end up in the Junk Folder, because the detection technology is `Spoof intra-org`. Spoof Intelligence will not trigger your anti-phishing policy for this detection.

To prevent email spoofing and phishing using this domain, you need to set up a DMARC reject policy on the MOERA domain, as you do on all of your send and [non-send domains](https://vand3rlinden.com/post/spf-dkim-dmarc-explanation/#protect-all-non-sending-domains). Setting DMARC for the MOERA is most often forgotten because you do not own the MOERA domain, but Microsoft has made it possible to manage the public DNS of your MOERA domain in the Microsoft 365 Admin Center.

1. In the [Microsoft 365 admin center](https://admin.microsoft.com), select Show all > Settings > Domains
2. On the Domains page, open your `onmicrosoft.com` domain and click on DNS Records
3. Click on Add record and set the following values
- Type: `TXT`
- TXT name: `_dmarc`
- TXT value: `v=DMARC1; p=reject;`

If you are not using this domain for outbound messaging, you can treat your `onmicrosoft.com` domain as a non-sending domain, and the DMARC Aggregate (`rua`) and DMARC Forensic (`ruf`) reports are not required to be listed in the DMARC `TXT` record.

## Enforce inbound DMARC rejection policy to reject
Microsoft 365 overrides the DMARC fail action from `reject` to `oreject` (override reject) because, it uses [implicit email authentication](https://learn.microsoft.com/en-us/defender-office-365/email-authentication-about#inbound-email-authentication-for-mail-sent-to-microsoft-365) to evaluate inbound emails in Exchange Online Protection (EOP). This method extends traditional SPF, DKIM, and DMARC checks by incorporating signals from various sources, such as:

- Sender reputation
- Sender history
- Recipient history
- Behavioral analysis
- Other advanced techniques

The results of these implicit checks are consolidated into a single value called [composite authentication](https://learn.microsoft.com/en-us/defender-office-365/email-authentication-about#composite-authentication) (`compauth`), which is set in the `Authentication-Results` header of the email.

If an email bypasses composite authentication (`compauth=none`) because other implicit signals pass successfully, a DMARC fail—even with a `reject` policy on the sender’s domain—can still be delivered.

This configuration exists to prevent the rejection of some legitimate emails that may fail DMARC checks. By leveraging implicit email authentication, messages that fail traditional authentication methods may still be allowed into Microsoft 365 if the overall evaluation deems them safe. False negatives from the advanced implicit authentication checks can be reported to Microsoft as [questionable emails](https://learn.microsoft.com/en-us/defender-office-365/submissions-admin#report-questionable-email-to-microsoft) to improve detection.

Example `Authentication-Results` header with DMARC fail action `oreject` and composite authentication `none` that arrived in the user's inbox and was not detected as Spoof by Spoof Intelligence:
```
spf=fail (sender IP is 11.222.33.444) smtp.mailfrom=contoso.com; dkim=none (message not signed) header.d=none;dmarc=fail action=oreject header.from=contoso.com;compauth=none reason=451
```

If you want more granular control over DMARC failures, from a domain whose DMARC `TXT` record has a policy of `p=reject`, you can configure a mail flow rule in Exchange Online. This rule would reject emails with the value `dmarc=fail action=oreject` in the `Authentication-Results` [header field](https://learn.microsoft.com/en-us/defender-office-365/message-headers-eop-mdo#authentication-results-message-header-fields) instead of delivering them.

Example of the mail flow rule in Exchange Online:
![IMAGE](/images/mdo-hardening-dkim-dmarc-config/mdo-hardening-dkim-dmarc-config2.png)

Example of the NDR:
![IMAGE](/images/mdo-hardening-dkim-dmarc-config/mdo-hardening-dkim-dmarc-config3.png)

> CAUTION: The outlined approach may lead to false positives due to EOP’s implicit authentication techniques. However, in **my personal opinion**, DMARC failures should always be rejected before reaching a user’s inbox. It’s the sender’s responsibility to ensure proper outbound authentication. Non-compliant DMARC emails should be treated as suspicious, even if they pass advanced checks. This method also catches emails that have not been flagged as Spoof by Spoof Intelligence (that has the [honor DMARC policy](https://vand3rlinden.com/post/mdo-anti-phishing-policies/#configure-anti-phishing-policies-and-best-practices) configured to reject) despite failed authentication, as they’ll now be handled by the proposed mail flow rule to reject.

## To summerize 
DKIM rotation ensures that even if a key is compromised, it will become obsolete after a period of time, limiting the window of vulnerability. Setting up DMARC for the MOERA domain is critical to protect against email fraud targeting this domain, and by preventing inbound DMARC failures with the oreject override action from reaching user inboxes, you will harden your DKIM and DMARC configuration in Microsoft 365.

## Reference
- [Rotate DKIM keys](https://learn.microsoft.com/en-us/defender-office-365/email-authentication-dkim-configure#rotate-dkim-keys)
- [Use the Microsoft 365 admin center to add DMARC TXT records for *.onmicrosoft.com domains in Microsoft 365](https://learn.microsoft.com/en-us/defender-office-365/email-authentication-dmarc-configure?view=o365-worldwide#use-the-microsoft-365-admin-center-to-add-dmarc-txt-records-for-onmicrosoftcom-domains-in-microsoft-365)