---
title: 'Microsoft Purview: Configuring a Data Loss Prevention Policy'
date: 2023-12-16T11:31:31+01:00
draft: false
categories: ["Microsoft Purview"]
---

> Organizations have sensitive information under their control, such as credit card numbers or social security numbers. To protect this sensitive information and reduce risk, organizations need a way to prevent users from sharing it with people who shouldn't have it. This practice is known as data loss prevention (DLP). In this article, I will explain how to configure it.

## Sensitive information type entity definitions
In this post, I will configure a DLP policy to block the Netherlands Citizen’s Service (BSN) Number from being shared with external contacts in Exchange Online and Teams. However, you can use this information to select any of the available sensitive information types or multiple types in a policy. These types of sensitive information are referred as Personally Identifiable Information (PII).

## Creating a DLP Policy
1. Sign in to the [Microsoft Purview compliance portal](https://compliance.microsoft.com/)

2. In the Microsoft Purview compliance portal > left navigation > Solutions > Data loss prevention > Policies > + Create policy.

3. Select ***Custom*** from the ***Categories*** list.

4. Select ***Custom*** from the ***Regulations*** list.

5. Give the policy a name (policy name cannot be changed later).

6. Fill in a description. You can use the policy intent statement here.

7. Select ***Next***.

8. Select ***Full directory*** under Admin units.

9. Set the ***Exchange email*** _(Scope: All groups)_ and ***Teams chat and channel messages*** _(Scope: All users & groups)_ locations status to On. Set all the other location status to Off.

10. Select ***Next***.

11. Select Create rule. Name the rule and provide a description.

12. Under Conditions select Add condition > Content contains > Add > Sensitive info types > Netherlands Citizen’s Service (BSN) Number. Choose Add.

13. Under Actions select Add an action > Restrict access or encrypt the content in Microsoft 365 locations > Block only people outside your organization.

14. Set User notifications to ***On***.

15. Select > Notify the user who sent, shared, or last modified the content.

16. Under Incident reports, select whether you want to send a report to your (compliance) admins when the policy is violated.

17. Select Save and you will be presented with a review of the rule.
    ![IMAGE](/images/purview-data-loss-prevention/dlp-condition-example.png)

18. Select ***Next***.

19. Test or save your policy, it can take up to an hour for the policy to take effect.

20. Review and finish your DLP policy
    ![IMAGE](/images/purview-data-loss-prevention/dlp-review-finish.png)

## Try it out for yourself
You can test this yourself when the policy is active. The e-mail or Teams message must contain at least the keywords described [here](https://learn.microsoft.com/en-us/purview/sit-defn-netherlands-citizens-service-number#keywords_netherlands_eu_national_id_card).

Do not use your own citizen service number or any other type of sensitive information for testing purposes. Search for an online generator such as https://cyberwar.nl/elfproef.html to generate Netherlands Citizen Service Numbers (BSN). Intended for creative testing, NOT abuse.

## Examples
- Exchange Online
    ![IMAGE](/images/purview-data-loss-prevention/dlp-exo-sender.png)
- Teams
    ![IMAGE](/images/purview-data-loss-prevention/dlp-teams-sender.png)

## Reference
- [All Sensitive information type entity definitions](https://learn.microsoft.com/en-us/purview/sensitive-information-type-entity-definitions)
- [All Microsoft Purview documentation](https://learn.microsoft.com/en-us/purview/)
- [More DLP scenarios](https://learn.microsoft.com/en-us/purview/dlp-create-deploy-policy)
