---
title: 'Microsoft Purview: Purging mails from Inboxes'
date: 2023-12-23T23:57:43+01:00
draft: false
categories: ["Microsoft Purview"]
cover: 
  image: /images/purview-purge-emails/purview-purge-emails-front.png
---

> _Purging emails from inboxes allows you to remove messages that your users should not have received. Utilizing Microsoft Purview, you have the capability to search for specific content and initiate the purging process. This post provides guidance on how to achieve this._

## Before getting started
Before commencing the purging process, it's crucial to acknowledge that some individuals may have already opened the email. Purging emails won't guarantee that no one has seen them, but it will reduce the number of individuals who have.

## Let's begin
1. You must designate yourself as an **eDiscovery Administrator** within the [Microsoft Purview portal](https://compliance.microsoft.com/compliancecenterpermissions).

![IMAGE](/images/purview-purge-emails/purview-purge-emails-1.png)

2. Initiate the creation of a new [content search](https://compliance.microsoft.com/contentsearchv2).

![IMAGE](/images/purview-purge-emails/purview-purge-emails-2.png)

3. Assign it a name such as ***Purge TICKETNUMBER***, and in the description, outline the content you intend to purge.

![IMAGE](/images/purview-purge-emails/purview-purge-emails-3.png)

4. Navigate to the "Locations" tab and configure the location settings to "Exchange Online," encompassing all users.

![IMAGE](/images/purview-purge-emails/purview-purge-emails-4.png)

5. Go to the ***Conditions*** tab and configure a condition using the ***Condition Card Builder***, such as ***Sender*** or ***Subject***. You can also use the ***KQL Editor*** to query the ***EmailEvents*** table for a more comprehensive search.

![IMAGE](/images/purview-purge-emails/purview-purge-emails-5.png)

6. Once you've configured the conditions, proceed to the next step. Review your search settings and save the configuration.

7. Once the search is completed, review the output by clicking on the search name.

> **IMPORTANT:** Verify the emails you intend to purge by checking the preview. It's crucial to confirm this, as failure to do so might result in deleting more than intended.

![IMAGE](/images/purview-purge-emails/purview-purge-emails-6.png)

8. Now, initiate the purging process for your search. To do this, log in to the Security & Compliance Center with PowerShell using the Exchange PowerShell module.
```
Connect-IPPSSession -UserPrincipalName UPN@domain.com
```

9. Once logged in, you are prepared to delete the content as outlined in step 7.
```
New-ComplianceSearchAction -SearchName "Name of content search" -Purge -PurgeType HardDelete
```

10. You can monitor the status of the purge using the following command.
```
Get-ComplianceSearch -Identity "Name of content search"
```

## Ending up
After the purge is finalized, the content will be deleted from the Inboxes within a few minutes. It's crucial to bear in mind, as mentioned in step 7, that purging should only be executed if you are 100% certain about the output of the search.