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
You should start by giving your users the ability to report email messages, which is also necessary for this attack simulation training. To do so, you can activate the [built-in Report button](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/submissions-user-reported-messages-custom-mailbox?view=o365-worldwide#use-the-microsoft-defender-portal-to-configure-user-reported-settings) in Outlook on the web using the [User reported settings](https://security.microsoft.com/securitysettings/userSubmission) or the [Microsoft Report Message or Report Phishing add-ins](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/submissions-users-report-message-add-in-configure?view=o365-worldwide#get-the-report-message-add-in) in virtually all Outlook platforms to report email messages. 

Reported messages appear in the [User Reported](https://security.microsoft.com/reportsubmission?viewid=user) section of the Submissions page, your reporting mailbox, and are visible in the simulation report.

### Step 2: Creating an attack simulation training
After the report button is implemented, you can begin creating an attack simulation training in the [Microsoft Defender Portal](https://security.microsoft.com/attacksimulator). You have the option to [simulate](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/attack-simulation-training-simulations) or use [automated flows](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/attack-simulation-training-simulation-automations). 

For now, we will take a deep dive into a ***Credential Harvest*** simulation, one of the several [social engineering techniques](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/attack-simulation-training-get-started?view=o365-worldwide#simulations) to choose from. Create the  ***Credential Harvest*** simulation using the steps provided by Microsoft to [simulate](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/attack-simulation-training-simulations) an attack. Upon completion, you should have a simulation in progress.

![IMAGE](/images/mdo-attack-simulation/mdo-attack-simulation-1.png)


### Step 3: Progress of the attack simulation
The attack simulation begins with users receiving credential phishing emails.

![IMAGE](/images/mdo-attack-simulation/mdo-attack-simulation-2.png)

A user can click on the link, which creates an outbound connection to an adversary-in-the-middle (AiTM) phishing site (the connection does not raise an alert in Defender).

![IMAGE](/images/mdo-attack-simulation/mdo-attack-simulation-3.png)

If the user logs in, they will land on the phish landing page that provides a learning moment to the user after getting phished.

![IMAGE](/images/mdo-attack-simulation/mdo-attack-simulation-4.png)

Once the user clicked on the link and logged in, they received an email from ```notification@attacksimulationtraining.com``` for each action to complete a training course. 

> ***NOTE:*** _You may want to remove the External tag for the domain ```attacksimulationtraining.com```. To do so, run the cmdlet ```Set-ExternalInOutlook -AllowList "attacksimulationtraining.com"``` by using the Exchange Online PowerShell module._

![IMAGE](/images/mdo-attack-simulation/mdo-attack-simulation-5.png)

The link will take the user to the Defender portal to complete the courses.

![IMAGE](/images/mdo-attack-simulation/mdo-attack-simulation-6.png)

![IMAGE](/images/mdo-attack-simulation/mdo-attack-simulation-7.png)

The simulation report allows you to analyze how your users performed in the attack simulation.

![IMAGE](/images/mdo-attack-simulation/mdo-attack-simulation-8.png)

## Conlusion
Despite advanced security measures, phishing tactics continue to evolve, making it difficult to catch every attempt. Thereby user awareness is important because users play a critical role in identifying and avoiding potential threats.

## Reference
[Attack simulation training documentation](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/attack-simulation-training-get-started)