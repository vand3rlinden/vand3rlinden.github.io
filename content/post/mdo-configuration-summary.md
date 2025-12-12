---
title: 'Microsoft Defender for Office 365: Configuration Summary'
date: 2025-11-23T00:17:31+01:00
draft: false
categories: ["Microsoft Defender for Office 365"]
cover: 
  image: /images/mdo-configuration-summary/mdo-configuration-summary-front.png
---

In this post, I will explain how to configure **Microsoft Defender for Office 365** in your tenant. Since I have already covered most of these settings in my earlier blogs, I thought it was time to combine all the information into one post. I will link to my previous articles, so you can use this post as a summary to help strengthen your Microsoft Defender for Office 365 configuration, `@` the speed of email!

> **NOTE**: These recommendations mostly align with the ***strict*** recommendations of the **Configuration Analyzer** and the **Microsoft Secure Score** in XDR. 

## Configuration Checklist
### 1: Setup Preset Security Policies
If your organization does not have the budget or time, Preset Security Policies ensure you can maintain a balance between keeping harmful content away from users and avoiding unnecessary disruptions. These policies can be configured within five minutes with just a few clicks. Unlike custom policies, which are highly configurable, most settings in Preset Security Policies cannot be modified.

> **NOTE**: If you choose to use Preset Security Policies; you can skip to **Step 6** of this check list.

The following Preset Security Policies are available:

Standard Preset Security Policy - A baseline protection profile that provides:
- Exchange Online Protection (inbound anti-spam, anti-malware, and anti-phishing)
- Microsoft Defender for Office 365 protection (Safe Attachments and Safe Links)

Strict Preset Security Policy - A more aggressive protection profile for selected users, such as high-value or priority accounts. It provides:
- Exchange Online Protection (inbound anti-spam, anti-malware, and anti-phishing)
- Microsoft Defender for Office 365 protection (Safe Attachments and Safe Links)

Built-in Protection Preset Security Policy - Default Safe Attachments and Safe Links policies for Defender for Office 365
- Provides baseline Microsoft 365 protection applied automatically to all users, safeguarding them against malicious links and attachments without requiring configuration.

Reference:
- [Configuration in Microsoft Defender XDR](https://security.microsoft.com/presetSecurityPolicies)
- [Policy settings in preset security policies](https://learn.microsoft.com/en-us/defender-office-365/preset-security-policies#policy-settings-in-preset-security-policies)

### 2: Configure Exchange Online Protection Threat policies
- [Anti-phishing policies](https://vand3rlinden.com/post/mdo-anti-phishing-policies/)
    - Anti-phishing policies provides control over incoming phishing emails, for instance, in cases where someone may attempt to impersonate your CEO or send messages from a domain that closely resembles yours.
- [Anti-spam policies](https://vand3rlinden.com/post/mdo-anti-spam-policies/)
    - Anti-spam policies give you control over both inbound and outbound email in Exchange Online. Within these policies, you can configure settings such as completely blocking automatic forwarding for outbound mail.
- [Anti-malware policies](https://vand3rlinden.com/post/mdo-anti-malware-policies/)
    - Anti-malware policies provides an additional layer of protection by blocking specific file types for both inbound and outbound mail traffic.

### 3: Configure Microsoft Defender for Office 365 Protection Threat policies
- [Safe Attachments policies](https://vand3rlinden.com/post/mdo-safeattachments-policies/)
	- Safe Attachments scans and evaluates attachments for malicious content before delivering messages to recipients. It also has the ability to protect files in SharePoint, OneDrive, and Microsoft Teams.
- [Safe Links policies](https://vand3rlinden.com/post/mdo-safelinks-policies/)
	- Safe Links scans URLs in incoming messages and checks them for malicious content at the time they are clicked. Safe Links protection works in Email/Outlook, Microsoft Teams, and Office apps.

### 4: Using Custom VS. The Default Always On Policies
The threat policies **Anti-phishing**, **Anti-spam**, and **Anti-malware** comes with default **Always On policies**. These policies are automatically used for every recipient domain in your tenant. You can adjust the settings of these **Always On policies**, for example by using the **strict** or **standard** recommendations from the **Configuration Analyzer**.

I often see many new **Custom Default policies** created, that are set for all recipient domains. This is not a good practice, because it becomes a problem when an IT admin adds a new recipient domain but forgets to update the custom threat policies. Users in the new recipient domain may then have weaker protection, which creates a security gap. It is better to use and adjust the built-in **Always On policies**, because they automatically apply to all recipient domains. 

I am not saying you should never use custom threat policies. For example, you can set the **Always On policies** to the **standard** recommendations, and then create a custom policy with **strict** recommendations for your key users, or users that are allowed to forward in a **Custom Outbound Anti-spam policy**.

The above recommandation is not for **Safe Attachments** and **Safe Links** policies, because there is **not** an **Always On default policy** you can adjust _(only the built-in protection preset security policy is available, and it cannot be adjusted)_. However, for these policies you can create a **Custom policy** based on the **standard** or **strict** recommendations, and make sure all recipient domains are **always** included.

> Recommended name convention for **Custom policies**: `<TENANTNAME> - <PURPOSE> - <POLICY>` (e.g., `TENANTNAME - Strict - Anti-Phishing policy`, `TENANTNAME - Allow Forward - Outbound Anti-spam policy`)
	
### 5: Use custom release request quarantine policies (instead of self-release)
- [Quarantine policies](https://vand3rlinden.com/post/mdo-quarantine-policies/)
	- If you are **not** using Preset Security Policies, you can create a quarantine policy to customize the user experience for quarantined messages. Quarantine policies give you more control over the quarantine for your end users and allow you to decide which quarantined items they are allowed to release.

### 6: Hardening Microsoft Defender for Office 365's DKIM and DMARC configuration
- [Hardening DKIM and DMARC configuration](https://vand3rlinden.com/post/mdo-hardening-dkim-dmarc-config/)
	- Improve email security in Microsoft Defender for Office 365 by fine-tuning DKIM, configuring DMARC for the MOERA domain, and blocking inbound DMARC failures from reaching user inboxes.

### 7: Reject Direct Send in Exchange Online
- [Exchange Online: Reject Direct Send](https://vand3rlinden.com/post/exo-reject-direct-send/)
	- Direct Send is a method used to send emails directly to Exchange Online hosted mailboxes from on-premises devices, applications, or third-party cloud services, using the MX record endpoint of your accepted domain in Exchange Online.

### 8: DNS configuration
- **Inbound email**:
    - [MTA-STS Policy](https://vand3rlinden.com/post/mta-sts-explained/)
		- MTA-STS is a security mechanism that allows the sending (outbound) mail server to enforce the use of HTTPS secured policies published in your DNS. This ensures that TLS connections between the sending (outbound) mail server and your mail server (inbound) are both encrypted and valid.
	- [Configure inbound SMTP DANE with DNSSEC in Exchange Online](https://vand3rlinden.com/post/exo-inbound-smtp-dane-dnssec/)
		- Functionally similar to MTA-STS, however, SMTP DANE uses DNSSEC to allow the sending (outbound) mail server to verify the TLS certificate of your mail server (inbound).
	
> **NOTE**: Neither SMTP DANE nor MTA-STS is universally **better**. SMTP DANE provides stronger security, but requires DNSSEC, and not every DNS provider supports DNSSEC yet. MTA-STS is easier to implement and provides good security through HTTPS and DNS. Using the two together can provide the best of both worlds, increasing security through a layered approach.
	
- **Outbound email**:
	- [Deploy SPF, DKIM, and DMARC the right way](https://vand3rlinden.com/post/spf-dkim-dmarc-explanation/)
		- SPF, DKIM, and DMARC are critical email authentication protocols that help prevent email spoofing, phishing attacks, and domain impersonation for outbound email.
	- [Get a handle on your SPF record](https://vand3rlinden.com/post/handle-your-spf-record/)
		- It is crucial to have a well-structured SPF procedure to avoid future problems, especially since exceeding the DNS lookup limit of 10 can cause issues.
		
### 9: Promote a think before you click mindset
- [Attack simulation training](https://vand3rlinden.com/post/mdo-attack-simulation/)
	- With Microsoft Defender for Office 365, you can create an attack simulation training to identify vulnerable users and mitigate potential threats before they impact your organization.
	
### 10: Understand how inbound email works in Microsoft 365
- [How inbound email works in Microsoft 365](https://vand3rlinden.com/post/mdo-handling-false-positives-false-negatives/)
    - To understand why your environment experiences **false positives** and **false negatives**, you first need to know how Microsoft 365 processes inbound email and how to correctly use submissions and the Tenant Allow/Block List.
		
## Final words
Since email security is still one of the main attack vectors used by malicious actors, you should not underestimate the importance of a strong Microsoft Defender for Office 365 setup, or any email security solution. If you have serious concerns about email privacy, you may also want to consider signing and encrypting your or the messages of your key users using [PGP](https://vand3rlinden.com/post/pgp-secure-email-communication/) or [S/MIME](https://vand3rlinden.com/post/s-mime-enhancing-email-security/). These methods ensure that your messages remain private and protected from unwanted access by big tech providers (even if you send them through Microsoft 365) or governments.


## Resources
- [Recommended email and collaboration threat policy settings](https://learn.microsoft.com/en-us/defender-office-365/recommended-settings-for-eop-and-office365)