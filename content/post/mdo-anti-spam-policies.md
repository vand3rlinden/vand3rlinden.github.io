---
title: 'Microsoft Defender for Office 365: Anti-spam policies'
date: 2023-12-18T15:24:50+01:00
draft: false
categories: ["Microsoft Defender for Office 365"]
cover: 
  image: /images/mdo-anti-spam-policies/mdo-anti-spam-policies-front.png
---

> _All inbound e-mail is automatically protected from spam by Exchange Online Protection (EOP) for Microsoft 365 organizations with mailboxes in Exchange Online. EOP uses anti-spam policies as part of your organization's overall spam defense._

## What you can manage with Anti-spam policies
Anti-spam policies provide you with control over both inbound and outbound email in Exchange Online. Within the Microsoft Security Portal, you can access the [Anti-Spam Policy section](https://security.microsoft.com/antispam), where three default policies are available for editing. Additionally, it is possible to create custom policies, and further details on this will be discussed later in this post.

### Anti-spam inbound policy (Default)
In this inbound policy you can manage:
- **Bulk email threshold and spam properties**, such as rejecting email from high-risk countries.

- **Spam actions**, to set actions (to junk, quarantine, delete and more) when spam is detected. Based on the verdict, you can choose what to do with it, such as quarantine the message, which you can manage with quarantine policies.

- **Allowed and blocked senders or domains**, such as ```someone@domain.com``` or ```domain.com```. Senders or domains that you list in the Allowed section* will be skipped for spam filtering, and outbound authentication protocols (SPF, DKIM, and DMARC) will be ignored. 

*It is strongly advised against utilizing this feature, as it makes it exceptionally easy to send emails from any sender or domain listed here. To establish a list of blocked/safe senders in EOP, please follow the instructions provided in the two links below:
- [Create blocked sender lists in EOP](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/create-block-sender-lists-in-office-365)
- [Create safe sender lists in EOP](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/create-safe-sender-lists-in-office-365)

### Connection filter policy (Default)
In this filter policy you can manage:
- **IP allow list:** IP addresses on this list will be skipped for spam filtering.

- **IP block list:** IP addresses from this list will be blocked.

- **Safe list:** A dynamic allow list in the Microsoft datacenter that does not require customer configuration.

### Anti-spam outbound policy (Default)
In this outbound policy you can manage:
- **Internal/external message limit (per hour)**, set an hourly limit for your users (the value cannot exceed the daily limit).

- **Daily message limit**, the [default limit](https://learn.microsoft.com/en-us/office365/servicedescriptions/exchange-online-service-description/exchange-online-limits#sending-limits) is ```10,000``` emails per day, this limit can be changed between ```0``` and ```10,000```.

- **Limit action**, to restrict users from sending email who reach the message limit.

- **Disable automatic forwarding**, one-third of the available options for disabling automatic forwarding in EOP; more details will be provided later in this post.

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
    - Disabling this option disables any automatic forwarding in your environment. There are other ways to block automatic forwarding, each with pros and cons. You can block automatic forwarding with:
        - **Mail flow rules (EXO):** does not block the OWA forwarding method.
        - **Remote Domains (EXO):** the user is not notified _(no NDR)_ that their forwarded message has been dropped.

- Configure message limits based on your organization's needs and set the limit action to restrict users from sending email. Users can be released from the [restricted entities](https://security.microsoft.com/restrictedentities) once there is no indication of a compromised user.

### Anti-spam outbound policy (Custom)
There are situations where you might want to allow automatic forwarding for mail accounts that need to have forwarding enabled, such as forwarding mail to internal team channels _(emails.teams.ms addresses)_.

In this case, you can create a new custom outbound policy, such as: _‘Anti-spam outbound policy (Allow Forward)’_

Add the desired mail account(s) to the policy and set Automatic forwarding to: _‘On — Forwarding is enabled’_.

> ***Note:*** Mail accounts can forward to any external domain once you add a mail account to the _'Anti-Spam Outbound Policy (Allow Forwarding)'_ policy. If you want to allow only certain domains, the best choice is to disable automatic forwarding with remote domains in Exchange Online. Mail flow rules should never be an option because direct forwards (OWA forwarding method) can still be used to any domain, only automatic forwards created with inbox rules are blocked.

### Reference
- [Anti-spam protection](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/anti-spam-protection-about)
- [Exchange Online Sending limits](https://learn.microsoft.com/en-us/office365/servicedescriptions/exchange-online-service-description/exchange-online-limits#sending-limits)