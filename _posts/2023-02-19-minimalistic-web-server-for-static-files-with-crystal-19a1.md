---
url: https://dev.to/miry/minimalistic-web-server-for-static-files-with-crystal-19a1
canonical_url: https://dev.to/miry/minimalistic-web-server-for-static-files-with-crystal-19a1
title: Minimalist web server for static files with Crystal
slug: minimalistic-web-server-for-static-files-with-crystal-19a1
description: Problem   Build a cross-platform web server to serve static files in
  the local folder.  Test...
tags:
- crystal
- crystallang
- howto
author: Michael Nikitochkin
username: miry
---

![Cover image](/assets/2023-02-19-minimalistic-web-server-for-static-files-with-crystal-19a1-cover_image-32ljw717osu23pim3fwb.jpg)

# Minimalist web server for static files with Crystal


## Problem

Build a cross-platform web server to serve static files in the local folder. 
Test local HTML, CSS, and JS files without CORS errors[^1].

> Many browsers, including Firefox and Chrome, now treat all local files as having opaque origins (by default). As a result, loading a local file with included local resources will result in CORS errors. 

## Solutions

The standard [Crystal] library allows an HTTP server to process requests[^2].

### Standard

Run a server on `http://127.0.0.1:8080` to serve files from the local directory.

```crystal
# server.cr
require "http/server"

def run(host="127.0.0.1", port=8080, local=".")
	server = HTTP::Server.new([
	  HTTP::ErrorHandler.new,
	  HTTP::LogHandler.new,
	  HTTP::CompressHandler.new,
	  HTTP::StaticFileHandler.new(local),
	])

	address = server.bind_tcp host, port
	puts "Listening on http://#{address}"
	server.listen
end

run
```

```shell
$ crystal run server.cr
Listening on http://127.0.0.1:8080
```

### Minimalist

Of course, you want to run it everywhere without creating additional files:

```shell
$ crystal eval -p 'require "http/server"; HTTP::Server.new([HTTP::StaticFileHandler.new(".")]).listen(8080)'
```

## References

[^1]: [Reason: CORS request not HTTP: Loading a local file](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS/Errors/CORSRequestNotHttp#loading_a_local_file)
[^2]: [Crystal: class HTTP::Server](https://crystal-lang.org/api/1.7.2/HTTP/Server.html)

[Crystal]: https://crystal-lang.org/


