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

## Start using MDO's attack simulation training

### Step 1: Report message button
You should start by giving your users the ability to report email messages, which is also necessary for this attack simulation training. To do so, you can activate the:
- [Built-in Report button](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/submissions-user-reported-messages-custom-mailbox?view=o365-worldwide#use-the-microsoft-defender-portal-to-configure-user-reported-settings) (Outlook on the web)
- [Microsoft Report Message or Report Phishing add-ins](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/submissions-users-report-message-add-in-configure?view=o365-worldwide#get-the-report-message-add-in) (all Outlook platforms)

Reported messages appear in the [User Reported](https://security.microsoft.com/reportsubmission?viewid=user) section of the Submissions page, your reporting mailbox, and are visible in the simulation report.

### Step 2: Creating an attack simulation training
After the report button is implemented, you can begin creating an attack simulation training in the [Microsoft Defender Portal](https://security.microsoft.com/attacksimulator). You have the option to:
- [Simulate a phishing attack](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/attack-simulation-training-simulations)
  - These simulations test your security policies and practices, as well as train your employees to increase their awareness and decrease their susceptibility to attacks.
- [Using automated flows for Attack simulation](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/attack-simulation-training-simulation-automations)
  -  Creating a simulation automation is similar to creating an individual simulation, except for the ability to select multiple techniques, payloads, and the automation schedule.
- [Training campaigns for Attack simulation](https://learn.microsoft.com/en-us/defender-office-365/attack-simulation-training-training-campaigns)
  - Instead of creating and launching simulated phishing attacks that eventually lead to training, you can create and assign Training campaigns directly to users.

For now, we will take a deep dive into a ***Credential Harvest*** simulation, one of the several [social engineering techniques](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/attack-simulation-training-get-started?view=o365-worldwide#simulations) to choose from. Create the  ***Credential Harvest*** simulation using the steps provided by Microsoft to [simulate a phishing attack](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/attack-simulation-training-simulations). Upon completion, you should have a simulation in progress.

![IMAGE](/images/mdo-attack-simulation/mdo-attack-simulation-1.png)

The basic elements of a simulation are:
- Select a ***social engineering technique***, such as credential harvesting
- Select a ***payload*** _(phishing emails and web pages that you use to launch simulations)_
  - Global payloads: Includes built-in payloads, such as the `Keep Office 365 Password` payload
  - Tenant payloads: Contains custom payloads, such as a fake email from an executive with the company's official signature.
- Select a ***phish landing page*** _(provides a learning moment for the user after being phished)_
  - Global phish landing pages: Includes built-in phish landing pages
  - Tenant Phish Landing Pages: Includes custom landing pages, such as with corporate branding
- Select a ***Login Page*** _(phish web login page for credential harvesting and link in attachment techniques)_
  - Global login pages: Includes built-in login pages, such as the Microsoft login page.
  - Tenant login pages: Includes custom login pages, such as a custom Microsoft login page with corporate branding.
- Who receives the simulated phishing message and on what schedule
  - All users or specific users and groups (dynamic distribution groups are not supported)

### Step 3: Progress of the attack simulation
The attack simulation begins with users receiving credential phishing emails.

![IMAGE](/images/mdo-attack-simulation/mdo-attack-simulation-2.png)

A user can click on the link, which creates an outbound connection to an adversary-in-the-middle (AiTM) phishing site (the connection does not raise an alert in Defender).

![IMAGE](/images/mdo-attack-simulation/mdo-attack-simulation-3.png)

If the user logs in, they will land on the phish landing page that provides a learning moment to the user after getting phished.

![IMAGE](/images/mdo-attack-simulation/mdo-attack-simulation-4.png)

Once the user clicked on the link and logged in, they received an email from ```notification@attacksimulationtraining.com``` for each action to complete a training course. 

![IMAGE](/images/mdo-attack-simulation/mdo-attack-simulation-5.png)
> ***NOTE:*** _You may want to remove the External tag for the notification email. To do so, run the cmdlet ```Set-ExternalInOutlook -AllowList "attacksimulationtraining.com"``` by using the Exchange Online PowerShell module._


The link will take the user to the Defender portal to complete the courses.

![IMAGE](/images/mdo-attack-simulation/mdo-attack-simulation-6.png)

![IMAGE](/images/mdo-attack-simulation/mdo-attack-simulation-7.png)

The simulation report allows you to analyze how your users performed in the attack simulation.

![IMAGE](/images/mdo-attack-simulation/mdo-attack-simulation-8.png)

## Conlusion
Despite advanced security measures, phishing tactics continue to evolve, making it difficult to catch every attempt. Thereby user awareness is important because users play a critical role in identifying and avoiding potential threats.

## Reference
[Attack simulation training documentation](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/attack-simulation-training-get-started)