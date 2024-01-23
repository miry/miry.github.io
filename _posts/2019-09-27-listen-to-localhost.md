---
url: https://medium.com/notes-and-tips-in-full-stack-development/listen-to-localhost-eb0c2e36e1f5
canonical_url: https://medium.com/notes-and-tips-in-full-stack-development/listen-to-localhost-eb0c2e36e1f5
title: Listen to localhost
subtitle: How many processes can listen on same port in localhost? I know the answer
  from school time, and it is ONE port per application.
slug: listen-to-localhost
description: How many processes can listen on the same port in the localhost? Depends
  on OS configuration and socket options, you could run multiple apps on the localhost
  with the same port.
tags:
- tcp
- crystal-lang
- learning
- learning-to-code
author: Michael Nikitochkin
username: miry
---

# Listen to localhost

![Photo by Danielle MacInnes on Unsplash](/assets/2019-09-27-listen-to-localhost-1_sI6NGALyiE-KR11LR2Yi1g.png)

How many processes can ‘listen to’ the same port of *localhost*? I know the answer, and it is **ONE** port.

> ***TL;DR** Depends on OS configuration and socket options, you could run multiple apps on* localhost *with the same port.*

Run a server [**Crystal**](https://crystal-lang.org/) application to listen to port 3000

```
[terminal 1] $ cat <<CODE | crystal eval
require "socket"
s = TCPServer.new("localhost", 3000)
while c = s.accept?
   c << "pong\n"
   c.close()
end
CODE
```

*I shrank it into one command, so you can start doing experiments right now (if you have [Crystal installed](https://crystal-lang.org/reference/installation/), of course). Feel free to save the code to a file and run it via `crystal run`.*

Current sample starts a *TCP* server application. On each connection prints a message and closes a connection. 
To verify that the server is working, try to open a client connection in a new terminal:

```
[terminal 2] $ nc localhost 3000
pong
```

**L**ooks **G**ood **T**o **M**e. Nothing suspicious. Let’s continue our crash course.

What if I run another *TCP* server on the *localhost* and use the same port? Should I get a runtime error?
In the new terminal window, run the same code:

```
[terminal 2] $ cat <<CODE | crystal eval
require "socket"
s = TCPServer.new("localhost", 3000)
while c = s.accept?
   c << "pong\n"
   c.close()
end
CODE
```

It works in my case! If you see an error, you can scroll to the end. The first process is still running at the same time. Okay. I’m starting getting the hang of this.

How could it be that multiple processes can listen to the same port?

It is possible, only if you set socket option `**SO_REUSEPORT**`. It works differently and depends on the OS. On Mac OS you can check socket options with `lsof` command with the`-Tf` argument:

```
[terminal 3] $ lsof -nP -iTCP:3000 -Tf
COMMAND PID USER FD TYPE NODE NAME
```

```
crystal 8890 m 11u  IPv6 TCP [::1]:3000 (SO=ACCEPTCONN,...)
```

```
crystal 8920 m 11u  IPv6 TCP [fe80:1::1]:3000 (SO=ACCEPTCONN,...)
```

There is no `**SO_REUSEPORT**` in the list of socket options. Let’s continue to run more servers, and see if there are some limits.

```
[terminal 3] $ cat <<CODE | crystal eval
require "socket"
s = TCPServer.new("localhost", 3000)
while c = s.accept?
   c << "pong\n"
   c.close()
end
CODE
```

It is running well as hell. Next one:

```
[terminal 4] $ cat <<CODE | crystal eval
require "socket"
s = TCPServer.new("localhost", 3000)
while c = s.accept?
   c << "pong\n"
   c.close()
end
CODE
```

```
Unhandled exception: bind: Address already in use (Errno)
  from TCPServer#initialize<String, Int32>:Nil
  from TCPServer::new<String, Int32>:TCPServer
  from __crystal_main
  from Crystal::main_user_code<Int32, Pointer(Pointer(UInt8))>:Nil
  from Crystal::main<Int32, Pointer(Pointer(UInt8))>:Int32
  from main
```

Great, there is the expected error. It means — no more new servers.

Why can I run 3 servers on *localhost*? What happened with the 4th instance?

To answer those questions, we can analyze the passive sockets for incoming connections. Run our `lsof`, for the current scope of applications:

```
[terminal 4] $ lsof -nP -iTCP:3000 -Tf
COMMAND PID  USER FD TYPE NODE NAME
crystal 9573 m   11u  IPv6 TCP [::1]:3000 (SO=ACCEPTCONN,...)
crystal 9638 m   11u  IPv6 TCP [fe80:1::1]:3000 (SO=ACCEPTCONN,..)
crystal 9835 m   11u  IPv4 TCP 127.0.0.1:3000 (SO=ACCEPTCONN,...)
```

The difference between all three lines is the **address**. The same line of code `TCPServer.new(“localhost”, 3000)` produces the following addresses: `[::1]:3000`, `[fe80:1::1]:3000` and `127.0.0.1:3000`.

### What are these addresses?

*Localhost* is not an interface or address. It is a standard *hostname*, that resolves to *IP* addresses assigned to the [loopback device](https://en.wikipedia.org/wiki/Loop_device) with name `lo0` in my case. To show thelistening addresses for our server we need to stop all currently running servers and run the next code snippet in each terminal:

```
[terminal *] $ cat <<EOF | crystal eval
require "socket"
s = TCPServer.new("localhost", 3000)
puts "Listen on #{s.local_address}"
while c = s.accept?
   c << "pong from #{s.local_address}\n"
   c.close()
end
EOF
```

In the output, you can see the servers’ addresses. My *OS* has *IPv6* enabled, so I have 2 extra *IP* addresses for *localhost*. Amazing, that in old times we have only one *IP* address assigned to *localhost* — `127.0.0.1`. That’s why I remember the rule of **ONE**.

### Clients

What could *TCP* clients say about our servers? In the new terminal run a standard *TCP* client tool `nc`:

```
[terminal 4] $ nc localhost 3000
pong from [::1]:3000
```

It chose the same IP address again as the first server. Check connections to all servers, with a magic shell script:

```
[terminal 4] $ for ip in "127.0.0.1" "fe80:1::1" "::1"; do
                 nc $ip 3000 
               done
pong from 127.0.0.1:3000
pong from [fe80::1]:3000
pong from [::1]:3000
```

### Why the server tries to listen to the next free IP address?

What I like in **Crystal** (and of course **Golang**), is that you can read sources of standard libraries in the same language. I started my learning from [*tcp_server.cr*](https://github.com/crystal-lang/crystal/blob/master/src/socket/tcp_server.cr#L32) and in a few jumps between functions I came to [*LibC::Addrinfo.new*](https://github.com/crystal-lang/crystal/blob/master/src/socket/addrinfo.cr#L101). From this step I searched for man page of [**getaddrinfo(3)**](http://man7.org/linux/man-pages/man3/getaddrinfo.3.html)

> The **getaddrinfo**() function allocates and initializes a **linked list** of
 addrinfo structures, one for each network address that matches node
 and service, subject to any restrictions imposed by hints, and
 returns a pointer to the start of the list in res. The items in the
 linked list are linked by the **ai_next** field.

> There are several reasons why the **linked list** may have more than one
 **addrinfo** structure, including: the network host is multihomed, acces‐
 sible over multiple protocols (e.g., both **AF_INET** and **AF_INET6**); or
 the same service is available from multiple socket types (one
 **SOCK_STREAM** address and another **SOCK_DGRAM** address, for example).
 Normally, the application should try using the addresses in the order
 in which they are returned. The sorting function used within **getad**‐
 **drinfo**() is defined in **RFC 3484**; the order can be tweaked for a par‐
 ticular system by editing /etc/gai.conf (available since glibc 2.5).
 — **getaddrinfo(3) — Linux manual page**

It means, the libc function returns a [linked list](https://www.geeksforgeeks.org/data-structures/linked-list/) of all addresses, and an application should decide (ideally use the same order) which one of the addresses to choose. 
That’s why we saw, the command `nc localhost 3000` connected to the first process `pong from [::1]:3000`. 
You can try to restart the processes in order to prove this theory.

Example: Stop the first process, and then try again to run `nc localhost 3000`.
Try to modify `/etc/hosts`: change the order of lines with *localhost* and restart the process to check a new order of addresses.

# Summary

We deal with the *localhost* every working day developing web applications. Consider the chance to have multiple processes running with the same port, as it could save a few minutes, to understand why there are no recent changes applied to an application after restarting a process.

Be accurate with the *localhost*. One of the solutions is using *IP* address `127.0.0.1` instead of `localhost`.

![That’s all Folks](/assets/2019-09-27-listen-to-localhost-1_iifqfnqorqkZVMpCyq1BjA.png)

**Michael Nikitochkin** *is a Lead Software Engineer. Follow him on [LinkedIn](https://www.linkedin.com/in/michaelnikitochkin/) or [GitHub](https://github.com/miry).*

> If you enjoyed this story, we recommend reading our [latest tech stories](https://jtway.co/latest) and [trending tech stories](https://jtway.co/trending).

# References

https://crystal-lang.org/api/0.31.0/Socket/Addrinfo.html

http://man7.org/linux/man-pages/man3/getaddrinfo.3.html

https://github.com/crystal-lang/crystal/blob/master/src/socket/tcp_server.cr


