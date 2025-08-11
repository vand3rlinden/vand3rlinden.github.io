---
title: 'Microsoft Defender for Office 365: Anti-phishing policies'
date: 2023-12-22T14:27:30+01:00
draft: false
categories: ["Microsoft Defender for Office 365"]
cover: 
  image: /images/mdo-anti-phishing-policies/mdo-anti-phishing-policies-front.png
---

> _Phishing is an email attack that aims to steal sensitive information through messages that appear to be from legitimate or trusted senders. You can enhance the security of your Exchange Online mailboxes by implementing anti-phishing policies._

## What you can manage with Anti-phishing policies
Anti-phishing policies provide enhanced control over incoming phishing emails, for instance, in cases where someone may attempt to impersonate your CEO or send messages from a domain that closely resembles yours. By default, a policy named 'Office365 AntiPhish Default (Default)' is automatically applied to all users.

## In an anti-phishing policy, you can configure
## Impersonation protection
Impersonation occurs when the sender of an email appears similar to a legitimate or expected sender's address. Attackers often use impersonated email addresses in phishing or other attacks to deceive recipients and gain their trust. There are two main types of impersonation:

- **User impersonation:** Contains subtle differences in the email alias.
- **Domain impersonation:** Contains subtle differences in the domain.

### User impersonation protection
### What you can protect:
User impersonation protection allows you to secure up to 350 internal and external key users from impersonation attempts. Internally, this might include roles like the CEO, CFO, and other senior executives. Externally, it could cover individuals such as council members or board directors.

> **NOTE:** User impersonation protection is not effective when there is a history of email communication between the sender and recipient. Detection of an impersonation attempt is only possible in cases where there is no prior email interaction between the sender and recipient.

### How it works
User impersonation involves using a combination of the user’s display name and email address to make the email appear similar to that of a legitimate or trusted sender. For example, your CEO `Shaggy Rogers <shaggy@contoso.com>` could be impersonated as Shaggy Rogers, but with a completely different email address, such as `Shaggy Rogers <shaggy.rogers@fabrikam.com>`. Even though SPF, DKIM and DMARC will pass for the `fabrikam.com` domain, the email will be flagged as protection policy category `UIMP` (user impersonation) in the `X-Forefront-Antispam-Report` message header.

### Domain impersonation protection
### What you can protect:
Domain impersonation protection helps secure both your organization’s domains (accepted domains) and external domains, such as vendor domains, from impersonation attempts.

### How it works
Domain impersonation occurs when the sender's domain is manipulated to resemble the legitimate domain. For example, `contoso.com` could be impersonated as `c0ntoso.com` or `contoso.co`. Even though SPF, DKIM and DMARC will pass for the domain `c0ntoso.com` or `contoso.co`, the email will be labeled with Protection Policy Category `DIMP` (domain impersonation) in the `X-Forefront-Antispam-Report` message header.

### Impersonation insight
You can use Impersonation insight in the [Microsoft Defender Portal](https://security.microsoft.com/impersonationinsight) to quickly identify messages from impersonated senders or sender domains that are included in impersonation protection in anti-phishing policies.

## Mailbox intelligence
### What you can protect:
Mailbox Intelligence uses artificial intelligence (AI) to determine the email patterns of users with their most frequent contacts.

### How it works:
Mailbox intelligence operates similarly to user impersonation protection; however, it utilizes the contents of the mailbox to identify phishing attempts. For instance, if you regularly exchange emails with `Shaggy Rogers <shaggy@contoso.com>`, and one day you receive an email from `Shaggy Rogers <shaggy.rogers@fabrikam.com>` that successfully passes SPF, DKIM, and DMARC checks, it will still be flagged with the protection policy category `GIMP` (Mailbox Intelligence Based Impersonation) in the `X-Forefront-Antispam-Report` message header. This is because the AI within Mailbox Intelligence assesses that you have not previously interacted with `Shaggy Rogers <shaggy.rogers@fabrikam.com>`, suggesting a potential impersonation.

Mailbox Intelligence has two settings:

1. **Enable Mailbox Intelligence:** This setting helps the AI distinguish between messages from legitimate and impersonated senders. By default, this setting is turned on.
2. **Enable Impersonation Protection:** By default, this setting is off. This setting uses the contact history learned from Mailbox Intelligence (both frequent contacts and no contacts).

To activate Mailbox Intelligence, both settings must be turned on.

> **NOTE:** Mailbox intelligence protection is inactive when there is a history of email communication between the sender and recipient. It becomes active and identifies a message as an impersonation attempt if there has been no prior email interaction between the sender and recipient.

## Spoof intelligence
### What Spoofing is
Spoofing occurs when the `From` address (P2 Sender, the sender address that's shown in email clients) in an email message doesn’t match the domain of the email source (P1 Sender).

### P1 vs P2- sender explanation
| Postal Letter      | Precise Term                        | Protected by  |
| -----------        | -----------                         | -----------   |
| Sender on envelope | `RFC5321.MailFrom` (P1 Sender)      |  SPF          |
| Author on letter   | `RFC5322.From` (P2 Sender)          |  DKIM + DMARC |

### What you can protect
When Spoof Intelligence is enabled, Spoof Intelligence Insight provides a list of spoofed senders that have been automatically detected and either allowed or blocked by the system. Exchange Online Protection (EOP) evaluates messages and determines whether to allow or block them using a combination of standard email authentication methods (SPF, DKIM and DMARC), with sender reputation, behavioral analysis, and other advanced techniques ([implicit email authentication](https://learn.microsoft.com/en-us/defender-office-365/email-authentication-about#inbound-email-authentication-for-mail-sent-to-microsoft-365)).

![IMAGE](/images/mdo-anti-phishing-policies/mdo-anti-phishing-policies-2.png)

### How it works
If Spoof Intelligence has good signals that a domain may be suspicious, Spoof Intelligence will identify it as a **block** and mark the email with protection policy category `SPOOF` in the `X-Forefront-Antispam-Report` message header. However, if Spoof Intelligence has good signals that a domain should pass extended authentication checks, it will be **allowed** by Spoof Intelligence and the [Unauthenticated Sender Indicators](https://learn.microsoft.com/en-us/defender-office-365/anti-phishing-policies-about?view=o365-worldwide#unauthenticated-sender-indicators) should be visible to the recipient.

### Spoof intelligence insight
You can use the Spoof Intelligence Insight in the [Microsoft Defender portal](https://security.microsoft.com/spoofintelligence) to identify spoofed senders who are sending unauthenticated emails (messages that fail SPF, DKIM, or DMARC and sender reputation checks). Spoof Intelligence Insight allows you to manually [override](https://learn.microsoft.com/en-us/defender-office-365/anti-spoofing-spoof-intelligence#override-the-spoof-intelligence-verdict) the Spoof Intelligence verdict to either **allow** or **block** the detected spoofed senders. However, once overridden, the spoofed sender will no longer appear in the Spoof Intelligence Insight and will instead be listed under the **Spoofed Senders tab** on the [Tenant Allow/Block Lists](https://security.microsoft.com/tenantAllowBlockList?viewid=SpoofItem) page.

## Configure anti-phishing policies and best practices
> The best practices are based on the [Strict recommendations settings](https://learn.microsoft.com/en-us/defender-office-365/recommended-settings-for-eop-and-office365?view=o365-worldwide#eop-anti-phishing-policy-settings) of the Configuration Analyzer.

1. Sign in to the Microsoft Defender portal and navigate to the [Anti-phishing section](https://security.microsoft.com/antiphishing).

2. Navigate to 'Phishing Threshold & Protection' and select 'Edit Protection Settings'.

3. Set the Phishing email threshold to '4 - Most Aggressive'.
   - A higher phishing threshold means more sensitivity for applying machine learning models to messages for determining a phishing verdict. Since phishing attacks are still the number one method of gaining access to an environment, you should not risk lowering this threshold.

4. Check 'Enable Users to Protect' (User impersonation protection), you can include up to 350 key users. 

- For bulk user additions, create a CSV file in the following format:
```
Policy,Users
Office365 AntiPhish Default,Firstname Lastname;user1@domain.com
Office365 AntiPhish Default,Firstname Lastname;user2@domain.com
```

- To import this CSV into the policy using the Exchange PowerShell Module, execute the following command:
```
$Users = Import-CSV -Path 'C:\temp\users.csv'

ForEach ($User in $Users){
  
        Set-AntiPhishPolicy -Identity $User.Policy -TargetedUsersToProtect @{add=$User.Users}
        Write-Host -ForegroundColor Green $User.Users "is added!"
}
```

5. Check 'Enable domains to protect' (Domain impersonation protection).
   - These can be your own domains (Include domains I own) or domains that belong to partners or suppliers (Include custom domains).

6. In 'Add Trusted Senders and Domains', you can specify senders or domains that will not be flagged for impersonation.
    - ***Trusted Senders and Domains*** does not bypass all spam, spoofing, phishing protection and sender authentication (SPF, DKIM, DMARC) like ***Allowed Senders or Allowed Domains*** in the inbound anti-spam policy. However, it is still not recommended for use, especially for domains. Messages from the specified senders and sender domains will never be classified by the policy as impersonation-based attacks. Require that your key users use no other address to communicate within your environment. But, depending on your organization, only add senders or domains that are incorrectly identified as impersonation attempts. Once you have added entries, monitor the list frequently.

7. Both 'Enable Mailbox Intelligence' and 'Enable Intelligence for Impersonation Protection' should be checked, as explained earlier.

8. Check 'Spoof Intelligence' and click on Save.

9.  After saving, navigate to ‘Action’ and select 'Edit Actions'.

10. I recommend setting all actions to 'Quarantine the message'.
      - Select ***request to release*** [quarantine policies](https://vand3rlinden.com/post/mdo-quarantine-policies/) for all actions, except the mailbox intelligence action. 

11. Turn on 'Honor DMARC record policy when the message is detected as spoof', this setting will honor the sender's DMARC policy for email authentication failures (explicit failures).
      - Setting: If the message is detected as spoof and DMARC Policy is set as ***p=quarantine***
        - Action: Quarantine the message
      - Setting: If the message is detected as spoof and DMARC Policy is set as ***p=reject***
        - Action: Reject the message (NDR)

> NOTE: The Honor DMARC record policy actions are **only effective** if a message is detected as Spoof by Spoof Intelligence. If EOP, through its [implicit email authentication checks](https://learn.microsoft.com/en-us/defender-office-365/email-authentication-about#inbound-email-authentication-for-mail-sent-to-microsoft-365), allows the sender based on the results in the [Composite Authentication section](https://learn.microsoft.com/en-us/defender-office-365/email-authentication-about#composite-authentication), a DMARC fail may still reach your recipients. To address this, consider creating a mail flow rule to reject such messages. However, it’s advisable to review [this](https://vand3rlinden.com/post/mdo-hardening-dkim-dmarc-config/#enforce-inbound-dmarc-rejection-policy-to-reject) blog post for a detailed approach.

12. Check all safety tips, to help recipients be more aware of red flags in an email.

## EXTRA: Self-to-self spoofing attack
I’ve noticed that when a user is targeted by self-to-self spoofing, and the message is detected by Spoof Intelligence, with the action for **If the message is detected as spoof and DMARC Policy is set as p=reject** is configured to **Reject the message**. The user receives an NDR in their Inbox from Exchange Online with the original spoofed email attached in `.eml` format. While expected behavior, this outcome is typically unwanted. I contacted Microsoft about this issue, and they have implemented a fix. Now, this NDR backscatter is assigned either a high-confidence spam (`HSPM`) or spam (`SPM`) verdict, ensuring the email ends up in the Junk Folder. The NDR backscatter is treated differently than regular email and the `HSPM` and `SPM` actions in the inbound anti-spam policies do not apply.

![IMAGE](/images/mdo-anti-phishing-policies/mdo-anti-phishing-policies-1.png)

## In summary
### User Impersonation:
- User impersonation involves using a combination of the user’s display name and email address to make the email appear similar to that of a legitimate or trusted sender.
- Protection Policy Category (CAT): `UIMP` (User impersonation) in the X-Forefront-Antispam-Report header

### Domain Impersonation:
- Domain impersonation occurs when the sender's domain is manipulated to resemble the legitimate domain.
- Protection Policy Category (CAT): `DIMP` (Domain impersonation) in the X-Forefront-Antispam-Report header

### Mailbox Intelligence:
- Operates similarly to user impersonation protection; however, it utilizes the contents of the mailbox to identify phishing attempts.
- Protection Policy Category (CAT): `GIMP` (Mailbox intelligence impersonation) in the X-Forefront-Antispam-Report header
  
### Spoof Intelligence:
- Spoofing takes place when the `From` address (P2 Sender) in an email message does not match the domain of the email source (P1 Sender).
- Protection Policy Category (CAT): `SPOOF` (Spoofing) in the X-Forefront-Antispam-Report header
  
## Reference
- [Anti-phishing protection](https://learn.microsoft.com/en-us/defender-office-365/anti-phishing-protection-about)
- Impersonation settings:
  - [Impersonation settings in anti-phishing policies in Microsoft Defender for Office 365](https://learn.microsoft.com/en-us/defender-office-365/anti-phishing-policies-about?view=o365-worldwide#impersonation-settings-in-anti-phishing-policies-in-microsoft-defender-for-office-365)
  - [User impersonation protection](https://learn.microsoft.com/en-us/defender-office-365/anti-phishing-policies-about?view=o365-worldwide#user-impersonation-protection)
  - [Domain impersonation protection](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/anti-phishing-policies-about?view=o365-worldwide#domain-impersonation-protection)
  - [Impersonation insight](https://learn.microsoft.com/en-us/defender-office-365/anti-phishing-mdo-impersonation-insight)
  - [Mailbox intelligence impersonation protection](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/anti-phishing-policies-about?view=o365-worldwide#mailbox-intelligence-impersonation-protection)
- Spoof settings:
  - [Spoof Intelligence](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/anti-phishing-policies-about?view=o365-worldwide#spoof-settings)
  - [Spoof intelligence insight](https://learn.microsoft.com/en-us/defender-office-365/anti-spoofing-spoof-intelligence)
    - [Information about spoof detections](https://learn.microsoft.com/en-us/defender-office-365/anti-spoofing-spoof-intelligence#view-information-about-spoof-detections)
  - [Anti-spoofing protection in EOP](https://learn.microsoft.com/en-us/defender-office-365/anti-phishing-protection-spoofing-about)
  - [Anti-spoofing protection FAQ](https://learn.microsoft.com/en-us/defender-office-365/anti-phishing-protection-spoofing-faq)
- Other:
  - [Spoofing vs. impersonation](https://learn.microsoft.com/en-us/defender-office-365/anti-phishing-policies-about?view=o365-worldwide#spoofing-vs-impersonation)
  - [X-Forefront-Antispam-Report message header fields](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/message-headers-eop-mdo?view=o365-worldwide#x-forefront-antispam-report-message-header-fields)
  - [Impersonation safety tips](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/anti-phishing-policies-about?view=o365-worldwide#impersonation-safety-tips)
  - [Recommended anti-phishing policy settings](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/recommended-settings-for-eop-and-office365?view=o365-worldwide#eop-anti-phishing-policy-settings)
  - [First contact safety tip](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/anti-phishing-policies-about?view=o365-worldwide#first-contact-safety-tip)