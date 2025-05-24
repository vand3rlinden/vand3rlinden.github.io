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
You should start by enabling your users to report email messages from the attack simulation using the built-in Report button, which is required for this training. If you haven’t already, [activate the built-in Report button](https://learn.microsoft.com/en-us/defender-office-365/submissions-user-reported-messages-custom-mailbox?view=o365-worldwide#use-the-microsoft-defender-portal-to-configure-user-reported-settings).

> If you are still using the Microsoft Report Message or Report Phishing add-ins, consider [transitioning to the built-in report button](https://learn.microsoft.com/en-us/defender-office-365/submissions-users-report-message-add-in-configure). These add-ins have security vulnerabilities that make them unsafe for your organization and will be deprecated over time. 

Reported messages appear in the [User Reported](https://security.microsoft.com/reportsubmission?viewid=user) section of the Submissions page, your reporting mailbox, and are visible in the simulation report.

### 3: Required permissions 
Required role: [Attack Simulation Administrator](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/permissions-reference#attack-simulation-administrator)

Users in this role can handle every facet of attack simulations, including creation, launch, scheduling, and result review. They have full access to all simulations within the tenant.

The role is available in the [Microsoft Defender](https://security.microsoft.com/emailandcollabpermissions) portal or in Entra ID (e.g. through PIM).

### 4: Remove external tagging for the default end user notifications address
Once the user clicked on the link and/or logged on to the phishing site, they received an email From `notification@attacksimulationtraining.com`.
Exclude this email address or domain from your external tagging configuration (_Exchange Online mail flow rules_ or _Exchange Online’s External Email Tagging Feature_). 
> You can also use Tenant notifications to change the From address to an internal address.

### 5: Turn on auditing
In order for Attack simulation training to have reporting capabilities, auditing needs to be enabled.
1. Connect to Exchange Online PowerShell
2. Enable Organization Customization by running: `Enable-OrganizationCustomization`
3. Then run the following PowerShell command to turn on auditing: `Set-AdminAuditLogConfig -UnifiedAuditLogIngestionEnabled $true`

## The basic elements of a simulation are:
- **Social Engineering Techniques** (e.g., credential harvesting)

- **Payloads**: Phishing emails and web pages that you use to launch simulations
  - **Global Payloads**: Includes built-in payloads, such as the `Keep Office 365 Password` payload
  - **Tenant Payloads**: Contains [custom payloads](https://learn.microsoft.com/en-us/defender-office-365/attack-simulation-training-payloads#create-payloads), such as a fake email from an executive with an official company signature. You can also use ChatGPT (or any other AI tool) to prompt a tenant payload to generate an HTML email for SharePoint Online document sharing, for example. A prompt can be: _Can you generate an HTML email template that looks like a Microsoft email to share a SharePoint document?_ This prompt leads to [this](https://vand3rlinden.com/images/mdo-attack-simulation/example-tenant-payload/sharepoint-documentshare.html) tenant payload, the HTML content can be copied and pasted into the `Configure Payload` section and change the Dynamic tags such as `${firstName}` and `${phishingUrl}`.

> Images that you use in tenant payloads may be [blocked](https://learn.microsoft.com/en-us/defender-office-365/attack-simulation-training-faq#why-are-images-in-simulation-messages-blocked-by-outlook) with a message that the sender is not in the Outlook Safe Senders list. This happens by default because Outlook is configured to block automatic image downloads in messages from the Internet.

- **Login Pages**: Phish web login page for credential harvesting and link in attachment techniques
  - **Global Login Pages**: Includes built-in login pages, such as the Microsoft login page
  - **Tenant Login Pages**: Includes custom login pages, such as a custom Microsoft login page with corporate branding

- **Phish Landing Pages**: Provides a learning moment for the user after being phished
  - **Global Phish Landing Pages**: Includes built-in phish landing pages
  - **Tenant Phish Landing Pages**: Includes custom landing pages, such as with corporate branding

- **End user notifications**
  - **Global notifications**: Includes built-in end user notifications send From `notification@attacksimulationtraining.com`
  - **Tenant notifications**: Includes custom end user notifications for branding and to set a different From address to an internal mailbox

> Images that you use in end user notifications may be [blocked](https://learn.microsoft.com/en-us/defender-office-365/attack-simulation-training-faq#why-are-images-in-simulation-messages-blocked-by-outlook) with a message that the sender is not in the Outlook Safe Senders list. This happens by default because Outlook is configured to block automatic image downloads in messages from the Internet.

- **[Target users](https://learn.microsoft.com/en-us/defender-office-365/attack-simulation-training-simulations#target-users)**: Who will receive the simulated phishing message and on what schedule
  - All users or specific users and groups
    - All users are all mailboxes (user and shared) and resources (room and equipment) in Exchange Online
  - Supported groups: **Microsoft 365** (static and dynamic), **distribution list** (static only) and **mail-enabled security** (static only).

## Creating an attack simulation training
After the requirements are set, you can begin creating an attack simulation in the [Microsoft Defender Portal](https://security.microsoft.com/attacksimulator). 

- You have the option to create:
  - [Simulate a phishing attack](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/attack-simulation-training-simulations)
    - These simulations test your security policies and practices, as well as train your employees to increase their awareness and decrease their susceptibility to attacks.
  - [Using automated flows for Attack simulation](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/attack-simulation-training-simulation-automations)
    - Creating a simulation automation is similar to creating an individual simulation, except for the ability to select multiple techniques, payloads, and the automation schedule.

- You also have the ability to create:
  - [Training campaigns for Attack simulation](https://learn.microsoft.com/en-us/defender-office-365/attack-simulation-training-training-campaigns)
    - Instead of creating and launching simulated phishing attacks that eventually lead to training, you can create and assign Training campaigns directly to users.
  - [Payload automations for Attack simulation training](https://learn.microsoft.com/en-us/defender-office-365/attack-simulation-training-payload-automations)
     - Payload automations, also known as payload harvesting, can collect data from real-world phishing attacks reported as phishing by users within your organization (and [verified by Microsoft as phishing](https://learn.microsoft.com/en-us/defender-office-365/attack-simulation-training-payload-automations#appendix)). You can define specific conditions to identify in phishing attacks, such as recipient details, social engineering techniques, or sender information.

## Automation vs. Individual simulations
### Automated Simulations
While Automated Simulations can be a great fit for your organization, there are some important considerations. For example, if you plan to automate a simulation for an entire year (which is the maximum duration for an automation schedule), you won’t be able to modify the content after the initial setup. This means you’ll be locked into the configuration choices made at the beginning.

### Individual Simulations 
Individual simulations, on the other hand, offer more flexibility, especially if you prefer to run separate monthly simulations. They allow for adjustments and customizations between campaigns.

### Payload Selection
#### Automated simulation - Payload Selection:
When setting up an automated simulation, you have two options for payload selection:
- **Manually select**: Up to 20 payloads, including both global and tenant-specific payloads.
- **Randomize**: Allowing Microsoft to select payloads automatically.

#### Individual Simulation - Payload Selection:
Only one payload can be configured per simulation.

> **NOTE**: It’s important to note that even in an automated simulation, only one payload is sent out per launch. All targeted users will receive the same payload (phishing email).

### Schedule Selection
#### Automated simulation - Schedule Selection:
When configuring the automation schedule, you have two options:  

- **Randomized Schedule**: Can start simulations with randomize send times. However, you cannot limit it to just one simulation per month. Instead, you can schedule up to 10 simulations per year on specific allowed days.
- **Fixed Schedule**: where simulations follow a weekly or monthly recurrence. However, they will always occur on a specific day of the week or month, and you cannot customize the send time.

Each automated simulation will be listed in the **Simulations tab** with a naming convention like:  
`AutomatedSimulation_PayloadName [Technique]_date`

#### Individual Simulation - Schedule Selection:
Each individual simulation can be configured to start at a different time. You can either create a new simulation each month or set up all 12 simulations at once, using varying dates and times to maintain user engagement and reduce predictability.

#### Simulation schedule comparison

| Action           | Automation randomized schedule | Automation fixed schedule | Individual schedule |
|-----------       |-----------                     |-----------                |-----------          |
| Random send time | Yes                            | No                        | Yes*                |
| Monthly  		     | No                             | Yes                       | Yes**               |
| Static send time | No                             | No                        | Yes                 |

- _*Each individual simulation can be configured to start at a different static time._
- _**You can create an individual simulation each month and schedule it up to two weeks before the launch date._

## The best practices of a simulation are:
- **Individual simulations**: Consider creating individual monthly simulations and plan a yearly schedule that varies the simulation dates and times each month to keep users engaged and reduce predictability. Keep in mind that all target users will receive the same payload. However, using individual monthly simulations gives you greater control and flexibility over content, timing, and targeting throughout the year.

- **Target users**: Include all **Microsoft Defender for Office Plan 2** users in your organization by using a dynamic Microsoft 365 group with the following syntax:
```
user.assignedPlans -any (assignedPlan.servicePlanId -eq "8e0c0a52-6a6c-4d40-8370-dd62790dcd70" -and assignedPlan.capabilityStatus -eq "Enabled")
```

> The servicePlanId `8e0c0a52-6a6c-4d40-8370-dd62790dcd70` corresponds to **Microsoft Defender for Office Plan 2** and remains the same across all tenants.

If you want to target a specific department, such as HR, you can use the following syntax:
```
(user.department -eq "HR")
```

> **NOTE**: Users may appear in the report as `FailedToDeliverEmail` because they are blocked from signing in. This is expected behavior, and you can filter them out in the report.

- **Training**: By enabling training during an attack simulation, Microsoft can assign courses and modules customized to the user’s previous simulation and training results through learning pathways. The training is based on user interactions, specifically whether they clicked and submitted their credentials. A compromised user may receive two training sessions. You can choose standalone training campaigns and disable training within the attack simulation. However, this approach will not be as adaptive as the learning pathways provided through training campaigns within an attack simulation.

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
> **NOTE**: You should remove the External tag for the notification email. To do so, run the cmdlet `Set-ExternalInOutlook -AllowList @{Add="attacksimulationtraining.com"}` by using the Exchange Online PowerShell module. You can also use **Tenant notifcations** to change the content or from address.

The link will take the user to the Defender portal to complete the courses at `https://security.microsoft.com/trainingassignments`.

![IMAGE](/images/mdo-attack-simulation/mdo-attack-simulation-6.png)

![IMAGE](/images/mdo-attack-simulation/mdo-attack-simulation-7.png)

The simulation report allows you to analyze how your users performed in the attack simulation.

![IMAGE](/images/mdo-attack-simulation/mdo-attack-simulation-8.png)

## Conlusion
Despite advanced security measures, phishing tactics continue to evolve, making it difficult to catch every attempt. Thereby user awareness is important because users play a critical role in identifying and avoiding potential threats.

## Reference
- [Attack simulation training documentation](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/attack-simulation-training-get-started)
- [Supported target users](https://learn.microsoft.com/en-us/defender-office-365/attack-simulation-training-simulations#target-users)
- [Attack simulation training deployment considerations and FAQ](https://learn.microsoft.com/en-us/defender-office-365/attack-simulation-training-faq)
- [Turn on auditing](https://learn.microsoft.com/en-us/purview/audit-log-enable-disable?view=o365-worldwide&tabs=microsoft-purview-portal#turn-on-auditing)
- [Automation Schedule details](https://learn.microsoft.com/en-us/defender-office-365/attack-simulation-training-simulation-automations#schedule-details)
- [Manage rules for dynamic membership groups in Microsoft Entra ID](https://learn.microsoft.com/en-us/entra/identity/users/groups-dynamic-membership)
- [Product names and service plan identifiers for licensing](https://learn.microsoft.com/en-us/entra/identity/users/licensing-service-plan-reference)