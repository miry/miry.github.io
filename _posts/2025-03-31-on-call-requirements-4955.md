---
url: https://dev.to/miry/on-call-requirements-4955
canonical_url: https://dev.to/miry/on-call-requirements-4955
title: On-Call Requirements
slug: on-call-requirements-4955
description: This document outlines on-call requirements for global companies. Since
  employees are spread across various countries, each with its own labor laws, it's
  essential to align expectations before joining an on-call rotation.
tags:
- sre
- incidentresponse
- incidentmanagement
- resiliency
author: Michael Nikitochkin
username: miry
---

![Cover image](/assets/2025-03-31-on-call-requirements-4955-cover_image-6eb2nq71dizalts427pb.jpg)

# On-Call Requirements


## Summary

This document outlines on-call requirements for global companies. Since employees are spread across various countries, each with its own labor laws, it's essential to align expectations before joining an on-call rotation.

Being on-call comes with responsibilities and limitations. It affects your social life, sleep schedule, and availability. You serve as a crucial safety net for the organization.

![Image description](/assets/2025-03-31-on-call-requirements-4955-tu89uly1d6h21qlt5hqj.jpg)

## Hardware

### Phone

The company should provide a dedicated on-call phone. It doesn’t need to be high-end but must be secure and support the following apps:

* **PagerDuty** or **Opsgenie** – for incident notifications
* **Mail** – for alerts and updates
* **Slack**, **Discord**, or **Google Meet** – for team communication
* **Browser** – for troubleshooting
* **1Password** (or similar) – for credential management
* **Two-Factor Authentication** (2FA) apps – FreeOTP, Yubico Authenticator, Authy, etc.

### Mobile Contract

The mobile plan should support incoming calls from **PagerDuty** and allow incident acknowledgment via mobile data. A *10GB* monthly data plan is typically sufficient. In case of an incident, the developer should be able to connect their laptop and triage the issue from wherever they are. Outgoing calls are only needed for escalation when other methods fail.

![Image description](/assets/2025-03-31-on-call-requirements-4955-buz7emxufqn3s8xzjmcu.jpg)

## Balanced On-Call

To avoid burnout, no more than 25% of an engineer's time should be spent on-call. Following this rule:

* A single-site team requires at least eight engineers for a sustainable rotation.
* A dual-site team should have at least six engineers per site.
* Each shift should include both a primary and secondary on-call engineer.

## Quality Balance

Engineers need time for incident response and follow-ups, including writing postmortems. An incident is defined as a sequence of events related to the same contribution factor and should be treated as a single issue.

## On-Call Policies & Practices

A well-structured on-call system requires clear policies to ensure smooth operations. Engineers should not have to figure things out when an alert goes off. Instead, proactive planning should include:

* **Incident severity definitions**
* **Playbooks for common issues**
* **Clear escalation rules**

Aligning these elements in advance helps create an effective and manageable on-call process.


