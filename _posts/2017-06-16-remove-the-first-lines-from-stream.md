---
url: https://medium.com/notes-and-tips-in-full-stack-development/remove-the-first-lines-from-stream-83350e049166
canonical_url: https://medium.com/notes-and-tips-in-full-stack-development/remove-the-first-lines-from-stream-83350e049166
title: Remove the First Lines from Stream
subtitle: I am not a linux hacker, so I wasted a lot of time trying to find a solution
  to strip first lines from output stream. The first my solution…
slug: remove-the-first-lines-from-stream
description: ""
tags:
- linux
author: Michael Nikitochkin
username: miry
---

# Remove the First Lines from Stream

I am not a linux hacker, so I wasted a lot of time trying to find a solution to strip first lines from output stream. The first my solution was the following:

```
$ tail -f some_file | ruby -e \
> 'a =0; while t=gets; a+=1; puts t if a > 1; end'
```

It looks very long, and I thought that this problem is very popular, and at least one tool already exists in the world.

I knew a tool **sed** and have used it before. So I have read the manual and voilà:

```
$ tail -f some_file | sed "1d"
```

Remove the first 10 lines:

```
$ tail -f some_file | sed '1,10d'
```

It does the same thing as the first solution, but it is more clear and simple. **sed** is a great tool.


