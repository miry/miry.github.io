---
url: https://medium.com/notes-and-tips-in-full-stack-development/visualization-of-your-code-d4701affa767
canonical_url: https://medium.com/notes-and-tips-in-full-stack-development/visualization-of-your-code-d4701affa767
title: Visualization of Your Code
subtitle: 'Working at long and big projects, it is fun to see on a release date how
  this project was built: who worked at a particular part; when a…'
slug: visualization-of-your-code
description: ""
tags:
- homebrew
author: Michael Nikitochkin
username: miry
---

# Visualization of Your Code

![](/assets/2017-06-14-visualization-of-your-code-1_JwZVyrKtHu3QxTRbJ4ck6A.jpeg)

Working at long and big projects, it is fun to see on a release date how this project was built: who worked at a particular part; when a new guy joined the project, deleted some code and did refactoring; or when the lead developer went to vacation. There is a lot of such tools, you can even use a browser version. But I still prefer using `gource` (https://github.com/acaudwell/Gource)

This story would be about how to setup and automize video creation, what general issues you will encounter and other stuff.

It is an old tool. I started using it in 2009 when we finished the next sprint.

# Step 1: Installation

![](/assets/2017-06-14-visualization-of-your-code-1_LBp-koQb0Djywov4ZpWKsA.jpeg)

Ok, you wouldn’t believe me, but I use … use … *MacOS* :)

```
$ brew update
$ brew install gource
```

That’s all.

# Step 2: Show simple video

Now you are able to see visualization of any project of yours.

```
$ cd <path to project>
$ gource
```

# Step 3: Avatars

Ok, now you see some color dots and color figures. Let’s add some recognizable images to authors. There is an old example showing how to get gravatars of users:

```
$ curl https://gist.githubusercontent.com/miry/dbf01f5a198060710255/raw/d07db1571a045e057e6d56444943bc798dd91be3/authors_gravatar.pl > authors_gravatar.pl
```

Run the script `authors_gravatar.pl` and it would fetch gravatar images by author’s email to `.git/avatar`.

```
$ perl authors_gravatar.pl
$ gource --user-image-dir .git/avatar
```

# Step 4: Video

First read this article [Gource Videos](https://code.google.com/p/gource/wiki/Videos#Linux_/_Mac)

```
$ gource --camera-mode overview --seconds-per-day 1 --user-image-dir .git/avatar/ -1280x720 -o gource/gource.ppm 

$ ffmpeg -y -r 60 -f image2pipe -vcodec ppm -i gource/gource.ppm -vcodec libx264 -preset ultrafast -pix_fmt yuv420p -crf 1 -threads 4 -bf 0 gource/gource.mp4
```

# Step 5: Soundtrack

Open video file in QuickTime. From Finder Drag and Drop our soundtrack file to the QuickTime window. It is very easy.


