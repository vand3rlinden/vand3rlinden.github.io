---
title: 'Microsoft 365: Blocking device code flow abuse'
date: 2026-08-18T13:05:56+02:00
draft: false
categories: ["Microsoft 365", "email"]
cover: 
  image: /images/m365-blocking-dcf-abuse/m365-blocking-dcf-abuse-front.png
---

I see device code flow phishing pop up more and more. With device code flow phishing an attacker starts a device code sign in on their own machine, then tries to social engineer a user into completing it through a phishing email, phone call (vishing) or message. The user is asked to open a link, usually the real `microsoft.com/devicelogin` page, and paste in a code the attacker just generated. The user pastes the code and signs in, and the own session of the attacker picks up the resulting token. No password needed, no malicious page involved, just a user doing exactly what they were told.

This is what makes device code flow phishing hard to catch, and needs a security control at the tenant level. 

The page where the user enters the device code is a legitimate Microsoft URL. There is no fake login form, no lookalike domain, nothing that a **check the URL** reminder would catch.

![IMAGE](/images/m365-blocking-dcf-abuse/dcf-example.png)

## What device code flow actually is
Device code flow (OAuth 2.0 device authorization grant) exists for devices without a proper browser or keyboard, think smart TVs, some IoT devices, or CLI tools like older versions of Azure CLI. The device shows a short code and a URL, the user opens that URL on a second device such as their phone, signs in, enters the code, and the authentication state gets transferred back to the original device.

## Step 1: Audit current device code flow usage
Before blocking anything, check whether device code flow is actually used in your tenant, you will want this to build a Conditional Access exclusion list and to explain to the business why this control matters.

Check this in Microsoft Entra sign in logs, or with the following query against the `SigninLogs` table if you have the **Microsoft Entra ID connector** set up in Sentinel.

```kusto
SigninLogs
| where TimeGenerated > ago(30d)
//| where UserPrincipalName =~ "UserPrincipalName"
| where AuthenticationProtocol =~ 'deviceCode' or OriginalTransferMethod =~ 'deviceCodeFlow'
| project TimeGenerated, UserPrincipalName, AppDisplayName, IPAddress, Location, DeviceDetail, ConditionalAccessStatus, ConditionalAccessPolicies, AuthenticationProtocol, OriginalTransferMethod
| sort by TimeGenerated desc
```

> **Note:** The `AuthenticationProtocol` column shows device code flow for the sign in event itself, and `OriginalTransferMethod` shows whether the session was originally established through device code flow, even if a later token refresh does not show device code as the protocol anymore. Microsoft tracks this as "protocol tracking", a session that started with device code flow stays tagged as such through later requests. Use both fields, relying on `AuthenticationProtocol` alone can make you miss sessions that are still tied to an earlier device code sign in.

Whatever shows up here, legacy tooling, break glass procedures, CLI usage, becomes your exclusion list in the next step.

## Step 2: Block device code flow with Conditional Access
> **IMPORTANT:** Get as close as possible to a full block. Microsoft's own recommendation is to only allow device code flow for well documented, unavoidable use cases like legacy tooling that genuinely cannot be updated.

Setting: **Entra ID > Conditional Access > Policies > New policy**

| Assignments | |
| --- | --- |
| Users | **Include:** All Users **Exclude:** BTG accounts, allowed users such as meeting rooms and certain admins |
| Target resources | **Include:** All resources **Exclude:** - |
| Conditions > Authentication flows | Transfer methods: Device code flow |

| Access controls | |
| --- | --- |
| Grant | Block access |

> **CAUTION:** If your tenant uses device code flow for device registration, exclude the Device Registration Service resource from Target resources. Since September 2024 Microsoft enforces authentication flow policies on that resource too when a policy targets all resources, and a tenant wide block without this exclusion can break device registration for legitimate scenarios like Teams devices. But again, get as close as possible to a full block for device code flow sign in

## Step 3: Start in report only, then flip it on
Set **Enable policy** to **Report-only** first, not **On**. Give it a few days, then review the results through policy impact or report-only mode alongside the sign in logs from step 1. Confirm nothing legitimate gets caught, confirm your exclusion list is complete, and only then move the **Enable policy** toggle from **Report-only** to **On**.

Keep monitoring after that. Review the exclusion group regularly, and treat any device code flow sign in outside of it as something worth looking into.

## References
- [Block authentication flows with Conditional Access policy](https://learn.microsoft.com/entra/identity/conditional-access/policy-block-authentication-flows)
- [Microsoft identity platform and the OAuth 2.0 device authorization grant flow](https://learn.microsoft.com/entra/identity-platform/v2-oauth2-device-code)
- [Restrict device code flow for Microsoft Teams devices with Conditional Access](https://learn.microsoft.com/entra/identity/conditional-access/policy-teams-devices-device-code-flow)
