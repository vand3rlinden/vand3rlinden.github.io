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

- ***Allowed and blocked senders or domains**, such as `someone@domain.com` or `domain.com`. 

_*To establish a list of blocked/safe senders in EOP, please follow the instructions provided in the two links below:_
- _[Create blocked sender lists in EOP](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/create-block-sender-lists-in-office-365)_
- _[Create safe sender lists in EOP](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/create-safe-sender-lists-in-office-365)_

### Connection filter policy (Default)
In this filter policy you can manage:
- **IP allow list:** IP addresses on this list will be skipped for spam filtering.

- **IP block list:** IP addresses from this list will be blocked.

- **Safe list:** The safe list in the connection filter policy is a dynamic allow list that requires no customer setup. Microsoft automatically identifies trusted email sources from third-party subscriptions. You can enable or disable the safe list but cannot configure its servers. Incoming messages from these servers bypass spam filtering.

### Anti-spam outbound policy (Default)
In this outbound policy you can manage:
- **Internal/external message limit (per hour)**, set an hourly limit for your users (the value cannot exceed the daily limit).

- **Daily message limit**, the [default limit](https://learn.microsoft.com/en-us/office365/servicedescriptions/exchange-online-service-description/exchange-online-limits#sending-limits) is `10,000` emails per day, this limit can be changed between `0` and `10,000`.

> Beginning in January 2025, Exchange Online will begin enforcing an [External Recipient Rate Limit (ERR)](https://techcommunity.microsoft.com/t5/exchange-team-blog/exchange-online-to-introduce-external-recipient-rate-limit/ba-p/4114733) of 2,000 recipients in 24 hours. The 2,000 ERR limit becomes a sub-limit within this `10,000` Recipient Rate limit. There is no change to the Recipient Rate limit, and both are rolling limits for 24-hour windows. You can send to up to `2,000` external recipients in a 24-hour period, and if you max out the external recipient rate limit, you can still send to up to `8,000` internal recipients in the same period. If you don't send to any external recipients in a 24-hour period, you can send to up to `10,000` internal recipients. If you have a cloud-hosted mailbox that needs to exceed the ERR limit, you can move to [Azure Communication Services for Email](https://learn.microsoft.com/en-us/azure/communication-services/concepts/email/email-overview), which is designed specifically for high-volume email sent to recipients outside your tenant. For interal messages you could use [High Volume Email for Microsoft 365](https://techcommunity.microsoft.com/t5/exchange-team-blog/public-preview-high-volume-email-for-microsoft-365/ba-p/4102271) (HVE).

- **Limit action**, to restrict users from sending email who reach the message limit.

- **Disable automatic forwarding**, one-third of the available options for disabling automatic forwarding in EOP; more details will be provided later in this post.

## Best practices

### Anti-spam inbound policy
- Set the bulk email threshold to 6 as a minimum, a higher bulk email threshold means more bulk email will be delivered.

- Mark high-risk countries as spam.

- Mark languages you don’t communicate with as spam.

- Turn off ASF (Advanced Spam Filter), such as 'SPF record: hard fail'. This setting does not need to be enabled because legitimate email can be marked as high confidence phishing (HSPM) in situations where email is unable to pass DMARC with SPF and passed with DKIM for the P2 sender domain.
    - Enabling one or more of the [ASF settings](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/anti-spam-policies-asf-settings-about) is an aggressive approach to spam filtering that can often cause false positives. The effectiveness of these settings in reducing spam has severely declined over the years.

- Set all your spam actions to 'Quarantine message', except for the ***Spam message action*** and ***Bulk message action***, this can be set on 'Move message to Junk Email folder'.
    - My preference is to select a [quarantine policies](https://vand3rlinden.com/post/mdo-quarantine-policies/) to ***request to release*** items for the actions ***Phishing message action*** and ***High confidence phishing message action*** (which is the default for HPHISH).

- Set intra-organizational messages to the default setting. This setting control spam filtering, and the corresponding actions will be applied to internal messages (messages sent between users within the organization).
    - The default value is the same as selecting High confidence phishing messages.

- Keep ZAP (Zero-hour auto purge) on for spam and phishing messages (enabled by default).
    - ZAP continuously monitors spam and malware updates seamlessly for users. It automatically detects and acts on messages in a user's mailbox, searching the last 48 hours of delivered email. Users aren't notified of detected and moved messages.

- Do not set up any allowed senders or domains.
    - Allowed senders or allowed domains that are listed in anti-spam policies, creates a [high risk](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/create-safe-sender-lists-in-office-365?view=o365-worldwide#use-allowed-sender-lists-or-allowed-domain-lists) because it is possible to bypass all spam, spoof, phishing protection (except high confidence phishing), and sender authentication (SPF, DKIM, DMARC). You should consider using the [Tenant Allow/Block List](https://security.microsoft.com/tenantAllowBlockList) for [creating a safe sender lists](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/create-safe-sender-lists-in-office-365).

### Connection filter policy
- IP allow list configuration:
    - Adding allowed IP addresses is a more convenient option than allowing senders or domains. But you should always be careful, if you add IPs to the list for any reason (do not use IP ranges), check these IPs regularly to know the exact origin of each IP. Although IP spoofing is less common, it does happen.

- IP block list configuration:
    - A spammer or phisher will always send over multiple IP ranges. In most use cases, the email subject pattern is more likely to match when you receive an email attack. You can temporarily block this with a mail flow rule, for example.

- Safe list configuration:
    - You should not enable this dynamic allow list, because incoming messages from this dynamic list will bypass spam filtering.

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
| Method                          | Pros                                                                                              | Cons                                                                                                                                                       | Exclusions                  |
|-                                |-                                                                                                  |-                                                                                                                                                           |-                            |
| Remote domains (EXO)            | Applies to all types of forwarding that a user can set up.                                        | The user will not be notified that their forwarded message will be dropped.                                                                                | Per domain                  |
| Mail flow rules (EXO)           | Allows more granularity on conditions and actions, such as setting a specific address to forward. | Does not block the OWA direct forwarding method                                                                                                            | Per domain and address      |
| Outbound Anti-spam policy (MDO) | Allows you to turn off automatic forwarding completely.                                           | You cannot specify the address or domain as an exclusion; if an allowed user sets up a client forwarding rule, they can forward to any address and domain  | Per user                    |

If you only want to allow users to forward to a few specific domains, the best choice is to disable automatic forwarding with remote domains in Exchange Online. Mail flow rules should never be an option because direct forwards (OWA forwarding method) can still be used to any domain, only automatic forwards created with inbox rules are blocked.

### Reference
- [Anti-spam protection](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/anti-spam-protection-about)
- [Configure connection filtering](https://learn.microsoft.com/en-us/defender-office-365/connection-filter-policies-configure)
- [Exchange Online Sending limits](https://learn.microsoft.com/en-us/office365/servicedescriptions/exchange-online-service-description/exchange-online-limits#sending-limits)
- [Zero-hour auto purge (ZAP)](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/zero-hour-auto-purge)
- [Recommended anti-spam policy settings - Inbound anti-spam policy](https://learn.microsoft.com/en-us/defender-office-365/recommended-settings-for-eop-and-office365?view=o365-worldwide#eop-anti-spam-policy-settings)
- [Recommended anti-spam policy ASF settings - Inbound anti-spam policy](https://learn.microsoft.com/en-us/defender-office-365/recommended-settings-for-eop-and-office365?view=o365-worldwide#asf-settings-in-anti-spam-policies)
- [Recommended anti-spam policy settings - Outbound anti-spam policy](https://learn.microsoft.com/en-us/defender-office-365/recommended-settings-for-eop-and-office365?view=o365-worldwide#eop-outbound-spam-policy-settings)
- [Control automatic external email forwarding in Microsoft 365](https://learn.microsoft.com/en-us/defender-office-365/outbound-spam-policies-external-email-forwarding)