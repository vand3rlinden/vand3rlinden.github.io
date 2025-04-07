---
title: 'Microsoft Defender for Office 365: Approach for Non-RFC compliant emails'
date: 2025-04-07T12:16:19+02:00
draft: false
categories: ["Microsoft Defender for Office 365"]
cover: 
  image: /images/mdo-rfc-compliant-emails/mdo-rfc-compliant-emails-front.png
---

> _In this post, I’ll explain what RFC compliant emails are, share examples, and discuss what this means for senders. Especially those who may unknowingly be using improperly formatted sender addresses._

## Microsoft 365’s enhanced detection
Recently, Microsoft has made a significant improvement in how it detects and handles non-RFC compliant emails. Microsoft has enhanced its detection mechanisms to better identify these types of messages, helping to protect users from potential risks.

As a result, users may now see safety tips or warnings in their Outlook clients when they receive emails that do not comply with RFC standards. These alerts are intended to raise awareness and encourage caution when engaging with such messages.

Over time, Microsoft plans to strengthen these measures, which may include blocking or rejecting emails that do not comply with RFC standards. 

## What are RFC compliant emails?
The **Internet Engineering Task Force (IETF)** defines standards through documents known as RFCs (Request for Comments).

One key standard is **RFC 5322 (Internet Message Format)**, which defines how sender address headers should be structured. **RFC 5322** is also referred to as the **P2 sender** or `From:` (`RFC5322.From`, `header.from`) header, which is the sender address visible to recipients.

Non-RFC compliant emails can be used to:

- **Evade detection**: Malicious actors use malformed sender addresses to bypass filters and deliver harmful content directly to inboxes.
- **Mislead recipients**: By manipulating sender addresses, attackers can make messages appear to come from trusted sources.
- **Facilitate phishing and fraud**: These tactics increase the chances of successful phishing attacks, which can result in data breaches and financial losses.

Enforcing RFC compliant emails is driven by several key reasons:

- **Prevent delivery issues**: Ensures emails are properly accepted by receiving systems, especially those with strict validation rules.
- **Improve security**: Reduces the risk of spoofed or malicious emails by enforcing proper formatting.
- **Enhance authentication alignment**: Supports better alignment with DKIM and DMARC for the **P2 sender domain**.
- **Protect recipients**: Helps prevent phishing and fraud by blocking malformed or deceptive emails.
- **Maintain trust**: Ensures consistent and professional email communication that meets industry standards.

## Examples of RFC compliant and non-RFC compliant email addresses
Properly RFC compliant P2 (`From:`) addresses are formatted as: `“Display Name" <email@address.com>`. This structure ensures that the displayname and email address are clearly defined and properly interpreted by email systems. Below are some examples of RFC compliant and non-RFC compliant P2 sender addresses:

| P2 Address                                                      | Comment                                                                                                    | Compliance        |
|----------                                                       |-------                                                                                                     |----------         |
| `shaggy.rogers@example.com`                                     | None                                                                                                       | RFC compliant     |
| `<shaggy.rogers@example.com>`                                   | None                                                                                                       | RFC compliant     |
| `Shaggy Rogers <shaggy.rogers@example.com>`                     | None                                                                                                       | RFC compliant     |
| `"Shaggy Rogers" <shaggy.rogers@example.com>`                   | Properly formatted with display name                                                                       | RFC compliant     |
| `"Rogers, Shaggy" <shaggy.rogers@example.com>`                  | Properly formatted with display name                                                                       | RFC compliant     |
| `"Shaggy Rogers <shaggy.rogers@example.com>"`                   | The entire `From:` value is in double quotes, which is not allowed           | Non-RFC compliant |
| `Rogers, Shaggy <shaggy.rogers@example.com>`                    | A display name that includes a comma must be enclosed in double quotes                                     | Non-RFC compliant |
| `shaggy.rogers@example.com <shaggy.rogers@another-example.com>` | Display name contains `@` and is not in quotes                                                             | Non-RFC compliant |
| `"Shaggy Rogers"<shaggy.rogers@example.com>`                    | Missing white space between the display name and email address                                             | Non-RFC compliant |
| `Shaggy Rogers shaggy.rogers@example.com`                       | When a display name is used, the email address must be enclosed in angle brackets (`<email@address.com>`)  | Non-RFC compliant |

## Impact of enhanced RFC compliant P2 address detection for Microsoft 365 email senders
If you are currently using non-RFC compliant P2 sender addresses, it’s important to update your email systems to follow RFC standards, as shown in the above table. Switching to RFC compliant formats helps ensure that your emails are successfully delivered to Microsoft 365 mailboxes without being flagged by detection mechanisms or triggering safety tips for recipients.

## SPF, DKIM, and DMARC vs. RFC compliant emails
Both help prevent email spoofing, but it’s important to note that SPF, DKIM, and DMARC focus on email authentication and domain protection, not message formatting. Even if your messages pass these checks, they can still be rejected or flagged if they don’t use RFC compliant P2 sender addresses.

Think of it this way: email authentication proves who you are, but RFC compliant P2 sender addresses shows you’re speaking the correct language.

## Microsoft Defender XDR Advanced Hunting detection
The following KQL query can be used to detect whether possible vendors or other external senders are sending inbound email messages that do not comply with the RFC standard for RFC compliant P2 (`From:`) addresses:

```kql
EmailEvents
| where Timestamp >= ago(30d)
| where not(SenderFromAddress matches regex @"^[a-zA-Z0-9.!#$%&'*+/=?^_`{|}~-]+@[a-zA-Z0-9-]+(\.[a-zA-Z0-9-]+)*$")
| project Timestamp,
          SenderMailFromAddress,
          SenderFromAddress,
          Subject,
          RecipientEmailAddress,
          DeliveryAction,
          NetworkMessageId
| order by Timestamp desc
| summarize count() by SenderFromAddress
```

[Source](https://github.com/alexverboon/Hunting-Queries-Detection-Rules/blob/main/Defender%20For%20Office%20365/MDO-Non-RFC%20Compliant%20Emails.md)

## References
- [Strengthening Email Security: Our New Approach to Non-RFC Compliant Emails](https://techcommunity.microsoft.com/blog/microsoftdefenderforoffice365blog/strengthening-email-security-our-new-approach-to-non-rfc-compliant-emails/4338306)
- [Enhancing RFC-compliance for message header from addresses](https://support.hornetsecurity.com/hc/en-us/articles/22036971529617-Enhancing-RFC-compliance-for-message-header-from-addresses)
- [Defender for Office 365 - Identify Non-RFC Compliant Emails](https://github.com/alexverboon/Hunting-Queries-Detection-Rules/blob/main/Defender%20For%20Office%20365/MDO-Non-RFC%20Compliant%20Emails.md)
- [The Internet Message Format is defined in RFC 5322](https://www.rfc-editor.org/info/rfc5322)