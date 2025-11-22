---
title: 'Microsoft Defender for Office 365: Safe Attachments policies'
date: 2024-07-12T22:43:43+02:00
draft: false
categories: ["Microsoft Defender for Office 365"]
cover: 
  image: /images/mdo-safeattachments-policies/mdo-safeattachments-policies-front.png
---

>_Safe Attachments scans and evaluates attachments for malicious content before delivering messages to recipients._

## What you can manage with a Safe Attachments policy
With a Safe Attachments policy, administrators can configure an additional layer of protection against malicious content in email attachments. Safe Attachments scans and evaluates attachments in a virtual environment before delivering messages to recipients. You can create a custom policy to, specify actions for unknown malware, select a quarantine policy, and configure global settings to protect files in SharePoint, OneDrive, and Teams with Safe Attachments.

Although there's no default Safe Attachments policy, the ***Built-in protection*** [preset security policy](https://learn.microsoft.com/en-us/defender-office-365/preset-security-policies) provides Safe Attachments protection to all recipients (users who aren't defined in the Standard or Strict preset security policies or in custom Safe Attachments policies).

## Safe Attachments configuration
You configure Safe Attachments policies in Microsoft Defender XDR or in Exchange Online PowerShell with the `New-SafeAttachmentRule` cmdlet.

In this article, we will use Microsoft Defender XDR for the configuration.

1. Go to the [Safe Attachments policies](https://security.microsoft.com/safeattachmentv2) in Microsoft Defender XDR.

2. Click on 'Global settings'.

3. Check 'Turn on Defender for Office 365 for SharePoint, OneDrive, and Microsoft Teams'.
    - If a file in any SharePoint, OneDrive, or Microsoft Teams library is detected as malicious, [Safe Attachments for SharePoint, OneDrive, and Microsoft Teams](https://learn.microsoft.com/en-us/defender-office-365/safe-attachments-for-spo-odfb-teams-about) will prevent users from opening and downloading the file.

4. Check 'Turn on Safe Documents for Office clients'.
    - To protected your users, [Safe Documents](https://learn.microsoft.com/en-us/defender-office-365/safe-documents-in-e5-plus-security-about) sends file information to the Microsoft Defender for Endpoint cloud for analysis (Only available with Microsoft 365 E5 or Microsoft 365 E5 Security license). 

5. Click on 'Save'.

6. Click on 'Create'.

7. Specify a policy name such as `TENANTSHORT - Safe Attachments policy`.

8. Under 'Users and Domains', select the users, groups, and/or domains you want to include (In my case, I configured all the accepted domains).
    - If desired, exclude groups such as Microsoft 365 groups or mail-enabled security groups.

9. Under 'Safe Attachments unknown malware response', choose: 'Block - Block current and future messages and attachments with detected malware'.

10. Under 'Quarantine policy', select a 'Request to release' policy. Preferred: AdminOnlyAccessPolicy. 

11. Do not configure the 'Redirect messages with detected attachments'. If necessary, you can download the message from Quarantine to investigate the attachment in a sandbox.

> ***NOTE:*** Allow up to 6 hours for a new or updated policy to be applied.

## Reference
- [Recommended Safe Attachments settings](https://learn.microsoft.com/en-us/defender-office-365/recommended-settings-for-eop-and-office365?view=o365-worldwide#safe-attachments-policy-settings)
- [Safe Attachments in Microsoft Defender for Office 365](https://learn.microsoft.com/en-us/defender-office-365/safe-attachments-about)