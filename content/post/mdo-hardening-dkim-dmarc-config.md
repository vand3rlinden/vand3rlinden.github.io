---
title: 'Microsoft Defender for Office 365: Hardening DKIM and DMARC configuration'
date: 2024-04-21T15:31:27+02:00
draft: false
categories: ["Microsoft Defender for Office 365"]
cover: 
  image: /images/mdo-hardening-dkim-dmarc-config/mdo-hardening-dkim-dmarc-config-front.png
---

> Improve email security in Microsoft 365: Fine-tuning DKIM and setup DMARC for the MOERA domain.

## Fine-tune DKIM by frequently rotating the DKIM keys
After [setting up DKIM](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/email-authentication-dkim-configure) in Microsoft Defender for Office 365, it is also important to set up frequent rotation of these DKIM keys to prevent adversaries from intercepting and decrypting your cryptographic keys. Key rotation helps to minimize the risk of compromising the private keys. In Microsoft 365, you can rotate DKIM keys for your custom domain to increase security. After four days (96 hours), the new DKIM key begins signing outbound messages for the custom domain. However, you may want to automate this task by using Azure Automation. To do this, you can set up a runbook with the following script.
```
#Connect EXO
Connect-ExchangeOnline -ManagedIdentity -Organization yourorg.onmicrosoft.com

#Rotate the DKIM keys for all enabled domains
$Domains = (Get-DkimSigningConfig | Where-Object {$_.Enabled -eq 'True'}).domain

foreach ($domain in $domains){
    Rotate-DkimSigningConfig -KeySize 2048 -Identity $domain
}
```
To configure the runbook, you can access my [MDO Azure Automation](https://github.com/vand3rlinden/MDO-Azure-Automation) repository on my Github.

## Setup DMARC for the MOERA domain.
Your `onmicrosoft.com` domain, also known as the Microsoft Online Email Routing Address (MOERA) domain, does not have a DMARC policy set up by default. This means that this domain is vulnerable to abuse. As for spoofed inbound messages from your `onmicrosoft.com`, they will end up in the Junk folder because the detection technology is `Spoof intra-org`, and Spoof Intelligence will not be triggered in the anti-phishing policy.

To prevent email spoofing and phishing using this domain, you need to set up a DMARC reject policy on the MOERA domain, as you do on all of your send and [non-send domains](https://vand3rlinden.com/post/spf-dkim-dmarc-explanation/#protect-all-non-sending-domains). Setting DMARC for the MOERA is most often forgotten because you do not own the MOERA domain, but Microsoft has made it possible to manage the public DNS of your MOERA domain in the Microsoft 365 Admin Center.

1. In the [Microsoft 365 admin center](https://admin.microsoft.com), select Show all > Settings > Domains
2. On the Domains page, open the `*.onmicrosoft.com` domain and click on DNS Records
4. Click on Add record and set the following values
- Type: `TXT`
- TXT name: `_dmarc`
- TXT value: `v=DMARC1; p=reject;`


## To summerize 
DKIM rotation ensures that even if a key is compromised, it will become obsolete after a period of time, limiting the window of vulnerability. Setting up DMARC for the MOERA domain is an essential practice to protect against email fraud on this domain.