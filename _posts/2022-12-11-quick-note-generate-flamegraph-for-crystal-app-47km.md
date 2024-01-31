---
url: https://dev.to/miry/quick-note-generate-flamegraph-for-crystal-app-47km
canonical_url: https://dev.to/miry/quick-note-generate-flamegraph-for-crystal-app-47km
title: 'Quick note: Generate flamegraph for Crystal app'
slug: quick-note-generate-flamegraph-for-crystal-app-47km
description: Quick note for me to remember how to generate flamegraphs to profile
  Crystal lang app. Let’s see how...
tags:
- performance
- crystal
- crystallang
author: Michael Nikitochkin
username: miry
---

![Cover image](/assets/2022-12-11-quick-note-generate-flamegraph-for-crystal-app-47km-cover_image-https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fx40vzxdaox83rd30r71b.png)

# Quick note: Generate flamegraph for Crystal app


*Quick note for me to remember how to generate flamegraphs to profile Crystal lang app. Let’s see how it works…*

## **Flamegraph?**

* [Flamegraph](https://github.com/brendangregg/FlameGraph)
* [Memory Leak (and Growth) Flame Graphs](https://www.brendangregg.com/FlameGraphs/memoryflamegraphs.html)

## Installation

```shell
$ brew install flamegraph
$ ls /opt/homebrew/Cellar/flamegraph/1.0_1/bin
difffolded.pl                pkgsplit-perf.pl             stackcollapse-elfutils.pl    stackcollapse-instruments.pl stackcollapse-perf-sched.awk stackcollapse-recursive.pl   stackcollapse-vsprof.pl
files.pl                     range-perf.pl                stackcollapse-gdb.pl         stackcollapse-jstack.pl      stackcollapse-perf.pl        stackcollapse-sample.awk     stackcollapse-vtune.pl
flamegraph.pl                stackcollapse-aix.pl         stackcollapse-go.pl          stackcollapse-ljp.awk        stackcollapse-pmc.pl         stackcollapse-stap.pl        stackcollapse.pl
```

## **Profile**

Run application and extract Process ID(PID), that requires to record data.
For example I am looking to debug my app that has name `crystal-run`:

```shell
$ sudo dtrace -x ustackframes=100 -n 'profile-97 /pid == $(pgrep -n crystal-run)/ { @[ustack()] = count(); } tick-60s { exit(0); }' -o out.user_stacks
$ stackcollapse.pl out.user_stacks | flamegraph.pl > user.svg
```



