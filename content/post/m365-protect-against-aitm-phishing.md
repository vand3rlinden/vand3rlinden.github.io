---
title: 'Microsoft 365: Protect your environment against AiTM phishing attacks'
date: 2025-03-23T22:52:19+01:00
draft: false
categories: ["Microsoft 365", "email"]
cover: 
  image: /images/m365-protect-against-aitm-phishing/m365-protect-against-aitm-phishing-front.png
---

> _In this blog post, we’ll walk through practical strategies to reduce the likelihood and impact of successful AiTM phishing attacks in Microsoft 365._

In today’s threat landscape, adversary-in-the-middle (AiTM) phishing attacks have emerged as a particularly dangerous tactic. These attacks bypass traditional defenses by placing a malicious proxy between the user and the legitimate service, capturing credentials and session tokens in real time. Once attackers have session tokens, they can bypass even multi-factor authentication (MFA), making AiTM attacks especially difficult to defend against.

While there is no single configuration to fully prevent AiTM phishing attacks in Microsoft 365 environments, a layered defense strategy can significantly reduce risk. This includes a combination of user education, strong authentication methods, and granular Conditional Access policies.

## Path of an AiTM Phishing Attack
The attack typically follows these steps:

1. **Phishing Email Received**  
   The user receives a phishing email that may closely resemble a legitimate message.  

   ![IMAGE](/images/m365-protect-against-aitm-phishing/m365-protect-against-aitm-phishing-1.png)

2. **User Clicks the Link**  
   If the user doesn't recognize any red flags in the email and clicks the link, they are redirected to an **AiTM phishing site** that mimics a real login page.  

   ![IMAGE](/images/m365-protect-against-aitm-phishing/m365-protect-against-aitm-phishing-2.png)

3. **Credential Harvesting**  
   If the user also fails to notice red flags in the URL and enters their credentials, the attacker is able to intercept and steal **authorization tokens**, giving them access to the user’s session, even with MFA enabled.  

   ![IMAGE](/images/m365-protect-against-aitm-phishing/m365-protect-against-aitm-phishing-3.png)

This illustrates how AiTM phishing attacks bypass traditional authentication defenses by hijacking active sessions.

## Train Your Users – The First Line of Defense
Human error remains one of the leading causes of successful phishing attacks. No technical control can completely compensate for an untrained or unaware user.

[Attack simulation training using Microsoft Defender for Office 365](https://vand3rlinden.com/post/mdo-attack-simulation/) or other platforms, such as **Knowbe4**, allows organizations to run real-world phishing simulations and educate users on how to spot suspicious links, spoofed login pages, or unexpected MFA prompts.

## Phishing-Resistant MFA
Not all MFA methods are created equal. Most AiTM attacks work because they can intercept standard MFA flows like SMS or app-based codes. To defend effectively, organizations should enforce phishing-resistant MFA methods such as: FIDO2 security keys, these hardware-based tokens that are inherently resistant to AiTM attacks. This is because FIDO2 authentication is bound to the specific origin (i.e., the legitimate website or application). A sign-in attempt on a different surface, such as a phishing site, will fail if the origin does not match. This effectively prevents credentials from being reused on malicious or unauthorized platforms.

[Enable passkeys (FIDO2) for your organization](https://learn.microsoft.com/en-us/entra/identity/authentication/how-to-enable-passkey-fido2)

## Harden Conditional Access Policies
### Enforce Phishing-Resistant MFA
Start by securing admin accounts first. Accounts with highly privileged administrative rights are frequent targets for attackers. Enforcing phishing-resistant MFA on these accounts is a straightforward and effective way to significantly reduce the risk of compromise.

[Policy Implementation](https://learn.microsoft.com/en-us/entra/identity/conditional-access/policy-admin-phish-resistant-mfa)

Tip: When using eligible roles in PIM (Privileged Identity Management), it’s recommended to include an Entra ID group, preferably a dynamic group based on your admin naming convention (e.g., ‘adm’), instead of include roles directly to the Conditional Access Policy. Also include an Entra ID group with the standard (non-privileged) accounts of the admins, as attackers can often find information online (e.g., LinkedIn) and target them with phishing emails sent to their regular user accounts.

### Require compliant devices
To ensure that only trusted and compliant devices can access corporate resources, you may want to start by applying these restrictions to admin accounts. As mentioned earlier in this blog, accounts with administrative privileges are prime targets for attackers. Requiring these users to operate only from compliant or Microsoft Entra hybrid joined devices helps reduce the risk of compromise and limits potential exposure.

Why it helps: Blocks attackers using stolen credentials via AiTM phishing, even if they bypass MFA, because their device won’t meet compliance requirements.

[Policy Implementation](https://learn.microsoft.com/en-us/entra/identity/conditional-access/policy-alt-admin-device-compliand-hybrid)

Tip: When using eligible roles in PIM (Privileged Identity Management), it’s recommended to include an Entra ID group, preferably a dynamic group based on your admin naming convention (e.g., ‘adm’), instead of include roles directly to the Conditional Access Policy. Also include an Entra ID group with the standard (non-privileged) accounts of the admins, as attackers can often find information online (e.g., LinkedIn) and target them with phishing emails sent to their regular user accounts.

### Restrict Access by Network Location
Implement location-based Conditional Access policies to limit access to specific countries or trusted IP ranges.
- Example: Allow sign-ins only from countries where your organization operates.

Why it helps: Many AiTM attacks originate from foreign IP addresses. Restricting logins to known geographies can immediately block these attempts.

[Policy Implementation](https://learn.microsoft.com/en-us/entra/identity/conditional-access/policy-block-by-location)

### Enable Risk Policies
Allows you to define automatic responses, such as enforcing multi-factor authentication (MFA), blocking access, or requiring a password reset when certain risk thresholds are met.

- User risk policy: Automatically respond when a user’s account is compromised.
- Sign-in risk policy: Require MFA or block access when risk is high.

[Policy Implementation](https://learn.microsoft.com/en-us/entra/id-protection/howto-identity-protection-configure-risk-policies)

### Require re-authentication and disable browser persistence
Protect user access on unmanaged devices by configuring browser sessions to sign out automatically when the browser is closed and setting the sign-in frequency to 1 hour. This reduces the risk of unauthorized access by ensuring users must re-authenticate regularly and session data is not retained.

Why it helps:
- Sign-in frequency: By default, users in Microsoft Entra ID can remain signed in to applications like Microsoft 365 for up to 90 days without reauthenticating. Enforcing a timeout for MFA helps ensure that sessions are not kept alive indefinitely, enhancing security by requiring users to re-authenticate periodically.

- Persistent browser session - Never persistent: The concept of a non-persistent browser session is designed to prevent users from remaining signed in after closing their browser. Disabling browser persistent sessions helps protect against drive-by attacks by preventing the creation and storage of session cookies, leaving nothing behind for an attacker to steal. While this does not prevent AiTM attacks, since AiTM targets active sessions rather than stored session data, non-persistent sessions still contribute to a layered defense. They help by limiting the time an attacker can use stolen session tokens, enforcing more frequent reauthentication.

[Policy Implementation](https://learn.microsoft.com/en-us/entra/identity/conditional-access/policy-all-users-persistent-browser)

### Require token protection for sign-in sessions
Microsoft is rolling out token protection to combat token theft. As of now, it’s supported for **SharePoint Online** and **Exchange Online**.

How token protection works:
- Token protection binds access tokens to the device they were issued on.
- If the token is stolen and used elsewhere, it will fail authorization even if the session was authenticated.

> Important: AiTM phishing kits can still successfully steal the token and initiate a session. However, with token protection, authorization will fail unless the token is used from the original device. This makes token theft less valuable for attackers and adds another hurdle for them to overcome.

[Policy Implementation - preview](https://learn.microsoft.com/en-us/entra/identity/conditional-access/concept-token-protection)

## Harden Microsoft Defender
### Block Risky Web Categories with Microsoft Defender for Endpoint
AiTM phishing kits often use newly registered domains and parked domains to host their proxy phishing sites. These domains are frequently created just hours before an attack to avoid detection by traditional domain reputation engines.

To proactively block access to such sites, you can configure web content filtering in Microsoft Defender for Endpoint:
- In the Microsoft 365 Defender portal, go to Settings > Endpoints > Advanced features > Enable: Web content filtering.
- Once enabled, navigate to: Endpoints > Web content filtering
- Create a policy that blocks the following categories:
  - Newly Registered Domains
  - Parked Domains

These categories are frequently leveraged by threat actors to bypass email filters and trick users into entering their credentials.

Why it helps: Blocking access to these domains prevents users from ever reaching the phishing page, even if they click a link in a legitimate-looking email.

This is especially effective when combined with [Defender SmartScreen](https://learn.microsoft.com/en-us/windows/security/operating-system-security/virus-and-threat-protection/microsoft-defender-smartscreen/) and [Network Protection](https://learn.microsoft.com/en-us/defender-endpoint/enable-network-protection) policies to stop outbound access to malicious or suspicious domains.

### Configure Automatic Attack Disruption in Microsoft Defender XDR
Attackers move fast, sometimes within minutes of obtaining access. That’s why it’s crucial to respond faster.

Microsoft Defender XDR now offers automatic attack disruption, which can take immediate action when high-confidence signals of compromise are detected. This capability works across Defender for Endpoint, Defender for Identity, Defender for Office 365, and Defender for Cloud Apps.

What it does:
- Automatically isolates compromised user accounts or devices.
- Disrupts lateral movement and command-and-control activity.
- Uses AI to correlate telemetry across workloads and identify active attacks.

[Configure automatic attack disruption in Microsoft Defender XDR](https://learn.microsoft.com/en-us/defender-xdr/configure-attack-disruption)

Why it helps: Even if AiTM phishing is successful and the attacker gains a foothold, automatic attack disruption can contain the threat before significant damage occurs.

## Summary
AiTM phishing attacks are hard to stop with a single control. By combining phishing-resistant MFA, Conditional Access policies, risk-based access controls, token protection, web filtering, user training, and automatic attack disruption in Microsoft Defender XDR, you build a strong, layered defense. The goal isn’t just to prevent attacks, but to detect, contain, and respond before damage is done. Defense in depth is your best strategy in the evolving threat landscape.