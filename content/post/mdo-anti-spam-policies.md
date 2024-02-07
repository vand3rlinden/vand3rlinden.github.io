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

- ***Allowed and blocked senders or domains**, such as ```someone@domain.com``` or ```domain.com```. 

_*To establish a list of blocked/safe senders in EOP, please follow the instructions provided in the two links below:_
- _[Create blocked sender lists in EOP](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/create-block-sender-lists-in-office-365)_
- _[Create safe sender lists in EOP](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/create-safe-sender-lists-in-office-365)_

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
- Set the bulk email threshold to 6 as a minimum, a higher bulk email threshold means more bulk email will be delivered.
- Mark high-risk countries as spam.
- Mark languages you don’t communicate with as spam.
- Set all your spam actions to quarantine and use quarantine policies.
- Do not set up any allowed senders or domains.
    - Allowed senders or allowed domains that are listed in anti-spam policies, creates a [high risk](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/create-safe-sender-lists-in-office-365?view=o365-worldwide#use-allowed-sender-lists-or-allowed-domain-lists) because it is possible to bypass all spam, spoof, phishing protection (except high confidence phishing), and sender authentication (SPF, DKIM, DMARC). You should consider using the [Tenant Allow/Block List](https://security.microsoft.com/tenantAllowBlockList) for [creating a safe sender lists](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/create-safe-sender-lists-in-office-365).
- Turn off ASF (Advanced Spam Filter), such as 'SPF record: hard fail'. This setting does not need to be enabled because legitimate email can be marked as high confidence phishing (HSPM) in situations where email is unable to pass DMARC with SPF and passed with DKIM for the P2 sender domain.
    - Enabling one or more of the [ASF settings](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/anti-spam-policies-asf-settings-about) is an aggressive approach to spam filtering that can often cause false positives. The effectiveness of these settings in reducing spam has severely declined over the years.

### Connection filter policy
- IP allow list configuration:
    - Adding allowed IP addresses is a more convenient option than allowing senders or domains. But you should always be careful, if you add IPs to the list for any reason (do not use IP ranges), check these IPs regularly to know the exact origin of each IP. Although IP spoofing is less common, it does happen.

- IP block list configuration:
    - A spammer or phisher will always send over multiple IP ranges. In most use cases, the email subject pattern is more likely to match when you receive an email attack. You can temporarily block this with a mail flow rule, for example.

- Safe list configuration:
    - You should not want to enable this, I think you want to decide for yourself which party you want to bypass for spam filtering.

### Anti-spam outbound policy
- Disable automatic forwarding:
    - Disabling this option will disable any auto-forwarding in your environment. Please review the comparison chart below to determine if this option meets your organization's needs.

- Configure message limits based on your organization's needs and set the limit action to restrict users from sending email. Users can be released from the [restricted entities](https://security.microsoft.com/restrictedentities) once there is no indication of a compromised user.

### Anti-spam outbound policy (Custom, Allow Forward)
There are situations where you might want to allow automatic forwarding for mail accounts that need to have forwarding enabled, such as forwarding mail to internal team channels _(emails.teams.ms addresses)_.

In this case, you can:
1. Create a new custom outbound policy, such as: _‘Anti-spam outbound policy (Allow Forward)’_

2. Add the desired mail account(s) to the policy

3. Set Automatic forwarding to: _‘On — Forwarding is enabled’_.

### Comparison chart of methods for blocking automatic forwarding in Microsoft 365
| Method                          | Pros                                                                                              | Cons                                                                                                                                                   |
|-                                |-                                                                                                  |-                                                                                                                                                       |
| Remote domains (EXO)            | Applies to all types of forwarding that a user can set up.                                        | 1. The user will not be notified that their forwarded message will be dropped. 2. You cannot set a specific address to forward, only an entire domain. |
| Mail flow rules (EXO)           | Allows more granularity on conditions and actions, such as setting a specific address to forward. | Does not block the OWA direct forwarding method.                                                                                                              |
| Outbound Anti-spam policy (MDO) | Allows you to turn off automatic forwarding completely.                                           | You cannot specify the address or domain; if an allowed user sets up a client forwarding rule, they can forward to any domain.                         |

If you only want to allow users to forward to a few specific domains, the best choice is to disable automatic forwarding with remote domains in Exchange Online. Mail flow rules should never be an option because direct forwards (OWA forwarding method) can still be used to any domain, only automatic forwards created with inbox rules are blocked.

### Reference
- [Anti-spam protection](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/anti-spam-protection-about)
- [Exchange Online Sending limits](https://learn.microsoft.com/en-us/office365/servicedescriptions/exchange-online-service-description/exchange-online-limits#sending-limits)