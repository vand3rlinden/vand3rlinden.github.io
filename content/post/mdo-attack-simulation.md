---
title: 'Microsoft Defender for Office 365: Attack simulation training'
date: 2024-02-03T12:31:15+01:00
draft: false
categories: ["Microsoft Defender for Office 365"]
cover: 
  image: /images/mdo-attack-simulation/mdo-attack-simulation-front.png
---

> _With Microsoft Defender for Office 365, you can create an attack simulation training to identify vulnerable users and mitigate potential threats before they impact your organization._

## Think before you click
Understanding the intricacies of cybersecurity is crucial in today's digital landscape. Attack simulation training is indispensable for users as it provides hands-on experience in recognizing and defending against potential threats. This proactive approach empowers individuals to enhance their security awareness, identify vulnerabilities, and contribute to a more resilient organizational defense against cyber attacks.

Microsoft Defender for Office 365 provides an attack simulation training if you are licensed for Microsoft Defender for Office 365 Plan 2 _(add-on licenses or included in subscriptions such as Microsoft 365 E5)_. Without the need for third-party phishing simulations, this attack simulation training can be easily set up in the Defender portal.

This blog focuses more on the user part of the attack simulation and is an extension of the Microsoft Learn documentation, which already provides a good explanation of how to set up the attack simulation in the Defender portal. We will definitely have a summary of the configuration part as well.

## Requirements
### 1: Required license
All target users must have a ***Microsoft Defender for Office 365 Plan 2*** (add-on licenses or included in subscriptions such as Microsoft 365 E5).

### 2: Report message button
You should start by giving your users the ability to report email messages, which is also necessary for this attack simulation training. To do so, you can activate the:
- [Built-in Report button](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/submissions-user-reported-messages-custom-mailbox?view=o365-worldwide#use-the-microsoft-defender-portal-to-configure-user-reported-settings) (Outlook on the web)
- [Microsoft Report Message or Report Phishing add-ins](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/submissions-users-report-message-add-in-configure?view=o365-worldwide#get-the-report-message-add-in) (all Outlook platforms)

Reported messages appear in the [User Reported](https://security.microsoft.com/reportsubmission?viewid=user) section of the Submissions page, your reporting mailbox, and are visible in the simulation report.

### 3: Required permissions 
Required role: [Attack Simulation Administrator](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/permissions-reference#attack-simulation-administrator)

Users in this role can handle every facet of attack simulations, including creation, launch, scheduling, and result review. They have full access to all simulations within the tenant.

The role is available in the [Microsoft Defender](https://security.microsoft.com/emailandcollabpermissions) portal or in Entra ID (e.g. through PIM).

### 4: Remove external tagging for the default end user notifications address
Once the user clicked on the link and/or logged on to the phishing site, they received an email From `notification@attacksimulationtraining.com`.
Exclude this email address or domain from your external tagging configuration (_Exchange Online mail flow rules_ or _Exchange Online’s External Email Tagging Feature_). 
> You can also use Tenant notifications to change the From address to an internal address.

## Creating an attack simulation training
After the requirements are set, you can begin creating an attack simulation training in the [Microsoft Defender Portal](https://security.microsoft.com/attacksimulator). You have the option to:
- [Simulate a phishing attack](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/attack-simulation-training-simulations)
  - These simulations test your security policies and practices, as well as train your employees to increase their awareness and decrease their susceptibility to attacks.
- [Using automated flows for Attack simulation](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/attack-simulation-training-simulation-automations)
  -  Creating a simulation automation is similar to creating an individual simulation, except for the ability to select multiple techniques, payloads, and the automation schedule.
- [Training campaigns for Attack simulation](https://learn.microsoft.com/en-us/defender-office-365/attack-simulation-training-training-campaigns)
  - Instead of creating and launching simulated phishing attacks that eventually lead to training, you can create and assign Training campaigns directly to users.

## The basic elements of a simulation are:
- Select a ***Social Engineering Technique***, such as credential harvesting
- Select a ***Payload*** _(phishing emails and web pages that you use to launch simulations)_
  - Global Payloads: Includes built-in payloads, such as the `Keep Office 365 Password` payload
  - Tenant Payloads: Contains [custom payloads](https://learn.microsoft.com/en-us/defender-office-365/attack-simulation-training-payloads#create-payloads), such as a fake email from an executive with an official company signature. You can also use ChatGPT (or any other AI tool) to prompt a tenant payload to generate an HTML email for SharePoint Online document sharing, for example. A prompt can be: _Can you generate an HTML email template that looks like a Microsoft email to share a SharePoint document?_ This prompt leads to [this](https://vand3rlinden.com/images/mdo-attack-simulation/example-tenant-payload/sharepoint-documentshare.html) tenant payload, the HTML content can be copied and pasted into the `Configure Payload` section and change the Dynamic tags such as `${firstName}` and `${phishingUrl}`.

> Images that you use in tenant payloads may be [blocked](https://learn.microsoft.com/en-us/defender-office-365/attack-simulation-training-faq#why-are-images-in-simulation-messages-being-blocked-by-outlook) with a message that the sender is not in the Outlook Safe Senders list. This happens by default because Outlook is configured to block automatic image downloads in messages from the Internet.

- Select a ***Login Page*** _(phish web login page for credential harvesting and link in attachment techniques)_
  - Global Login Pages: Includes built-in login pages, such as the Microsoft login page
  - Tenant Login Pages: Includes custom login pages, such as a custom Microsoft login page with corporate branding

- Select a ***Phish Landing Page*** _(provides a learning moment for the user after being phished)_
  - Global Phish Landing Pages: Includes built-in phish landing pages
  - Tenant Phish Landing Pages: Includes custom landing pages, such as with corporate branding

- Select ***End user notifications***
  - Global notifications: Includes built-in end user notifications send From `notification@attacksimulationtraining.com`
  - Tenant notifications: Includes custom end user notifications for branding and to set a different From address to an internal mailbox

- Who receives the simulated phishing message and on what schedule
  - All users* or specific users and groups (dynamic distribution groups are not supported)
    - *All users are all mailboxes (user and shared) and resources (room and equipment) in Exchange Online
  - Supported groups: _distribution_ and _mail-enabled security groups_

## The best practices of a simulation are:
- Target users: Include all users in your organization (Assuming all user mailboxes are licensed for Microsoft Defender for Office Plan 2)
  - If you want to target a specific department, you could import a CSV containing all the members of that department. To do this, run the following command in Graph PowerShell:
```
Get-MgUser -All -Property "Department,UserPrincipalname" | Where-Object {$_.Department -eq "DepartmentNameHere"} | Select-Object UserPrincipalname | Export-CSV -Path <PATH> -NoTypeInformation
```

- Exclude users: Import a CSV file that contains all your shared and room mailboxes (also specify your mail-enabled service accounts in this CSV file)
  - To export these RecipientTypes, you can run the following command in ExchangeOnline PowerShell:
```
Get-Mailbox -RecipientTypeDetails SharedMailbox, RoomMailbox -ResultSize Unlimited | Select-Object PrimarySmtpAddress | Export-CSV <PATH> -NoTypeInformation
```

> Note: You may see that the excluded users end up in the report as `FailedToDeliverEmail`, this is because the given user is blocked from signing in, such as you shared mailbox identities. This is normal behavior and you can filter them out in the report.

- Training: Select the Microsoft training experience and let Microsoft assign training courses and modules based on a user's previous simulation and training results learning paths.

## Progress of the attack simulation
We will take a deep dive into a ***Credential Harvest*** simulation, one of the several [social engineering techniques](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/attack-simulation-training-get-started?view=o365-worldwide#simulations) to choose from. Create the  ***Credential Harvest*** simulation using the steps provided by Microsoft to [simulate a phishing attack](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/attack-simulation-training-simulations) and select the global payload: `Keep Office 365 Password`. Upon completion, you should have a simulation in progress.
![IMAGE](/images/mdo-attack-simulation/mdo-attack-simulation-1.png)

The attack simulation then begins with users receiving credential phishing emails with the selected payload.

Payload:
![IMAGE](/images/mdo-attack-simulation/mdo-attack-simulation-2.png)

A user can click on the link, which creates an outbound connection to an adversary-in-the-middle (AiTM) phishing site (the connection does not raise an alert in Defender).

Login Page:
![IMAGE](/images/mdo-attack-simulation/mdo-attack-simulation-3.png)

If the user logs in, they will land on the phish landing page that provides a learning moment to the user after getting phished.

Phish Landing Page:
![IMAGE](/images/mdo-attack-simulation/mdo-attack-simulation-4.png)

Once the user clicked on the link and logged in, they received an email from `notification@attacksimulationtraining.com` for each action to complete a training course. 

![IMAGE](/images/mdo-attack-simulation/mdo-attack-simulation-5.png)
> ***NOTE:*** _You should remove the External tag for the notification email. To do so, run the cmdlet `Set-ExternalInOutlook -AllowList @{Add="attacksimulationtraining.com"}` by using the Exchange Online PowerShell module. You can also use **Tenant notifcations** to change the content or from address._


The link will take the user to the Defender portal to complete the courses at `https://security.microsoft.com/trainingassignments`.

![IMAGE](/images/mdo-attack-simulation/mdo-attack-simulation-6.png)

![IMAGE](/images/mdo-attack-simulation/mdo-attack-simulation-7.png)

The simulation report allows you to analyze how your users performed in the attack simulation.

![IMAGE](/images/mdo-attack-simulation/mdo-attack-simulation-8.png)

## Conlusion
Despite advanced security measures, phishing tactics continue to evolve, making it difficult to catch every attempt. Thereby user awareness is important because users play a critical role in identifying and avoiding potential threats.

## Reference
- [Attack simulation training documentation](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/attack-simulation-training-get-started)
- [Attack simulation training deployment considerations and FAQ](https://learn.microsoft.com/en-us/defender-office-365/attack-simulation-training-faq)