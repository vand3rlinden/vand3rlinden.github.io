---
title: 'Microsoft Defender for Office 365: Safe Links policies'
date: 2023-12-29T11:19:27+01:00
draft: false
categories: ["Microsoft Defender for Office 365"]
cover: 
  image: /images/mdo-safelinks-policies/mdo-safelinks-policies-front.png
---

> _Safe Links scans URLs in incoming messages and checks the links for malicious content at the time they are clicked._

## What you can manage with a Safe Links policy
With a Safe Links policy, administrators can configure and manage this policy to protect users from clicking harmful links and being redirected to malicious websites. Safe Links provides URL scanning for links in email messages, Microsoft Teams, and supported Office 365 applications. You can create custom Safe Links policies that apply to specific users, groups, or domains. 

Although there's no default policy for Safe Links, the ***Built-in protection*** [preset security policy](https://learn.microsoft.com/en-us/defender-office-365/preset-security-policies) provides Safe Links protection for all recipients by default (users who aren't defined in the Standard or Strict preset security policies or in custom Safe Links policies).

Safe Links protection through Safe Links policies is accessible in the following locations:

- Email messages: 
  - Safe Links protection for links embedded in email messages.

- Microsoft Teams:
  - Safe Links protection for links within Teams conversations, group chats, or originating from channels.
  
- Office apps: 
  - Safe Links protection for supported Office desktop, mobile, and web applications.

## Safe Links policy configuration
You configure Safe Links policies in the Microsoft Security portal or in Exchange Online PowerShell with the `New-SafeLinksPolicy` cmdlet.

In this article, we will use the Microsoft Security portal for the configuration.

1. Go to the [Safe links Policies](https://security.microsoft.com/safelinksv2) in the Microsoft Security portal.

2. Click on 'Create'.

3. Specify a policy name such as `TENANTSHORT - Safe links policy`.

4. Under 'Users and Domains', select the users, groups, and/or domains you want to include (In my case, I configured all the accepted domains).
    - If desired, exclude groups such as Microsoft 365 groups or mail-enabled security groups.
    
5. Under 'URL & click protection settings' you can set your URL and click protection settings for Email, Teams and Office 365 Apps
    - Email: 
      - Safe Links checks a list of known, malicious links when users click links in email. URLs are rewritten by default (recommended value: `$true`).

      - Apply Safe Links to email messages sent within the organization (recommended value: `$true`).

      - Apply real-time URL scanning for suspicious links and links that point to files (recommended value: `$true`).

      - Wait for URL scanning to complete before delivering the message (recommended value: `$true`).

      - Do not rewrite URLs, do checks via Safe Links API only (recommended value: `$false`).

      - Do not rewrite the following URLs in email (recommended value: `none`).

        - URLs in the _"Don't rewrite the following URLs"_ list bypass Safe Links scanning. Report it as _"Should not have been blocked (False positive)"_ and choose _"Allow this URL"_ to prevent Safe Links from scanning during mail flow and at the time of click. This adds the URL to the [Tenant Allow/Block List](https://security.microsoft.com/tenantAllowBlockList?viewid=Url).
          - Instruction: [Report good URLs to Microsoft](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/submissions-admin?view=o365-worldwide#report-good-urls-to-microsoft)

      - Rewrite URLs example:

        ![IMAGE](/images/mdo-safelinks-policies/mdo-safelinks-policies-rewriteURLs.png)
      
      - Do not rewrite URLs example:

        ![IMAGE](/images/mdo-safelinks-policies/mdo-safelinks-policies-donotrewriteURLs.png)

    - Teams:
      - Safe Links checks a list of known, malicious links when users click links in Microsoft Teams. URLs are not rewritten (recommended value: `$true`).
        - This setting may require up to 24 hours to become effective. It influences the functionality of time-of-click protection.
    
    - Office 365 apps:
      - Safe Links checks a list of known, malicious links when users click links in Microsoft Office apps. URLs are not rewritten (recommended value: `$true`).
        - Safe Links is supported in Office 365 desktop and mobile (iOS and Android) apps.
    
    - Track user clicks (`$true`):
      - Let users click through to the original URL (recommended value: `$false`).
        - This disables the option to prevent users from clicking through to the original URL in [warning pages](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/safe-links-about?view=o365-worldwide#warning-pages-from-safe-links).
      - Display the organization branding on notification and warning pages (recommended value: `none`).

    - Notification:
      - Use custom notification text (recommended value: `none`).
  
6. Save you new Safe links policy

> ***NOTE:*** Allow up to 6 hours for a new or updated policy to be applied.

## Reference
- [Set up Safe Links policies in Microsoft Defender for Office 365](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/safe-links-policies-configure)
- [Safe Links settings for email messages](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/safe-links-about?view=o365-worldwide#safe-links-settings-for-email-messages)
- [Safe Links settings for Microsoft Teams](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/safe-links-about?view=o365-worldwide#safe-links-settings-for-microsoft-teams)
- [Safe Links settings for Office apps](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/safe-links-about?view=o365-worldwide#safe-links-settings-for-office-apps)
- [Recommended Safe Links policy settings](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/recommended-settings-for-eop-and-office365?view=o365-worldwide#safe-links-policy-settings)