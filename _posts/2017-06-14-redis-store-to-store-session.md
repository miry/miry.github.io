---
url: https://medium.com/notes-and-tips-in-full-stack-development/redis-store-to-store-session-78e0e089d702
canonical_url: https://medium.com/notes-and-tips-in-full-stack-development/redis-store-to-store-session-78e0e089d702
title: Redis Store to Store Session
subtitle: I use Redis to store session and cache. And when I try to cache some session
  values, I get an exception “TypeError (can’t dump TCPSocket)”…
slug: redis-store-to-store-session
description: ""
tags:
- redis
- cache
author: Michael Nikitochkin
username: miry
---

# Redis Store to Store Session

I use Redis to store session and cache. And when I try to cache some session values, I get an exception “TypeError (can’t dump TCPSocket)”. I have researched this problem. It happened because session.slice(‘keys’) returns not simple Hash instance, but SessionHash. So instance method *to_hash* fixes all troubles. Example:

```
Rails.cache.write("key", session.except("flash", :session_id, :_csrf_token))
```
> *[redis-store-session_hash.rb view raw](https://gist.githubusercontent.com/marchi-martius/5dc87dcb404ad8fa1b157acab9685271/raw/b932d54db6012d412c6dbf8eaa49ea27a1d6ba1e/redis-store-session_hash.rb)*

Solution:

```
Rails.cache.write("key", session.except("flash", :session_id, :_csrf_token).to_hash)
```
> *[redis-store-hash.rb view raw](https://gist.githubusercontent.com/marchi-martius/5717e000f50c7a54b1ebbb65584fa4e7/raw/79b9941492bc5bd21ec2432808a48dfb3df7b72b/redis-store-hash.rb)*


