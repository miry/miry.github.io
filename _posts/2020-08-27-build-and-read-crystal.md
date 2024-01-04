---
url: https://medium.com/notes-and-tips-in-full-stack-development/build-and-read-crystal-b6ba21b122cf
canonical_url: https://codeberg.org/miry/samples/src/branch/master/blog/20200827_build_and_read_crystal.md
title: Build and Read Crystal
subtitle: How simple to build a crystal binary from sources?
slug: build-and-read-crystal
description: ""
tags:
- programming
- learning
- crystal-lang
author: Michael Nikitochkin
username: miry
---

![Photo by Jason D on Unsplash](/assets/2020-08-27-build-and-read-crystal-0_BmauN6EER-PdvCee.jpeg)

# Build and Read Crystal

> **TL;DR** Notes how to build crystal programming language from sources

# **Prologue**

I work on a small client tool written in Crystal.
It calls “[Medup](https://github.com/miry/medup/)”. As usual, when I see a new version of Crystal,
just in a few days, test my code against the latest version. 
Crystal [0.35.0](https://crystal-lang.org/2020/06/09/crystal-0.35.0-released.html) and [0.35.1](https://crystal-lang.org/2020/06/19/crystal-0.35.1-released.html) were released mostly at the same time. 
I got free time to test new changes.
Crystal has trendy significant differences between versions until
it would be released [1.0.0](https://crystal-lang.org/2020/03/03/towards-crystal-1.0.html).
I required small code changes to run `crystal build` with success -
added `case…else` statement explicitly.
New messages appeared during the build:

```
Warning: Deprecated JSON.mapping. use JSON::Serializable instead (the legacy behavior is also available in a shard at github:crystal-lang/json_mapping.cr)
```

It requires you to change a lot of places. Because those are just warnings,
I ignored it. As suggested in the next releases, 
I would use an extracted version *(I am not a fan of annotations)*.

All tests were passed. I started the compiled version of the application with
a new Crystal [0.35.1](https://crystal-lang.org/2020/06/19/crystal-0.35.1-released.html) — unexpected error appeared.
After small debugging, I found the problem in the code to parse the command line arguments.
Crystal’s **CHANGELOG** mentioned [a new optimization of OptionParser](https://github.com/crystal-lang/crystal/pull/9009/).

I found it super easy to read the PR and code. 
I decided to write my own tests and check my case against 
the current `master` version(probably fixed in recent commits).

# Build Crystal

I have a machine optimized for **Ruby** development.
For me, all dependencies were already pre-installed.
**The most exciting part, that Crystal is written in Crystal.**
It has requirements that every next release should be built with the previous stable version, in my case, it was 0.35.1. I tried to check compatibility with 0.34.0 — failed.
The source has new features from 0.35.0 like `case..in`.
Steps to build Crystal available in the official [site](https://crystal-lang.org/install/from_sources/):

```
$ git clone git@github.com:crystal-lang/crystal.git
$ cd crystal
$ make
$ .build/crystal -v
$ export CRYSTAL_PATH=`pwd`/src
$ .build/crystal <path to src file>
```

Tested my code with a new build crystal binary.
The build still has the same problem.

# Read Crystal

Make testing faster, I decided to add tests for my case
in the [spec/std/option_parser_spec.cr](https://github.com/crystal-lang/crystal/blob/b4659a183f4890e460935762e457f5deddb369a9/spec/std/option_parser_spec.cr#L1). Run only my changes with

```
$ .build/crystal spec spec/std/option_parser_spec.cr
```

After a few experiments, I found a mistake in my example.
It appears the small collision of names is treated differently after refactoring of `OptionParser`.
Here an example:

```
require “option_parser”
```

```
opts = %w[-u user other args]
OptionParser.parse(opts) do |parser|
 parser.on(“-u USER”, “ — user=USER”, “Some text”) { |u| puts “user triggered” }
 parser.on(“-u”, “ — update”, “Some text”) { puts “update triggered” }
end
```

```
pp opts
```

As you see, there are 2 short flags `-u`. In version before 0.35.0,
the implementation changed the behavior,
so the priority has the last specification.

# Summary

I don’t know any other language, where you can start contributing just in a few steps.
Crystal is still under construction and is growing with new features
and behavior every release.
It adds some complexity in maintaining production code on every upgrade,
but it’s worth it.

![That’s all Folks](/assets/2020-08-27-build-and-read-crystal-1_iifqfnqorqkZVMpCyq1BjA.png)

**Michael Nikitochkin** *is a Lead Software Engineer. Follow him on [LinkedIn](https://www.linkedin.com/in/michaelnikitochkin/) or [GitHub](https://github.com/miry).*

> If you enjoyed this story, we recommend reading our [latest tech stories](https://jtway.co/latest) and [trending tech stories](https://jtway.co/trending).

# References

https://github.com/crystal-lang/crystal

https://github.com/miry/medup/

https://crystal-lang.org/2020/03/03/towards-crystal-1.0.html

https://github.com/crystal-lang/crystal/pull/9009/

https://crystal-lang.org/install/from_sources/


