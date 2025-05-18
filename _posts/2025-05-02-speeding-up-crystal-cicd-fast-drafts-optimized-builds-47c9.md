---
url: https://dev.to/miry/speeding-up-crystal-cicd-fast-drafts-optimized-builds-47c9
canonical_url: https://dev.to/miry/speeding-up-crystal-cicd-fast-drafts-optimized-builds-47c9
title: 'Speeding Up Crystal CI/CD: Fast Drafts, Optimized Builds'
slug: speeding-up-crystal-cicd-fast-drafts-optimized-builds-47c9
description: I have started working on a production web application built with Crystal
  and Marten. With every new...
tags:
- crystal
- crystallang
- marten
- cicd
author: Michael Nikitochkin
username: miry
---

![Cover image](/assets/2025-05-02-speeding-up-crystal-cicd-fast-drafts-optimized-builds-47c9-cover_image-bnay3nzy3vzfe05b2uky.png)

# Speeding Up Crystal CI/CD: Fast Drafts, Optimized Builds


I have started working on a production web application built with Crystal and Marten.
With every new feature I add to the project, the compilation time keeps growing—almost exponentially.

I found that waiting 50 minutes to build an image isn't worth it for quick experiments. I realized I don't need full performance for development builds.

Here's my view on how I can address the problem:

I'd like to introduce a "draft" image that builds in around 3 minutes and is ready for deployment—it even starts deploying to the production clusters.

After that, it would trigger a "pristine" build with all optimizations enabled, which might take 60 minutes.

![Man in front of north lights](/assets/2025-05-02-speeding-up-crystal-cicd-fast-drafts-optimized-builds-47c9-8wmox25fzvg8qiuvctfr.png)

If a new build is triggered in the meantime, the pristine build is cancelled and replaced by the most recent one job.

With this approach, I can still build and test quickly, while eventually delivering a highly optimized version for better performance.




