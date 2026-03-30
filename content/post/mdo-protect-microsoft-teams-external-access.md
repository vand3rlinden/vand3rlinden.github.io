---
title: 'Microsoft Defender for Office 365: Protecting external Teams access'
date: 2026-03-29T18:01:25+02:00
draft: false
categories: ["Microsoft Defender for Office 365"]
cover: 
  image: /images/mdo-protect-microsoft-teams-external-access/mdo-protect-microsoft-teams-external-access-front.png
---

In the wild, i see a lot of impersonated **IT Support** domains contacting users via Teams chat. Think of MOERA domains like `internalsupport.onmicrosoft.com`, starting a chat with a display name like `Corporate IT Service (Internal)` from `corporateit@internalsupport.onmicrosoft.com`. We all know that unaware users can fall for this, and the worst case scenario is that a user starts a screen sharing session, leaking internal information (data exfiltration), installing malware, or gets phished by passing account credentials.

[User awareness training](https://vand3rlinden.com/post/mdo-attack-simulation/) is always a good practice, however as IT Engineers it is our duty to protect users within the tenant in a way that aligns with business needs. In this blog I share my best practices for external Teams access. Keep in mind that these best practices will probably not fit every business, for example businesses that work with a lot of external users. However, this post will give you an idea of the different capabilities when it comes to external Teams access and how you can protect and monitor your environment.

## Step 1: Follow up Microsoft Secure Score recommendation for Teams
As with all products, it is recommended to work through the Microsoft Secure Score recommendations (https://security.microsoft.com/exposure-secure-score). For Teams, the following recommendations were listed at the time of writing:

- **Configure which users are allowed to present in Teams meetings**
  - Teams admin center > Meetings > Meeting Policies > each group/direct policy > Content sharing section > **Who can present** > **Only organizers and co-organizers** 
  - This setting controls who can be a presenter in Teams meetings. Organizers and co-organizers can change this when setting up a Teams meeting. This reduces the risk of unwanted or inappropriate content being shared.

- **Limit external participants from having control in a Teams meeting**
  - Teams admin center > Meetings > Meeting Policies > each group/direct policy > Content sharing section > **External participants can give or request control** >  **Off**
  - This setting controls whether external participants, anonymous users and guests can be given control of, or request control of, a screen being shared by someone in your organization during a Teams meeting.

- **Only invited users should be automatically admitted to Teams meetings**
  - Teams admin center > Meetings > Meeting Policies > each group/direct policy > Meeting join and lobby section > **Who can bypass the lobby** > **People who were invited**
  - This setting controls who can join a meeting directly and who must wait in the lobby until admitted, organizers and co-organizers can change this when setting up a Teams meeting.

- **Restrict anonymous users from joining meetings**
  - Teams admin center > Meetings > each group/direct policy > **Anonymous users can join a meeting** > **Off**
  - When this setting is on, anyone can join Teams meetings, including users who are not logged in to Teams with a work or school account.

- **Restrict anonymous users from starting Teams meetings**
  - Teams admin center > Meetings > Meeting Policies > each group/direct policy > Meeting join and lobby section > **Anonymous users and dial-in callers can start a meeting** > **Off**
  - When this setting is on, anonymous users and dial-in callers can start a meeting without anyone else in attendance, including users who are not logged in to Teams with a work or school account.

- **Restrict dial-in users from bypassing a meeting lobby**
  - Teams admin center > Meetings > Meeting Policies > each group/direct policy > Meeting join and lobby section > **People dialing in can bypass the lobby** > **Off**
  - When this setting is on, dial-in users skip the lobby and join the meeting directly.

## Step 2: Design an External access strategy 
Within Microsoft Teams you have a few options to manage external domains to **call, chat, and set up meetings**.

Not every configuring scenario will fit every tenant, so it is worth thinking about your strategy here. The following scenarios are available in **Teams admin center > External collaboration > External access**:

- **Allow all external domains:** The default setting in Teams, it lets users in your organization find, call, chat, and set up meetings with people outside of your organization in any domain **(not recommended)**.
- **Allow only specific external domains:** By adding domains to an allow list, you limit external access to only the allowed domains, once you set up a list of allowed domains, all other domains will be blocked **(recommended, but can become unmanageable if your tenant has a lot of external Teams communication)**.
- **Block specific domains:** By adding domains to a block list, you can communicate with all external domains except the ones you have blocked, once you set up a list of blocked domains, all other domains are allowed **(preferred if an allow list does not fit your organization's needs)**.
- **Block all external domains:** Prevents users in your organization from finding, calling, chatting, and setting up meetings with people outside of your organization in any domain.

> **IMPORTANT:** People from blocked domains can still join meetings anonymously if anonymous access is allowed, but we already mitigated this in step 1.

For most organizations the **Block specific domains** scenario fits best. In practice, most malicious activity comes from impersonated MOERA domains (`*.onmicrosoft.com`), so you may want to add this as your first block entry. Note that by default, blocking a domain does not block its subdomains. For example, if you block `onmicrosoft.com`, `internalsupport.onmicrosoft.com` is not blocked. If you want to block all subdomains, you can use the `Set-CsTenantFederationConfiguration` PowerShell cmdlet with the `-BlockAllSubdomains` parameter.

```
Set-CsTenantFederationConfiguration -BlockAllSubdomains $True
```

> **CAUTION**: Before proceeding, please review your existing `*.onmicrosoft.com` Teams communication. If you have already connected the Microsoft 365 workload as in step 3, use the `CloudAppEvents` table. Otherwise, check the `OfficeActivity` table from the Microsoft 365 Sentinel connector.

- `CloudAppEvents` (Advanced Hunting)
```
CloudAppEvents
| where Timestamp > ago(30d)
| where AccountId endswith "onmicrosoft.com"
| where Application =~ "Microsoft Teams"
| project-reorder Timestamp, AccountId
| sort by Timestamp desc
```

- `OfficeActivity` (Sentinel)
```
OfficeActivity
| where TimeGenerated > ago(60d)
// Search for onmicrosoft[.]com domains that do not contains your Entra ID guest users (guest_user[.]com#EXT#@YourMOERA.onmicrosoft[.]com)
| where UserId endswith 'onmicrosoft.com' and UserId !contains "#EXT#"  
// This #EXT# UserId is only used as a UserId in Entra ID and not for Teams communication
| where OfficeWorkload =~ 'MicrosoftTeams'
| where Operation contains 'Message'
| project-reorder TimeGenerated, UserId
| sort by TimeGenerated desc
```

> **IMPORTANT:** For every scenario, make sure to enable the option **Allow my security team to manage blocked domains and blocked users**. This will block and delete all existing and incoming messages from flagged senders when your SOC team adds a Teams sender to the Tenant Allow/Block List.

## Step 3: Connect Microsoft 365 workload to Defender for CloudApps
Connecting apps to MDA such as [Microsoft 365 workloads](https://learn.microsoft.com/en-us/defender-cloud-apps/protect-office-365#connect-office-365-to-microsoft-cloud-app-securit) gives you visibility and control over your environment, and improves insight into user activities through the `CloudAppEvents` table in Advanced Hunting. When connected, and if you have not set a block on `onmicrosoft.com` yet, you could query or build a custom detection rule to alert when one of your users starts a screen sharing session with an `onmicrosoft.com` domain.

Example query:

```
CloudAppEvents
| where Timestamp > ago(30d)
| where AccountId endswith "onmicrosoft.com"
| extend Artifacts = parse_json(RawEventData).ArtifactsShared
| where isnotnull(Artifacts)
| mv-expand Artifact = Artifacts
| where Artifact.ArtifactSharedName == 'screenShared'
| sort by Timestamp desc
```

## References
- [Manage external meetings](https://learn.microsoft.com/en-us/microsoftteams/trusted-organizations-external-meetings-chat)
- [Manage anonymous participant access to Teams meetings](https://learn.microsoft.com/en-us/microsoftteams/anonymous-users-in-meetings)
- [Microsoft Teams and the Tenant Allow/Block List](https://learn.microsoft.com/en-us/defender-office-365/tenant-allow-block-list-teams-domains-configure)
- [How Defender for Cloud Apps helps protect your Microsoft 365 environment](https://learn.microsoft.com/en-us/defender-cloud-apps/protect-office-365#connect-office-365-to-microsoft-cloud-app-security)
- [CloudAppEvents](https://learn.microsoft.com/en-us/defender-xdr/advanced-hunting-cloudappevents-table)