---
title: 'Microsoft Defender for Office 365: Handling False Positives and False Negatives'
date: 2025-02-10T17:27:49+01:00
draft: false
categories: ["Microsoft Defender for Office 365"]
cover: 
  image: /images/mdo-handling-false-positives-false-negatives/mdo-handling-false-positives-false-negatives-front.png
---

> _False positives block important emails, while false negatives allow harmful ones through. Learn how to manage these emails effectively in Microsoft Defender for Office 365._

## How inbound email works in Microsoft 365
To understand why your environment experiences false positives and false negatives, you first need to understand how Microsoft 365 handles inbound email.

Microsoft 365 uses implicit email authentication to verify inbound email. This approach goes beyond traditional SPF, DKIM, and DMARC checks by incorporating additional signals to evaluate inbound email. By leveraging these extra signals, emails that would typically fail standard authentication can pass implicit authentication and be successfully delivered to Microsoft 365.

These signals include:
- Sender reputation
- Sender history
- Recipient history
- Behavioral analysis
- Other advanced techniques

The results of Microsoft’s implicit authentication checks are combined into a single value called composite authentication (`compauth`). This value is stamped into the `Authentication-Results` header within the message headers.

![IMAGE](/images/mdo-handling-false-positives-false-negatives/mdo-handling-false-positives-false-negatives-1.png)

## False positives and False negatives
- **False Positive (`FP`)**: is when an email isn't actually spam, but your system mistakenly thinks it is and puts it in the spam or quarantine folder.
- **False Negative (`FN`)**: happens when an email is actually spam, but your system mistakenly lets it through, thinking it's not spam.

Sometimes the signals of the implicit authentication gets it wrong, and therefore a good message can be a flagged as a bad one (false positive) and a questionable email message can be get through (false negative).

To help these signals to get better you can [report good emails](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/submissions-admin?view=o365-worldwide#report-good-email-to-microsoft) as False Positive or [report questionable emails](https://learn.microsoft.com/en-us/defender-office-365/submissions-admin#report-questionable-email-to-microsoft) as False Negative to Microsoft using the Report Submission page. 

## Handling false positives
Once you report a false positive ([good email](https://learn.microsoft.com/en-us/defender-office-365/submissions-admin?view=o365-worldwide#report-good-email-to-microsoft)), you can, if desired, create an allow entry in the Tenant Allow/Block List for domains, email addresses, files, and URLs. 

These entries are retained for 45 days after the filtering system determines that the entity is clean, and then the allow entry is removed. For example: If you create an allow entry on July 1 with removal set for 45 days after the last use, and the entity is deemed malicious until July 29 but clean from July 30, the last used date updates until July 29. Since it’s clean, updates stop on July 30, and the entry is removed on September 12—45 days after the last update.

> **NOTE**: You cannot report IP addresses through the Submissions page, but you can manually create an allow entry in the Tenant Allow/Block List. Note that only [IPv6 addresses are supported](https://learn.microsoft.com/en-us/defender-office-365/tenant-allow-block-list-ip-addresses-configure#create-allow-entries-for-ipv6-addresses), IPv4 addresses are not.

In addition to implicit authentication checks, you can also reduce false positives by disabling the [ASF (Advanced Spam Filter) settings](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/anti-spam-policies-asf-settings-about) in your Anti-spam inbound policy, as enabling one or more ASF settings is an aggressive approach to spam filtering that often results in false positives. For example, messages containing certain elements may be marked as spam or have their spam score increased. Additionally, messages filtered by ASF cannot be reported to Microsoft as false positives.

## Handling false negatives
Once you report a false negative ([questionable email](https://learn.microsoft.com/en-us/defender-office-365/submissions-admin#report-questionable-email-to-microsoft)), you can, if desired, create a block entry in the Tenant Allow/Block List for domains and email addresses, files and URLs. 

These entries expire after 30 days, but you can also set them to expire after 90 days or never. Block entries for spoofed senders and IP addresses never expire.

> **NOTE**: You cannot report IP addresses through the Submissions page, but you can manually create a block entry in the Tenant Allow/Block List. Note that only [IPv6 addresses are supported](https://learn.microsoft.com/en-us/defender-office-365/tenant-allow-block-list-ip-addresses-configure#create-block-entries-for-ipv6-addresses), IPv4 addresses are not.

In addition to the implicit authentication checks and the recommended [anti-phishing](https://vand3rlinden.com/post/mdo-anti-phishing-policies/) and [anti-spam](https://vand3rlinden.com/post/mdo-anti-spam-policies/) techniques provided by MDO, end users should remain vigilant by identifying red flags in email messages, such as carefully reviewing the sender address, subject line, and message content.

To give end users more red flags to look for when they receive an email message, the following can be used or implemented:
- [First contact safety tip](https://learn.microsoft.com/en-us/defender-office-365/anti-phishing-policies-about#first-contact-safety-tip)
- [External Email Tagging](https://learn.microsoft.com/en-us/powershell/module/exchange/set-externalinoutlook)

You must also allow users to report false negatives (questionable emails) and false positives (good emails) to Microsoft by enabling the [built-in Report button](https://learn.microsoft.com/en-us/defender-office-365/submissions-user-reported-messages-custom-mailbox?view=o365-worldwide#use-the-microsoft-defender-portal-to-configure-user-reported-settings), if you haven’t already done so, or by using a third-party solution such as KnowBe4.

> **NOTE**: If you are still using the Microsoft Report Message or Report Phishing add-ins, consider [transitioning to the built-in report button](https://learn.microsoft.com/en-us/defender-office-365/submissions-users-report-message-add-in-configure). These add-ins have security vulnerabilities that make them unsafe for your organization and will be deprecated over time. 

Finally, end users can be educated with a security awareness program to help them recognize questionable emails. See also [this blog post](https://vand3rlinden.com/post/mdo-attack-simulation/).

## Reference
- [Inbound email authentication for mail sent to Microsoft 365](https://learn.microsoft.com/en-us/defender-office-365/email-authentication-about#inbound-email-authentication-for-mail-sent-to-microsoft-365)
- [Use the Submissions page to submit suspected spam, phish, URLs, legitimate email getting blocked, and email attachments to Microsoft](https://learn.microsoft.com/en-us/defender-office-365/submissions-admin)
- [Manage allows and blocks in the Tenant Allow/Block List](https://learn.microsoft.com/en-us/defender-office-365/tenant-allow-block-list-about)
- [Automate Tenant Allow/Block List entries](https://techcommunity.microsoft.com/blog/microsoftdefenderforoffice365blog/automate-tenant-allowblock-list-entries/4213201)