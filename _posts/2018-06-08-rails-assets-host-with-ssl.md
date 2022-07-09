---
url: https://medium.com/notes-and-tips-in-full-stack-development/rails-assets-host-with-ssl-a843ded59ce4
canonical_url: https://medium.com/notes-and-tips-in-full-stack-development/rails-assets-host-with-ssl-a843ded59ce4
title: Rails Assets Host Trick
subtitle: When we use simple assets server, such as
slug: rails-assets-host-with-ssl
description: ""
tags:
- rails
- assets
- tutorial
- ruby
author: Michael Nikitochkin
username: miry
---

# Rails Assets Host Trick

When we use simple assets server, such as

```
ActionController::Base.asset_host = "http://assets.example.com"
```

we have a problem with pages which use *SSL* and some browsers alert users that some content is not safe.

So I found in google the following solution:

```
ActionController::Base.asset_host = Proc.new { |source| "//assets%d.example.com" % (source.hash % 4) }
```

It is beautiful, because for each image we use only one host, so when you refresh a page, an image always has an example host `*assets1*` on each page, not some random host.


