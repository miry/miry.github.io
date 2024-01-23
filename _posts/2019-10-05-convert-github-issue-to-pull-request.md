---
url: https://medium.com/notes-and-tips-in-full-stack-development/convert-github-issue-to-pull-request-c624834835d8
canonical_url: https://medium.com/notes-and-tips-in-full-stack-development/convert-github-issue-to-pull-request-c624834835d8
title: Convert Github Issue to Pull request
subtitle: Manage Github issues more effectively with command line tool.
slug: convert-github-issue-to-pull-request
description: A short example how can you improve working with Issues and Pull requests
  processes in Github.
tags:
- github
- hub
- git
- pull-request
- command-line
author: Michael Nikitochkin
username: miry
---

![Photo by Harley-Davidson on Unsplash](/assets/2019-10-05-convert-github-issue-to-pull-request-1_0dQuG3aJ3tGIFe3R6KgqpA.jpeg)

# Convert GitHub Issue to Pull request

The typical difficulty for most GitHub users is **to manage 2 cards per task** (*an Issue* and a connected *Pull request*). It consumes a lot of time and energy to maintain a project, mainly when you create an *Issue* before you start coding.

> **TL;DR:** **hub** **pull-request** attaches a branch to the existing GitHub issue or creates a new pull request: `hub pull-request -i <issue number>`

The most common steps for new users are:

* Create *“Issue #1”*;

* Start work on the problem in a new branch;

* After completion of the problem, create a new “*Pull request #2*” with “fixes #1” in description via Web UI;

* Sync “*Pull request #2*” attributes and states with “*Issue #1*”;

* Merge “*Pull request #2*”. It closes automatically “*Issue #1*”.

Instead of having one issue, we have 2. It introduces some problems:

***The first problem*:** The original description and communication located in the *“Issue #1”*. However, after a developer ask for a review of *“Pull request #2”*. People comment and continue the discussion in the pull request. It makes a bit harder to find information related to the task later.

***The second problem*** is to keep the state of the original *“Issue #1”* the same as for *“Pull request #2”*.

*Example*: in the project board you moved the “*Pull request #2*” to ‘Review’ column — please move “*Issue #1*” to the same column.

***The third*:** you should sync labels, milestones, and assigners. **It is not [a DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) way.**

*Example*: marked “*Issue #1*” with a label “*blocked*” — don’t forget about *“Pull request #2”*.

My suggestion — use the [hub](https://github.com/github/hub) command-line tool instead of Web UI. There are at least several advantages:
- You can create a new pull request, right just after you push your changes to GitHub;
- You can reuse the original *Issue*;
- Automate the creation of a new pull request with common attributes for your project.

Here is a sample workflow of how I convert “*Issue #1*” to “*Pull request #1*”:

```
$ cd <path to repo>
$ git checkout -b 1-change-title
$ git commit -a
$ git push
$ hub pull-request -i 1
```

Voila, there is no “*Issue #1*” anymore, but “*Pull request #1*” goes with the branch **1-change-title** attached and the same description and comments. Plus, your GitHub project automatically moves this Issue to the next column (if you have setup column automation).

For GitHub Enterprise users, here is a hint of how to change default domain to custom:

```
$ git config add hub.host github.jetthoughts.com
```

If you need to create a fresh new *“Pull request”* — run `hub pull-request` without the `-i` argument.

Check for other [**hub pull request options**](https://hub.github.com/hub-pull-request.1.html) available: create a draft pull request, assign reviewers, set labels, and other choices.

**Hub authentication**

The first challenge to use `hub` would be authentication. `hub` would ask for **Github** **user** and **password**, but the** password** is not working.

```
$ hub pull-request -i 321
github.com username: miry
github.com password for miry (never stored): 
```

You should use a token instead. Visit **Github -> Settings ->Developer settings -> Personal Access tokens** via [https://github.com/settings/tokens](https://github.com/settings/tokens).

Create a new token with at least scopes to access private repos, gist, and write discussion. Use the generated token in prompt for password in the terminal window.

![Github token scopes](/assets/2019-10-05-convert-github-issue-to-pull-request-1_02a_1hajFgwJjfDXXhymmA.png)

More details about you can find in [the man page](https://hub.github.com/hub.1.html#github-oauth-authentication).

### Summary

Stop creating a separate pull request for each *Issue* on GitHub.
I encourage you to learn more about other commands from [**hub**](https://hub.github.com/).

![](/assets/2019-10-05-convert-github-issue-to-pull-request-1_iifqfnqorqkZVMpCyq1BjA.png)

**Michael Nikitochkin** *is a Lead Software Engineer. Follow him on [LinkedIn](https://www.linkedin.com/in/michaelnikitochkin/) or [GitHub](https://github.com/miry).*

> *If you enjoyed this story, we recommend reading our [latest tech stories](https://jtway.co/latest) and [trending tech stories](https://jtway.co/trending).*

### Refernces

https://hub.github.com/hub-pull-request.1.html


