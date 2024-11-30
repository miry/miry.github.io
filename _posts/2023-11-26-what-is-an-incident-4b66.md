---
url: https://dev.to/miry/what-is-an-incident-4b66
canonical_url: https://dev.to/miry/what-is-an-incident-4b66
title: What is an Incident?
slug: what-is-an-incident-4b66
description: This article explores incident management, defining incidents as unforeseen
  disruptions. It emphasizes categorization, prioritization, and resilient practices.
  The internal team is acknowledged as a vital customer. Learning from incidents involves
  risk analysis, and knowledge sharing is facilitated through various practices. In
  conclusion, a well-defined incident management process is crucial for prompt and
  effective responses.
tags:
- sre
- incident
- incidentresponse
- incidentmanagemenent
author: Michael Nikitochkin
username: miry
---

![Cover image](/assets/2023-11-26-what-is-an-incident-4b66-cover_image-duhb38496cur2suqfvsi.jpg)

# What is an Incident?


## Understanding Incidents

An incident is an unforeseen event that disrupts regular operations, demanding immediate attention to minimize its impact and restore normalcy.

While definitions may slightly vary among experts, incidents generally encompass events or occurrences that adversely affect an organization's processes, services, or resources, manifesting in diverse forms.

## Categorizing and Prioritizing Incidents

Most companies necessitate categorizing incidents based on type and criticality, aligning priorities to balance resources between new features and maintenance work. Clearly defining impact rules aids developers in enhancing response times. Communicating incident definitions to customers fosters transparency, clarifying criticality and delineating progress versus maintenance.

Incidents can be classified as internal or customer-facing, the latter potentially involving third-party providers. Resilient practices, when in place, shield customers from visibility, enabling prioritization based on team load or postponement to regular working hours. In customer-facing incidents, social networks and customer support channels may witness a surge in activity, even for seemingly minor incidents affecting core features, potentially eroding trust in the product.

Automated fallbacks in workflows, if effective, may not constitute incidents but rather showcase resilient practices. If, however, these fallbacks prove insufficient and create issues for internal or external stakeholders, an incident arises.

**Example of Incident Prioritization:**

1. Success rate of Checkouts is below 50% globally.
2. 50% of users are affected and cannot complete critical business operations.
3. Reporting data for users is delayed by more than 1 hour.
4. 10% of users have corrupted data.

Adjusting percentages and emphasizing critical metrics through visible dashboards is essential for evaluating the impact effectively.

## Incidents Have Customers

Not all incidents are apparent to external business customers. Issues with deployment or observability tooling may not disrupt user flows but can render the system vulnerable to potential major incidents. Treating internal teams as customers, especially infrastructure teams, is vital. They serve as service providers akin to application developers, ensuring a holistic approach to maintaining developer happiness and facilitating faster feedback collection.

## Learning from Incidents

Analyzing incident risks can preempt known incidents. Practices such as diversifying resources and establishing fallbacks mitigate potential incidents. Some companies categorize any alert as an incident, leading to burnout among teams. Streamlining incident signals aids on-call teams in focusing on essential tasks.

## Sharing Knowledge

Facilitating easy access to past incidents, complete with relevant information, promotes knowledge sharing. Regularly reporting key incidents aids other teams in staying informed and leveraging their expertise.

**Some Practices in Place:**

- Regular incident reporting publicly or internally (e.g., Github Availability reports [^1], Cloudflare Post Mortmes [^2], Internal Newsletters).
- Incident tools in service catalogs, working on postmortems, and analytics about the number and types of incidents per service, categorizing action items.
- Internal channels for communication about past or current incidents.

## Conclusion

For organizations, a well-defined incident management process is indispensable for prompt and effective responses to incidents. Structuring incident management reduces chaos, fostering trust between stakeholders. Embracing a comprehensive approach to incident management is pivotal in navigating the complex landscape of unforeseen disruptions.

## References

[^1]: [Github Availability reports](https://github.blog/tag/github-availability-report/)
[^2]: [Cloudflare Post Mortmes](https://blog.cloudflare.com/tag/post-mortem/)

- [SRE Book: Chapter 14 - Managing Incidents](https://sre.google/sre-book/managing-incidents/)
- [Atlassian Support: What are incidents?](https://support.atlassian.com/jira-service-management-cloud/docs/what-are-incidents/)



