---
title: 'Microsoft Defender for Office 365: Anti-spam policies'
date: 2023-12-18T15:24:50+01:00
draft: false
categories: ["Microsoft Defender for Office 365"]
cover: 
  image: /images/mdo-anti-spam-policies/mdo-anti-spam-policies-front.png
---

> _In Microsoft 365 organizations with mailboxes in Exchange Online, all inbound e-mail messages are automatically protected from spam by Exchange Online Protection (EOP). EOP uses anti-spam policies as part of your organization's overall spam defense._

## What you can manage with Anti-spam policies
Anti-spam policies give you control over inbound and outbound e-mail in Exchange Online. There are three out-of-the-box default policies that you can edit in the [Anti-Spam Policy section](https://security.microsoft.com/antispam) of the Microsoft Security Portal, where you can also create custom policies (more about this later).

### Anti-spam inbound policy (Default)
In this inbound policy you can manage:
- Bulk email threshold and spam properties, such as rejecting email from high-risk countries.
- Spam actions, to set actions (to junk, quarantine, delete and more) when spam is detected. Based on the verdict, you can choose what to do with it, such as quarantine the message, which you can manage with quarantine policies.
- Allowed and blocked senders or domains, such as ```someone@domain.com``` or ```domain.com```. Senders or domains that you list in the Allowed section will be skipped for spam filtering. You should never want to use this, as it is very easy to send email from any sender or domain you list here*. Outbound authentication protocols (SPF, DKIM and DMARC) are ignored.

*To create a blocked/safe senders list in EOP, follow the instructions in the two links below:
- [Create blocked sender lists in EOP](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/create-block-sender-lists-in-office-365)
- [Create safe sender lists in EOP](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/create-safe-sender-lists-in-office-365)

### Connection filter policy (Default)
In this filter policy you can manage:
- IP allow list: IP addresses on this list will be skipped for spam filtering.
- IP block list: IP addresses from this list will be blocked.
- Safe list: A dynamic allow list in the Microsoft datacenter that does not require customer configuration.

### Anti-spam outbound policy (Default)
In this outbound policy you can manage:
- Message limits, the default limit of 10,000 emails per day can be divided into how many of the 10,000 emails can be sent internally and externally, and to restrict users who have reached the limit you set.
- Disable automatic forwarding rules (1/3 of the available options to disable automatic forwarding, more later).

## Best practices

### Anti-spam inbound policy
- Mark high-risk countries as spam.
- Mark languages you don’t communicate with as spam.
- Set all your spam actions to quarantine and use quarantine policies.
- Do not set up any allowed senders or domains as written in the beginning of this post.

### Connection filter policy
- IP allow list configuration:
    - Adding allowed IP addresses is a more convenient option than allowing senders or domains. But you should always be careful, if you add IPs to the list for any reason (do not use IP ranges), check these IPs regularly to know the exact origin of each IP. Although IP spoofing is less common, it does happen.

- IP block list configuration:
    - A spammer or phisher will always send over multiple IP ranges. In most use cases, the email subject pattern is more likely to match when you receive an email attack. You can temporarily block this with a mail flow rule, for example.

- Safe list configuration:
    - You should not want to enable this, I think you want to decide for yourself which party you want to bypass for spam filtering.

### Anti-spam outbound policy
- Disable automatic forwarding:
    - Disabling this option disables any automatic forwarding in your environment. Such as for inbox rules or direct forwards. When a user sets up automatic forwarding, they receive an NDR that says _‘Your organization does not allow external forwarding’_ There are more ways for blocking automatic forward, each with pros and cons. You can block automatic forward with:
        - Mail flow rules (EXO): does not block the OWA forwarding method.
        - Remote Domains (EXO): the user is not notified that their forwarded message has been dropped.

- Configure message limits based on your organization’s needs:
    -  The default limit of 10,000 emails per day should be divided into how many of the 10,000 emails can be sent internally and externally by normal users, and to restrict those users who have reached the limit you set. By configuring this setting, you can prevent abnormal sending behavior if a user is compromised.

### Anti-spam outbound policy (Custom)
There are situations where you might want to allow automatic forwarding, such as for mail accounts that need to have forwarding enabled for Teams channels (emails.teams.ms addresses).

In this case, you can create a new custom outbound policy, such as: _‘Anti-spam outbound policy (Allow Forward)’_

Add the desired mail account(s) to the policy and set Automatic forwarding to _‘On — Forwarding is enabled’_.

Note that mail accounts can forward to any external domain. If you want to allow only certain domains, the best choice is to disable automatic forwarding to remote domains.

Mail flow rules should never be an option because direct forwards (OWA forwarding method) can still be used to any domain, only automatic forwards to inbox rules will be blocked.

### Reference
- [Anti-spam protection](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/anti-spam-protection-about)
- [Exchange Online Sending limits](https://learn.microsoft.com/en-us/office365/servicedescriptions/exchange-online-service-description/exchange-online-limits#sending-limits)