---
url: https://medium.com/notes-and-tips-in-full-stack-development/netcat-with-ssh-port-forwarding-148177b2e850
canonical_url: https://medium.com/notes-and-tips-in-full-stack-development/netcat-with-ssh-port-forwarding-148177b2e850
title: Netcat with SSH Port Forwarding
subtitle: I have encountered a problem with getting access to the private local service
  which available from the local machine. And I need to write…
slug: netcat-with-ssh-port-forwarding
description: ""
tags:
- docker
- tutorial
- linux
- terminal
author: Michael Nikitochkin
username: miry
---

![](/assets/2017-06-16-netcat-with-ssh-port-forwarding-1_XFJSnXub8gzTQlw4iuRkWQ.jpeg)

# Netcat with SSH Port Forwarding

I have encountered a problem with getting access to the private local service which available from the local machine. And I need to write application using this service in another network. I have only one port open to access the local machine `ssh` daemon. After a few minutes of problem discussion, we decided to use `NetCat`.

```
$ mkfifo pipe
$ while [ 1 ]; do nc -l -p 8080 < pipe | ssh gw_to_private_net \ -p 22977  "nc 192.168.12.230 80" | tee pipe; done
```

So we need run this only on local machine. Now we have the open local port `8080` forwarded to private machine `192.168.12.230:80`.

Let me describe these commands

First open a local port to hear from our network connections. So this port will be proxy the data to private net:

```
$ nc -l -p 8080
```

Next step is to connect is to login to private getaway host via `ssh` and open connection to local private host with ip `192.168.12.230` on port `80`:

```
$ ssh gw_to_private_net -p 22977  "nc 192.168.12.230 80"
```

So what we have. We can read data from local port to output and we can write data to remote host. Let’s combine these commands:

```
$ nc -l -p 8080 | ssh gw_to_private_net -p 22977 \
 "nc 192.168.12.230 80"
```

After running this command we can send request to the local port `8080` and see a response from private host in the output. It is not that we wanted, lets redirect the response to local connection. We use for this `pipe` or Unix sockets.

```
$ mkfifo pipe; nc -l -p 8080 < pipe | ssh gw_to_private_net \ 
-p 22977  "nc 192.168.12.230 80" | tee pipe
```

Wow, that works, but after each request the connection gets closed. That’s why we need to reconnect each time via simple loop.

```
$ mkfifo pipe
$ while [ 1 ]
> do
>   nc -l -p 8080 < pipe | ssh gw_to_private_net \
-p 22977  "nc 192.168.12.230 80" | tee pipe
> done
```

![That’s all Folks](/assets/2017-06-16-netcat-with-ssh-port-forwarding-1_iifqfnqorqkZVMpCyq1BjA.png)

**Michael Nikitochkin** *is a Lead Software Engineer. Follow him on [LinkedIn](https://www.linkedin.com/in/michaelnikitochkin/) or [GitHub](https://github.com/miry).*

> If you enjoyed this story, we recommend reading our [latest tech stories](https://jtway.co/latest) and [trending tech stories](https://jtway.co/trending).


