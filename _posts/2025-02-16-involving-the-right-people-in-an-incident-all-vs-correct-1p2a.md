---
url: https://dev.to/miry/involving-the-right-people-in-an-incident-all-vs-correct-1p2a
canonical_url: https://dev.to/miry/involving-the-right-people-in-an-incident-all-vs-correct-1p2a
title: Involving the Right People in an Incident
slug: involving-the-right-people-in-an-incident-all-vs-correct-1p2a
description: Effective incident management requires bringing in the right people instead
  of everyone, leveraging observability tools, and engaging teams incrementally to
  minimize noise, reduce stress, and ensure faster resolution.
tags:
- sre
- incidentresponse
- incidentmanagement
author: Michael Nikitochkin
username: miry
---

![Cover image](/assets/2025-02-16-involving-the-right-people-in-an-incident-all-vs-correct-1p2a-cover_image-22eq6xvkpfukyuypz1qx.jpg)

# Involving the Right People in an Incident


It's been a while since I last wrote about incidents. Lately, I’ve been more focused on backend development in a Ruby and Crystal projects, but after handling a few recent incidents, I wanted to jot down my thoughts.

### The Problem: Over-Involving Teams During an Incident

It’s common for an Incident Commander to be paged when something isn’t working. As the Incident Commander, you might see reports from customers. Your responsibility is to identify contributing factors and bring the right people together to stop the bleeding.

However, the concept of bringing the "correct" people is sometimes misunderstood. Some Incident Commanders assume this means inviting *everyone* who might be remotely involved. They create massive video or audio calls, hoping someone will figure out the problem. While this might seem like a thorough approach, it often leads to frustration among teams who are pulled into the incident but have nothing to contribute. They end up waiting passively, leading to wasted time and effort.

This broad approach may give Incident Commanders a false sense of control—believing that if all teams are present, they’ve done everything possible. But in reality, each team may assume the issue lies elsewhere, leading to passive listening rather than active problem-solving.

### The Consequences of Over-Involvement

Bringing too many people into an incident can have several negative effects:
- **High-cost meetings with low productivity:** More people in the call means more noise, more conflicting theories, and a harder time reaching a consensus.
- **Blame-shifting and distraction:** Each team might focus on their own long-standing issues rather than identifying the real root cause.
- **Loss of the bigger picture:** With too many perspectives, the core problem can become obscured, making it harder to pinpoint the actual failure.

In complex systems, problems can be hidden under layers of dependencies, making a broad approach ineffective. It’s crucial to separate valid long-term concerns from immediate incident causes.

![Situation room with a lot of folks](/assets/2025-02-16-involving-the-right-people-in-an-incident-all-vs-correct-1p2a-kz41pritx60wtupg14r6.jpeg)

### How to Solve an Incident Without Over-Involving Teams

So, how can an Incident Commander solve an unknown issue efficiently without involving too many people, while still resolving the problem as quickly as possible?

1. **Stop the bleeding** using all available tools. Start by narrowing down the issue to its closest impact point—typically where users are directly affected. Investigate progressively deeper into microservices and vendor solutions.
2. **Analyze patterns** by building a timeline and reproducing the problem as closely as possible to the reported issue. This is often the hardest step, especially if the issue is intermittent or device-specific.
3. **Leverage observability tools** across mobile, backend services, and database profiling. These tools should be a core part of every playbook.
4. **Identify the success lines** in monitoring reports to determine possible mitigation steps.
5. **Engage teams incrementally** — bring in only the necessary teams one at a time, verifying details with each and syncing on next steps before continuing the investigation. Even if details are shared in an incident channel, it's more effective to request targeted help in short bursts.
6. **Consult experts when needed** — if someone has experience with a similar issue, involve them, but avoid defaulting to large group calls.
7. **Track multiple leads separately** in different threads, summarizing findings regularly.
8. **Mitigation over resolution:** Depending on the incident’s criticality, full resolution might not be immediate. Collaborate closely with 1-2 relevant teams to assess mitigation strategies before broadening involvement.
9. **Maintain focused escalation:** Always escalate and page when necessary. Most people are willing to help, but ensure they have a clear role rather than keeping them in a call unnecessarily.

### Conclusion: All vs. Correct

Should you bring *everyone* into an incident call? Or should you focus on identifying the *correct* people? While including all teams might seem like a faster way to solve the problem, understanding the issue through observability tools and selectively involving the right teams is a more effective approach. This minimizes stress and improves resolution time.

Does this mean you should hesitate to escalate? Absolutely not — always escalate when necessary. People are generally willing to help, but ensure they have a clear role rather than keeping them in a call unnecessarily.

By shifting from an *“all-in”* approach to a *targeted*, *observability-driven* strategy, Incident Commanders can handle incidents more efficiently, reduce noise, and ensure faster recovery.

Of course, this isn’t something that can be perfected during an active incident. Understanding company structure, service dependencies, mitigation practices, and observability tools requires preparation. One of the best ways to improve is by reviewing past incidents and occasionally practicing simulated ones using exercises like *Wheel of Misfortune*.

And now, I trust you to make the right call!

---

Check out these resources to learn more:

- [Shrinking the time to mitigate production incidents—CRE life lessons](https://cloud.google.com/blog/products/management-tools/shrinking-the-time-to-mitigate-production-incidents)



