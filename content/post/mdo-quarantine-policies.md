---
title: 'Microsoft Defender: Threat policies — Setup and use Quarantine policies'
date: 2023-12-11T16:33:46+01:00
draft: false
categories: ["Microsoft Defender for Office 365"]
cover: 
  image: /images/mdo-quarantine-policies/mdo-quarantine-policies-front.png
---

> _Quarantine policies let you control the quarantine on how users can use it. This post will cover the default policies and how to create a custom policy._

**Product using:** Microsoft Defender for Office 365 (MDO)

## What are quarantine policies
Back in April 2020, Microsoft made it possible for users to view, release, or delete quarantined messages (expect high confidence phishing). Some organizations were not happy about users having access to their own quarantined items. Quarantine policies give you more control over quarantine for your end users and which quarantined items they can release.

## The default quarantine policies
You can access the quarantine policies by going to: https://security.microsoft.com/quarantinePolicies. 

Out of the box, you will see three policies: 
- DefaultFullAccessPolicy 
- AdminOnlyAccessPolicy
- DefaultFullAccessWithNotificationPolicy

### DefaultFullAccessPolicy
This policy has the quarantine settings as we have them since April 2020, and includes the following settings (expect high confidence phishing):

**User message access:**
- release the message from quarantine
- block sender (Outlook block sender list, [junk mail settings](https://support.microsoft.com/en-us/office/filter-junk-email-and-spam-in-outlook-db786e79-54e2-40cc-904f-d89d57b7f41d))
- delete the message
- preview the message

**Quarantine notification:**
- Disabled

### AdminOnlyAccessPolicy
This policy is the default High Confidence Phishing (HSPM) policy and assigns _no access_ permissions to the items, and includes the following settings:

**User message access:**
- No allowed actions

**Quarantine notification:**
- Disabled

### DefaultFullAccessWithNotificationPolicy
This policy has the same settings as the _DefaultFullAccessPolicy_ quarantine policy, but with quarantine notifications enabled, and includes the following settings:

**User message access:**
- release the message from quarantine
- block sender (Outlook block sender list, [junk mail settings](https://support.microsoft.com/en-us/office/filter-junk-email-and-spam-in-outlook-db786e79-54e2-40cc-904f-d89d57b7f41d))
- delete the message
- preview the message

**Quarantine notification:**
- Enabled

## Create a custom quarantine policy, such as for request to release
In some situations, you may want your users to be able to request that a message be released from quarantine, rather than releasing it themselves. To do this, you can create a custom quarantine policy by following the steps below:

- Step 1: Go to https://security.microsoft.com/quarantinePolicies
- Step 2: Click on 'Add custom policy'
- Step 3: Specify a policy name such as _RequestReleaseFullAccessPolicy_
- Step 4: Under Recipient Message Access select _Limited access_

![IMAGE](/images/mdo-quarantine-policies/mdo-quarantine-policies-newpolicy.png) 
*With **Set specific access** you can turn on or off each **User message access action** as desired, for more advanced configurations.*

- Step 5: Enable quarantine notifications, if desired
- Step 6: Safe your policy

### Outcome:
The _User message access_ actions for the created custom quarantine policy will be:
- request to release the message from quarantine
- block sender (Outlook block sender list, [junk mail settings](https://support.microsoft.com/en-us/office/filter-junk-email-and-spam-in-outlook-db786e79-54e2-40cc-904f-d89d57b7f41d))
- delete the message
- preview the message

**Quarantine notification:**
- Enabled

Users can _request release_ of quarantined items after you assign the quarantine policy to an action in one of the threat policies, as described above:

![IMAGE](/images/mdo-quarantine-policies/mdo-quarantine-policies-requestrelease.png)

_Tenant Admins_ or _[Quarantine Administrators](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/mdo-portal-permissions?view=o365-worldwide#modify-email--collaboration-role-group-role-assignments-in-the-microsoft-defender-portal)_ can approve or deny release requests in the quarantine:

![IMAGE](/images/mdo-quarantine-policies/mdo-quarantine-policies-approverelease.png)

## Global settings
You can also change the notification email settings under _Global Settings_, where you can set settings such as the recurrence of quarantine notifications or change the layout of the email.

## In summary
After you have set up the quarantine policies the way you want them, you can use your quarantine policies in the threat policies actions.

[Reference](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/quarantine-policies)
