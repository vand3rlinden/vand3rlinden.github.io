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
### User impersonation protection
With user impersonation protection, you can protect 350 internal and external users with key roles. Internally, these could be your CEO, CFO, and other senior executives. Externally, they could be council members or your board of directors.

> **NOTE:** User impersonation protection is not effective when there is a history of email communication between the sender and recipient. Detection of an impersonation attempt is only possible in cases where there is no prior email interaction between the sender and recipient.

### How it works
User impersonation is the combination of the user’s display name and email address. For example, your CEO ```Shaggy Rogers <shaggy@contoso.com>``` could be impersonated as Shaggy Rogers, but with a completely different email address, such as ```Shaggy Rogers <shaggy.rogers@fabrikam.com>```. Even though SPF, DKIM and DMARC will pass for the fabrikam.com domain, the email will be flagged as protection policy category UIMP (user impersonation) in the ```X-Forefront-Antispam-Report``` message header.

## Domain impersonation protection
### What you can protect:
With domain impersonation protection, you can protect the domains that you own in Microsoft 365 (accepted domains) as well as external domains, such as the domains of your vendors.

### How it works
Domain impersonation prevents the sender’s email domain from appearing in a message that looks like a real email domain. For example, ```contoso.com``` could be impersonated as ```c0ntoso.com``` or ```contoso.co```. Even though SPF, DKIM and DMARC will pass for the domain ```c0ntoso.com``` or ```contoso.co```, the email will be labeled with Protection Policy Category ```DIMP``` (domain impersonation) in the ```X-Forefront-Antispam-Report``` message header.

## Mailbox intelligence
### What you can protect:
Mailbox Intelligence uses artificial intelligence (AI) to determine the email patterns of users with their most frequent contacts.

### How it works:
Mailbox intelligence operates similarly to user impersonation protection; however, it utilizes the contents of the mailbox to identify phishing attempts. For instance, if you regularly exchange emails with ```Shaggy Rogers <shaggy@contoso.com>```, and one day you receive an email from ```Shaggy Rogers <shaggy.rogers@fabrikam.com>``` that successfully passes SPF, DKIM, and DMARC checks, it will still be flagged with the protection policy category ```GIMP``` (Mailbox Intelligence Based Impersonation) in the ```X-Forefront-Antispam-Report``` message header. This is because the AI within Mailbox Intelligence assesses that you have not previously interacted with ```Shaggy Rogers <shaggy.rogers@fabrikam.com>```, suggesting a potential impersonation.

Mailbox Intelligence has two settings:

1. **Enable Mailbox Intelligence:** This setting helps the AI distinguish between messages from legitimate and impersonated senders. By default, this setting is turned on.
2. **Enable Impersonation Protection:** By default, this setting is off. This setting uses the contact history learned from Mailbox Intelligence (both frequent contacts and no contacts).

To activate Mailbox Intelligence, both settings must be turned on.

> **NOTE:** Mailbox intelligence protection is inactive when there is a history of email communication between the sender and recipient. It becomes active and identifies a message as an impersonation attempt if there has been no prior email interaction between the sender and recipient.

## Spoof intelligence
### What Spoofing is
Spoofing (implicit failures) occurs when the ```From``` address (P2 Sender, the sender address that's shown in email clients) in an email message doesn’t match the domain of the email source (P1 Sender).

### P1 vs P2- sender explanation
| Postal Letter      | Precise Term                        | Protected by  |
| -----------        | -----------                         | -----------   |
| Sender on envelope | ```RFC5321.MailFrom``` (P1 Sender)  |  SPF          |
| Author on letter   | ```RFC5322.From``` (P2 Sender)      |  DKIM + DMARC |

### What you can protect
With spoof intelligence enabled, you control the response when the P1 Sender doesn’t match the P2 Sender. This check is similar to DMARC, though not all domains have DMARC configured.

### How it works
Phishers may use P1 spoofing, allowing SPF to pass on their domain (P1 Sender) while sending emails on behalf of another domain (P2 Sender). Spoof intelligence identifies this and labels the email with Protection Policy Category ```SPOOF``` (Spoofing) in the ```X-Forefront-Antispam-Report``` message header.

## Configure anti-phishing policies and best practices
> The best practices are based on the [Strict recommendations settings](https://learn.microsoft.com/en-us/defender-office-365/recommended-settings-for-eop-and-office365?view=o365-worldwide#eop-anti-phishing-policy-settings) of the Configuration Analyzer.

1. Sign in to the Microsoft Defender portal and navigate to the [Anti-phishing section](https://security.microsoft.com/antiphishing).

2. Navigate to 'Phishing Threshold & Protection' and select 'Edit Protection Settings'.

3. Set the Phishing email threshold to '4 - Most Aggressive'.
   - A higher phishing threshold value results in stricter actions for phishing detections. Since phishing attacks are still the number one method of gaining access to an environment, you should not risk lowering this threshold.

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

6. In 'Add Trusted Senders and Domains', you can specify senders or domains that will not be flagged for impersonation.
    - ***Trusted Senders and Domains*** does not bypass all spam, spoofing, phishing protection and sender authentication (SPF, DKIM, DMARC) like ***Allowed Senders or Allowed Domains*** in the inbound anti-spam policy. However, it is still not recommended for use, especially for domains. Messages from the specified senders and sender domains will never be classified by the policy as impersonation-based attacks. Require that your key users use no other address to communicate within your environment. But, depending on your organization, only add senders or domains that are incorrectly identified as impersonation attempts. Once you have added entries, monitor the list frequently.

7. Both 'Enable Mailbox Intelligence' and 'Enable Intelligence for Impersonation Protection' should be checked, as explained earlier.

8. Check 'Spoof Intelligence' and click on Save.

9. After saving, navigate to ‘Action’ and select 'Edit Actions'.

10. I recommend setting all actions to 'Quarantine the message'.
      - My preference is to select a quarantine policies to ***request to release*** except for the quarantine policy for mailbox intelligence. 

11. Turn on 'Honor DMARC record policy when the message is detected as spoof', this setting will honor the sender's DMARC policy for email authentication failures (explicit failures).
      - Setting: If the message is detected as spoof and DMARC Policy is set as ***p=quarantine***
        - Action: Quarantine the message
      - Setting: If the message is detected as spoof and DMARC Policy is set as ***p=reject***
        - Action: Reject the message (NDR)

12. Check all safety tips, to help recipients be more aware of red flags in an email.

## Self-to-self spoofing attack with DMARC reject policy
From the field I have seen that when a user is attacked by self-to-self spoofing. They receive an NDR from Exchange Online with the original email attached in .eml format, expected but unwanted. I have contacted Microsoft and they recently fixed this issue and this NDR backscatter should get a high confidence spam (HSPM) or spam (SPM) verdict and the email will end up in the JUNK folder. Backscatter is treated differently than regular email and the HSPM and SPM actions in the anti-spam policies do not apply.

![IMAGE](/images/mdo-anti-phishing-policies/mdo-anti-phishing-policies-1.png)

## In summary
### User Impersonation:
- User impersonation involves combining the user’s display name and email address, and it can be set up for a maximum of 350 users.
- Protection Policy Category (CAT): `UIMP` (User impersonation) in the X-Forefront-Antispam-Report header

### Domain Impersonation:
- Domain impersonation occurs when the domain is manipulated to resemble the legitimate domain.
- Protection Policy Category (CAT): `DIMP` (Domain impersonation) in the X-Forefront-Antispam-Report header

### Mailbox Intelligence:
- Operates similarly to user impersonation protection; however, it utilizes the contents of the mailbox to identify phishing attempts.
- Protection Policy Category (CAT): `GIMP` (Mailbox intelligence impersonation) in the X-Forefront-Antispam-Report header
  
### Spoof Intelligence:
- Spoofing takes place when the From address (P2 Sender) in an email message does not match the domain of the email source (P1 Sender).
- Protection Policy Category (CAT): `SPOOF` (Spoofing) in the X-Forefront-Antispam-Report header
  
## Reference
- [Impersonation settings in anti-phishing policies in Microsoft Defender for Office 365](https://learn.microsoft.com/en-us/defender-office-365/anti-phishing-policies-about?view=o365-worldwide#impersonation-settings-in-anti-phishing-policies-in-microsoft-defender-for-office-365)
- [User impersonation protection](https://learn.microsoft.com/en-us/defender-office-365/anti-phishing-policies-about?view=o365-worldwide#user-impersonation-protection)
- [Domain impersonation protection](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/anti-phishing-policies-about?view=o365-worldwide#domain-impersonation-protection)
- [Mailbox intelligence impersonation protection](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/anti-phishing-policies-about?view=o365-worldwide#mailbox-intelligence-impersonation-protection)
- [Spoof settings](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/anti-phishing-policies-about?view=o365-worldwide#spoof-settings)
- [Spoof protection and sender DMARC policies](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/anti-phishing-policies-about?view=o365-worldwide#spoof-protection-and-sender-dmarc-policies)
- [X-Forefront-Antispam-Report message header fields](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/message-headers-eop-mdo?view=o365-worldwide#x-forefront-antispam-report-message-header-fields)
- [Impersonation safety tips](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/anti-phishing-policies-about?view=o365-worldwide#impersonation-safety-tips)
- [Recommended anti-phishing policy settings](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/recommended-settings-for-eop-and-office365?view=o365-worldwide#eop-anti-phishing-policy-settings)
- [First contact safety tip](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/anti-phishing-policies-about?view=o365-worldwide#first-contact-safety-tip)