---
url: https://medium.com/notes-and-tips-in-full-stack-development/dockerfile-for-a-crystal-application-1e9db24efbc2
canonical_url: https://medium.com/notes-and-tips-in-full-stack-development/dockerfile-for-a-crystal-application-1e9db24efbc2
title: Dockerfile for a Crystal application
subtitle: Build a space-effective docker image for a Crystal application
slug: dockerfile-for-a-crystal-application
description: ""
tags:
- crystal-lang
- docker
- alpine
author: Michael Nikitochkin
username: miry
---

# Dockerfile for a Crystal application

### Build a space-effective docker image for a Crystal application

> TL;DR: Build an application with static linking and use Alpine base image to reduce the size of the Docker image.

The building process of a [**Crystal**](https://crystal-lang.org/) app requires the installation of additional lib packages. Even the result binary is small, the docker image is *significant*.

![Photo by Luke Stackpoole on Unsplash](/assets/2021-04-18-dockerfile-for-a-crystal-application-1_M8N2KFnZeTJCKeLhRXwayw.jpeg)

A straightforward solution is to build a binary with all required libraries, a.k.a. Static Linking. It would allow skipping the installation of extra packages.

Crystal supports [Static Linking](https://crystal-lang.org/reference/guides/static_linking.html), but it does not work with `glibc` (used in mostly all Linux distributives). To build an application with static linking, add the option `--static` .

So I changed the base image from **Ubuntu** to [**Alpine**](https://alpinelinux.org/). [**Alpine**](https://alpinelinux.org/) uses `musl` instead of `glibc`.

# Dockerfile

Here is my version of `Dockerfile` base on Alpine:

```
# Build image
FROM crystallang/crystal:1.0.0-alpine as builder
```

```
WORKDIR /app
```

```
# Cache dependencies
COPY ./shard.yml ./shard.lock /app/
RUN shards install --production -v
```

```
# Build a binary
COPY . /app/
RUN shards build --static --no-debug --release --production -v
```

```
# ===============
# Result image with one layer
FROM alpine:latest
WORKDIR /
COPY --from=builder /app/bin/medup .
```

```
ENTRYPOINT ["/medup"]
```

# Results

**DockerHub’**s [information](https://hub.docker.com/r/miry/medup/tags?page=1&ordering=last_updated) about the image size (probably it shows the size of new layers):

![](/assets/2021-04-18-dockerfile-for-a-crystal-application-1_jcVQmXInqs-ULf4-A8cGSQ.png)

Here are the sizes of images from my local machine:

```
$ docker images
REPOSITORY TAG IMAGE ID CREATED SIZE
miry/medup alpine b673ccf47be5 53 minutes ago 23.1MB
miry/medup ubuntu 7d692dcd0711 7 months ago 439MB
```

As you see, the size of the **Ubuntu** version was *400MB*, when the **Alpine** version is just *23MB*.

# References

https://crystal-lang.org/2020/02/02/alpine-based-docker-images.html

https://crystal-lang.org/reference/guides/static_linking.html

https://alpinelinux.org/

https://github.com/miry/medup/


