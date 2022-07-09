---
url: https://dev.to/miry/make-an-alias-for-existing-branch-5cf3
canonical_url: https://dev.to/miry/make-an-alias-for-existing-branch-5cf3
title: Make an alias for existing branch
slug: make-an-alias-for-existing-branch-5cf3
description: Photo by Mohammad Rahmani on Unsplash   If you happened to miss the master
  branch and your...
tags:
- git
author: Michael Nikitochkin
username: miry
---

![Cover image](/assets/2024-04-10-make-an-alias-for-existing-branch-5cf3-cover_image-https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3ezumsv3r7llh0x3meek.jpg)

# Make an alias for existing branch


> Photo by <a href="https://unsplash.com/@afgprogrammer?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Mohammad Rahmani</a> on <a href="https://unsplash.com/photos/a-close-up-of-a-keyboard-on-a-table-lPKIb8dJ8kw?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Unsplash</a>
  

If you happened to miss the `master` branch and your organization uses the more populistic `main` naming convention as the default branch, don't worry!
You can easily switch back to the previous environment without causing any issues using a simple Git alias command.: 

```shell
$ git symbolic-ref refs/heads/master refs/heads/main
```

This tool won't replace everything, but it can help you use old commands like:

```shell
$ git checkout master

$ git rebase -i master

$ git diff master
```



