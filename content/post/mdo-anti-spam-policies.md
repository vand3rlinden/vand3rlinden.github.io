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

- **Allowed and blocked senders or domains**, such as `someone@domain.com` or `domain.com` ([not recommended](https://vand3rlinden.com/post/mdo-anti-spam-policies/#best-practices)).
  - To create a list of blocked/safe senders in EOP, follow the instructions in the two links below:
    - [Create blocked sender lists in EOP](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/create-block-sender-lists-in-office-365)
    - [Create safe sender lists in EOP](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/create-safe-sender-lists-in-office-365)

### Connection filter policy (Default)
In this filter policy you can manage:
- **IP allow list:** IP addresses on this list will be skipped for spam filtering.

- **IP block list:** IP addresses from this list will be blocked.

- **Safe list:** The safe list in the connection filter policy is a dynamic allow list that requires no customer setup. Microsoft automatically identifies trusted email sources from third-party subscriptions. You can enable or disable the safe list but cannot configure its servers. Incoming messages from these servers bypass spam filtering.

### Anti-spam outbound policy (Default)
In this outbound policy you can manage:
- **Internal/external message limit (per hour)**, set an hourly limit for your users (the value cannot exceed the daily limit).

- **Daily message limit**, the [default limit](https://learn.microsoft.com/en-us/office365/servicedescriptions/exchange-online-service-description/exchange-online-limits#sending-limits) is `10,000` emails per day, this limit can be changed between `0` and `10,000`.

> Beginning in October 2025, Exchange Online will begin enforcing an [External Recipient Rate Limit (ERR)](https://techcommunity.microsoft.com/t5/exchange-team-blog/exchange-online-to-introduce-external-recipient-rate-limit/ba-p/4114733) of 2,000 recipients in 24 hours. If you have a cloud-hosted mailbox that needs to exceed the ERR limit, you can move to [Azure Communication Services for Email](https://learn.microsoft.com/en-us/azure/communication-services/concepts/email/email-overview), which is designed specifically for high-volume email sent to recipients outside your tenant. For interal messages you could use [High Volume Email for Microsoft 365](https://techcommunity.microsoft.com/blog/exchange/high-volume-email-continued-support-for-basic-authentication--other-important-up/4411197) ([HVE on Microsoft Learn](https://learn.microsoft.com/en-us/exchange/mail-flow-best-practices/high-volume-mails-m365) - Public Preview).

- **Limit action**, to restrict users from sending email who reach the message limit.

- **Disable automatic forwarding**, one-third of the available options for disabling automatic forwarding in EOP; more details will be provided later in this post.

## Best practices
> The best practices are based on the Strict recommendations settings of the Configuration Analyzer for the [Inbound anti-spam policy](https://learn.microsoft.com/en-us/defender-office-365/recommended-settings-for-eop-and-office365?view=o365-worldwide#eop-anti-spam-policy-settings) and [Outbound anti-spam policy](https://learn.microsoft.com/en-us/defender-office-365/recommended-settings-for-eop-and-office365?view=o365-worldwide#eop-outbound-spam-policy-settings)

### Anti-spam inbound policy
- Set the bulk email threshold to 5 as a minimum. 
  - A higher bulk email threshold means more bulk email will be delivered.
  - You can also review the [Bulk Senders Insight page](https://security.microsoft.com/senderinsights) to see how much mail is classified as bulk at each BCL level (1 to 9) over the past 60 days. On this same page, you can simulate changes to the bulk email threshold and view the impact on the number of messages delivered versus those identified as bulk in your inbound anti-spam policy. More information about [Bulk senders insight in Exchange Online Protection](https://learn.microsoft.com/en-us/defender-office-365/anti-spam-bulk-senders-insight).

- It is recommended to disable the [ASF (Advanced Spam Filter) settings](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/anti-spam-policies-asf-settings-about), as they often cause false positives. For example, certain message elements may cause the message to be flagged as spam or increase its spam score. For instance, with the ASF setting: [SPF record: hard fail](https://learn.microsoft.com/en-us/defender-office-365/anti-phishing-protection-spoofing-faq#do-i-still-need-to-enable-the-advanced-spam-filter-setting--spf-record--hard-fail----markasspamspfrecordhardfail---if-i-enable-anti-spoofing-), inbound messages sent from an IP address not listed in the sender’s SPF record are automatically marked as high confidence spam (HSPM). This occurs even if the message is DMARC compliant (dmarc=pass) due to successful DKIM validation for the P2 sender domain.
 
- Set all your spam actions to 'Quarantine message'.
    - My preference is to ***only*** select ***request to release*** [quarantine policies](https://vand3rlinden.com/post/mdo-quarantine-policies/) for the actions ***Phishing message action*** and ***High confidence phishing message action*** (also, quarantine release permissions will be ignored for high-confidence phishing messages).

- Set Intra-Organizational messages to the default setting. This setting control spam filtering, and the corresponding actions will be applied to internal messages (messages sent between users within the organization).
    - The default value is the same as selecting High confidence phishing messages.

- Leave 'Retain spam in quarantine for this many days' at the default 30 days.

- Enable Spam Safety Tips, a color-coded banner in Outlook that warns recipients of potentially harmful messages.

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
- Configure outbound message limits:
  - Recommended Strict:
    ```
    Restrict sending to external recipients (per hour)
    400
    Restrict sending to internal recipients (per hour)
    800
    Maximum recipient limit per day
    800
    ```

- Set the restriction placed on users who reach the message limit to **restrict users from sending email**, users can be released from the [restricted entities](https://security.microsoft.com/restrictedentities) once there is no indication of a compromised user.

- Disable automatic forwarding:
    - Disabling this option (`Off - Forwarding is disabled (Off)`) will disable any auto-forwarding in your environment. Please review the comparison chart below to determine if this option meets your organization's needs.
      - **NOTE**: `Automatic - System-controlled (Automatic)` is equivalent to `Off - Forwarding is disabled (Off)`, however it is recommended to set this value to `Off - Forwarding is disabled (Off)`.

### Anti-spam outbound policy (Custom, Allow Forward)
There are situations where you might want to allow automatic forwarding for mail accounts that need to have forwarding enabled, such as forwarding mail to internal team channels _(emails.teams.ms addresses)_.

In this case, you can:
1. Create a new custom outbound policy, such as: _‘Anti-spam outbound policy (Allow Forward)’_

2. Add the desired mail account(s) to the policy

3. Set Automatic forwarding to: _‘On — Forwarding is enabled’_.

### Comparison chart of methods for blocking automatic forwarding in Microsoft 365
Exchange Online provides several methods for blocking automatic forwarding. These are Remote domains, Transport Rules, and Anti-spam outbound policies. Each method has its pros and cons:

| Method                          | Pros                                                                                             | Cons                                                                                                                                                       | Exclusions                  |
|-                                |-                                                                                                 |-                                                                                                                                                           |-                            |
| Remote domains (EXO)            | Applies to all types of forwarding that a user can set up                                        | The user will not be notified that their forwarded message will be dropped                                                                                 | Per domain                  |
| Mail flow rules (EXO)           | Allows more granularity on conditions and actions, such as setting a specific address to forward | Does not block the OWA forwarding setting (ForwardingSmtpAddress)                                                                                          | Per domain and address      |
| Outbound Anti-spam policy (MDO) | Allows you to turn off automatic forwarding completely                                           | You cannot specify the address or domain as an exclusion; if an allowed user sets up a client forwarding rule, they can forward to any address and domain  | Per user                    |

If you only want to allow users to forward to a few specific domains, the best choice is to disable automatic forwarding with remote domains in Exchange Online. Mail flow rules should never be an option because direct forwards (OWA forwarding method) can still be used to any domain, only automatic forwards created with inbox rules are blocked.

### Reference
- [Anti-spam protection](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/anti-spam-protection-about)
  - [Configure anti-spam inbound policy](https://learn.microsoft.com/en-us/defender-office-365/anti-spam-policies-configure)
  - [Configure connection filter policy](https://learn.microsoft.com/en-us/defender-office-365/connection-filter-policies-configure)
  - [Configure anti-spam outbound policy](https://learn.microsoft.com/en-us/defender-office-365/outbound-spam-policies-configure)
- [Exchange Online Sending limits](https://learn.microsoft.com/en-us/office365/servicedescriptions/exchange-online-service-description/exchange-online-limits#sending-limits)
- [Zero-hour auto purge (ZAP)](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/zero-hour-auto-purge)
- [Recommended anti-spam policy settings - Inbound anti-spam policy](https://learn.microsoft.com/en-us/defender-office-365/recommended-settings-for-eop-and-office365?view=o365-worldwide#anti-spam-policy-settings)
- [Recommended anti-spam policy ASF settings - Inbound anti-spam policy](https://learn.microsoft.com/en-us/defender-office-365/recommended-settings-for-eop-and-office365?view=o365-worldwide#asf-settings-in-anti-spam-policies)
- [Recommended anti-spam policy settings - Outbound anti-spam policy](https://learn.microsoft.com/en-us/defender-office-365/recommended-settings-for-eop-and-office365?view=o365-worldwide#outbound-spam-policy-settings)
- [Azure Communication Services for Email - Solution for high volumes of external email](https://learn.microsoft.com/en-us/azure/communication-services/concepts/email/email-overview)
- [Public Preview: High Volume Email for Microsoft 365 - Solution for high volumes of internal email](https://learn.microsoft.com/en-us/exchange/mail-flow-best-practices/high-volume-mails-m365)
- [Bulk senders insight in Exchange Online Protection](https://learn.microsoft.com/en-us/defender-office-365/anti-spam-bulk-senders-insight)
- [Control automatic external email forwarding in Microsoft 365](https://learn.microsoft.com/en-us/defender-office-365/outbound-spam-policies-external-email-forwarding)