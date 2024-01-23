---
url: https://medium.com/notes-and-tips-in-full-stack-development/dont-release-the-same-feature-twice-c816bb47bb0d
canonical_url: https://medium.com/notes-and-tips-in-full-stack-development/dont-release-the-same-feature-twice-c816bb47bb0d
title: Don’t Release The Same Feature Twice
subtitle: I do not like bureaucracy, but I can not live without it. That’s why I have
  integrated Changelog to the project. After a few deploys, I…
slug: dont-release-the-same-feature-twice
description: ""
tags:
- git
- gitflow
author: Michael Nikitochkin
username: miry
---

# Don’t Release The Same Feature Twice

I do not like bureaucracy, but I can not live without it. That’s why I have integrated `Changelog` to the project. After a few deploys, I have met with magic. In the project `Changelog` file there are two records of one issue. It confused me. Let me explain what I imagined in my head.

![](/assets/2017-06-13-dont-release-the-same-feature-twice-1_b6iKn0k6GuDUNplyR2_Q3A.jpeg)

First feelings were as if a seller had sold the same thing to many customers :)

### Why it is not good for me?

We could not guarantee what was released in each version. Did we resolve issue in the first version, and what changes were in next versions?

### Why it is good for others?

I have asked my team. There was a bug in recently deployed version of an application. And instead of creating a new issue, just kept updating this one. And got following answers, why it could be good:

1. **All information related to this feature is kept in one place**. Ok, this is a good reason, but maybe we should track all new features in Wiki? After successful deploy, we will update documentation. And all new guys will be able to find useful information in one place.

1. **We can create a fix in the same branch**. I don’t think it is a good idea. According to Gitflow, we should create a patch in a hotfix branch instead. If we don’t use such flows, we should rebase the branch first. But what is the difference, if I create a new branch and rebase the merged branch against master?

### Solution

After deploy, instead of reopen, create a new issue, and mention about the related one in it. In this case we always knew what was wrong, how did we fix it and how fast we apply a patch. Also we can count how many bugs we have missed after deploy to production.


