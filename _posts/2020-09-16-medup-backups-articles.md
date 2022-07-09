---
url: https://medium.com/notes-and-tips-in-full-stack-development/medup-backups-articles-8bf90179b094
canonical_url: https://medium.com/notes-and-tips-in-full-stack-development/medup-backups-articles-8bf90179b094
title: Medup backups articles
subtitle: I am glad to present you my little project Medup to export Medium posts
  in markdown format.
slug: medup-backups-articles
description: ""
tags:
- medup
- crystal-lang
- open-source
author: Michael Nikitochkin
username: miry
---

# Medup backups articles

> **TL;DR** Explain features of medup client tool to export Medium and Dev.to posts

I am glad to present you my little project [**Medup**](https://github.com/miry/medup/) to export **Medium and Dev to posts** in **markdown** or **json** format.

![Photo by Mike Setchell on Unsplash](/assets/2020-09-16-medup-backups-articles-0_LZaURw4xtfA74nu9.jpeg)

# UPDATES:

**2023–11–02:** Add example of Soundcloud and Codepen embeded elements;
**2022–07–26:** Update document with more complex examples with overlapped styles and hidden links;
**2022–07–09:** Introduce Dev.to platform;
**2022–05–09:** Add example how to use Jekyll and Bridgetown with custom assets path;
**2022–04–30:** Update article to cover changes in v0.4.0 version;
**2022–04–27:** Insert Youtube element for testing;
**2022–04–09:** Update article to cover changes in v0.3.0 version;

# Usage

The app can only work with public posts, that’s why no need of *Medium* *API* *key* or credentials. The default directory of all exported articles is `posts`. It is easy to change the distination with option `-d <path>`.

**There are three ways to export articles:**

### By Post URL

It is a basic command that help you convert a post by *URL*

```
medup https://jtway.co/git-minimum-for-effective-project-development-841a0b865ef0
```

*URL* could contain a custom domain of a publications or standard medium domain.

### By Author

It is usefull when you are looking to backup all your posts or someone else. Default platform is **Medium**.

```
medup -u <username>
medup @<username>
```

### By User recommendations/claps

I have a list of good articles, that would like to read time to time. With extra argument `-r` download articles from a recommended list of a user:

```
medup -u <username> -r
medup @<username> -r
```

### By Publication

Export all posts published in Publications by publication name.

```
medup -p <publication slug>
medup <publication slug>
```

Example publication JetThoughts, that is accessed via [https://jtway.co/](https://jtway.co/), has a slug: “jetthoughts”. There is a way to get the slug — open view source of the page and search for text `data-collection-slug`. This attribute would contain the real slug name of the publication. After you identified the slug you can run: `medup -v7 jetthoughts` and check folder `posts` .

**Platform**

There are 2 supported platforms: Medium and Dev.to. The default platform is Medium. Export articles for author or organisation specify the argument `--platform=<devto|medium>` .

### Cheat Sheet

```
#!/usr/bin/env bash

medup -u miry -d ./posts/miry          # Articles written by miry
medup --platform=devto -u miry -d ./posts/miry_devto  # Articles written by miry in Dev.to
medup @miry -d ./posts/miry            # Alternative way to get articles written by miry
medup -u miry -d ./posts/favorites -r  # Favorite articles of miry (clapped one)
medup -u miry -d ./posts/miry --update # Update existing exported posts with latest versions of posts
medup -u miry -d ./posts/miry --assets-images  # Save images to assets folder
medup @miry -d ./posts/miry --assets-images --assets-dir ./assets --assets-base-path=/assets # Save images to assets folder and update base path from relative to absolute
medup -p jetthoughts -d ./posts/jetthoughts # Export Jetthought publication's posts
medup jetthoughts -d ./posts/jetthoughts # Alternative way to export Jetthought publication's posts
medup https://medium.com/notes-and-tips-in-full-stack-development/medup-backups-articles-8bf90179b094 # Export single article
medup https://jtway.co/git-minimum-for-effective-project-development-841a0b865ef0 # Export signle article with custom domain
medup -v7 -d _posts --assets-dir=assets --assets-base-path=/assets @miry # Export posts to Jekyll
medup -v7 -d src/_posts --assets-dir=src/assets --assets-base-path=/assets jetthoughts # Export posts to Bridgetown
medup https://dev.to/jetthoughts/      # Articles wirtten by jetthoughts in dev.to
medup https://dev.to/jetthoughts/how-to-use-linear-gradient-in-css-bi1  # Single article to export from dev.to
medup jetthoughts --dry-run            # Run command in read only mode. No modification in file system.



```
> *[usage.sh view raw](https://gist.githubusercontent.com/miry/d7e8a19eb66734fb69cf8ee4c32095bc/raw/221848cb3f0d2e3f77bb043dbd8f888d721270de/usage.sh)*

# Features

### Images

Support image embedding to a result markdown document. In general it downloads and encode an image. After put the image to the bottom of the document to have better read expirience.

It is possible to save images in `assets` folder with option `--assets-images`.

![Check JPEG images](/assets/2020-09-16-medup-backups-articles-1_CSF4xue7yFfg-9-wxAkDWw.jpeg)

```
medup @miry --assets-images
```

### Code Blocks

Code blocks are easy to convert, because it has similar structure to markdown format.

### Inline code, Bold and Italic

Convert words with formating styles to correspondent markdown styles.

### Paragraphs

Split the text blocks to be close to original document.

### Gists/Embeded

Converts *gists* or embed content to **IFRAME** and load as assets from the separate file. Included to the document as **HTML tag**. Currently there is a limitation to access gists 60 request per hour. If gist service returns 403 error code, **IFRAME** solution would be used.

### Youtube/Embeded

Converts *Youtube* media elements to **IFRAME** and load assets from the separate file. Included to the document as **HTML tag**.

[![Youtube](https://img.youtube.com/vi/30xiI21RraQ/hqdefault.jpg)](https://www.youtube.com/watch?v=30xiI21RraQ)

Caption under the video

### Twiter/Embeded

Here is an example of tweet message

> With https://t.co/MSHpm3lVIj I see how often my favorites articles were changed and see previous history of content. I am thinking to create a service to show archives of #medium articles or export to markdown. If you have ideas submit to https://t.co/yEV9W9b2ph
> - [@miry_sof](https://twitter.com/miry_sof/status/1519267429016297473)

Caption for the tweet message

### Soundcloud

__soundcloud__:

[<img height="166" alt="SoundCloud" src="https://i1.sndcdn.com/artworks-n6oiDHCmyOARrGyS-gr5J5g-t500x500.jpg"></img>](https://w.soundcloud.com/player/?url=https%3A%2F%2Fapi.soundcloud.com%2Ftracks%2F1635297666&show_artwork=true)

Caption for the Soundcloud embeded

I’ve included a sample SoundCloud embedded element with a customized style to increase its width compared to regular text. This will help us assess its compatibility with Markdown.

### Codepen

__codepen__:

[<img height="600" alt="CodePen" src="https://shots.codepen.io/username/pen/WqrrWK-512.jpg?version=1653663802"></img>](https://codepen.io/andriyparashchuk/embed/preview/WqrrWK?default-tabs=css%2Cresult&height=600&host=https%3A%2F%2Fcodepen.io&slug-hash=WqrrWK)

[https://codepen.io/andriyparashchuk/pen/WqrrWK?editors=1100](https://codepen.io/andriyparashchuk/pen/WqrrWK?editors=1100)

An example of Codepen integration with a link in the caption .

### Other rich elements/Embeded

Depends on an element it could create **IFRAME** or would use **IMAGE** with a link to the external resource.

### [🇺🇦 Titles with Emoji and links](https://medium.com/notes-and-tips-in-full-stack-development/medup-backups-articles-8bf90179b094)

Calculate emoji symbols size with Medium format.

### Overlapped styles with hidden links

***Paul Keen** is a Chief Technology Officer at [JetThoughts](https://www.jetthoughts.com). Follow him on* [LinkedIn](https://www.linkedin.com/in/paul-keen/) *or [GitHub](https://github.com/pftg).*

# Installation

### Homebrew

For *MacOS* users installation should be easy peasy. The most popular package manager for MacOS is [**Homebrew**](https://brew.sh/)

```
$ brew tap miry/medup
$ brew install medup
$ medup https://medium.com/notes-and-tips-in-full-stack-development/medup-backups-articles-8bf90179b094
```

### Docker

Other solution is to use [**Docker**](https://docs.docker.com/get-docker/):

```
$ docker run --rm -v $(pwd):/posts -it miry/medup [-u <user>|<url>]
```

It allows users to skip installation of [**Crystal**](https://crystal-lang.org/) and other development libraries.

### From sources

The code is written in Crystal. Build a binary use next commands

```
$ rake build
$ _output/medup -u <medium user> -d <destination folder>
```

The build is located in `_output/medup`.

# Demo

There is a way to test how the export is working. The project has automated task like:

### Jekyll

It is possible to create a Demo site with single command:

```
$ rake demo:serve
```

In the background for taks it do next:

```
$ gem install bundler jekyll
$ jekyll new demo
$ cd demo
$ bundle add webrick
$ bundle
$ cat <<EOF >> _config.yml
defaults:
  - scope:
      path: ""
    values:
      layout: default
EOF
$ medup -v7 -d _posts --assets-dir=assets --assets-base-path=/assets @miry
$ bundle exec jekyll serve
```

Then open in browser [http://localhost:4000/](http://localhost:4000/).

### Bridgetown

It is possible to create a Demo site with single command with [Bridgetown](https://www.bridgetownrb.com/):

```
$ rake demo:bridgetown:serve
```

In the background for taks it do next:

```
$ gem install bundler bridgetown
$ bridgetown new demo
$ cd demo
$ bundle
$ cat <<EOF >> src/_posts/_defaults.yml
layout: post
EOF
$ medup -v7 -d src/_posts --assets-dir=src/assets --assets-base-path=/assets @miry
$ bin/bridgetown start
```

Then open in browser [http://localhost:4000/](http://localhost:4000/)

# Contribution

All contribution to the project are welcome. You have some new ideas or just want to check for new features check the [Project Board](https://github.com/miry/medup/projects/1). I apreciate any help with implementing new features.

Do you use `medup` for your duties or routines let me know. I will be very happy to know about your use case.

# Key learnings

### Use less embeding sections as possible when create a Medium post.

Markdown is not so powerful as rich text rendeing engines. It was not designed for those things. Prefered way is to use plain code blocks over gist embeded snippets. It makes easy to read the exported document.

*to be continued…*

# Summary

I use this post as test with all possible elements for checking correctnes of **Medup**. Expect small changes in the document.

# References

https://github.com/miry/medup/


