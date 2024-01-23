---
url: https://medium.com/notes-and-tips-in-full-stack-development/debug-crystal-tcp-server-34cb3af4fbbc
canonical_url: https://medium.com/notes-and-tips-in-full-stack-development/debug-crystal-tcp-server-34cb3af4fbbc
title: Debug Crystal TCP server
subtitle: Learn how to debug TCP communications with Wireshark
slug: debug-crystal-tcp-server
description: ""
tags:
- wireshark
- tcp
- crystal-lang
author: Michael Nikitochkin
username: miry
---

# Debug Crystal TCP server

### Learn how to debug TCP communications with Wireshark

![Photo by Samuel Austin on Unsplash](/assets/2018-11-25-debug-crystal-tcp-server-0_WfU0E4DqjNDE3QdR.jpeg)

In my journey of learning Crystal language, I want to record notes and left guide on how to use tools. I started with understanding how `TCPServer` and `TCPScoket` are working and met some unexpected for my behavior. Today I am gonna discuss how to run a sample server application, debug packets with **Wireshark** and find better way write code (at least I think so).

### Open the port

The first step is to open the port and accept connections. I found a sample TCP server from [the official documentation](https://crystal-lang.org/api/0.27.0/TCPServer.html). A bit of tuning and logging:

```
# server.cr
require "socket"
```

```
def handle_client(client)
  puts "Connected client: #{client}"
  while message = client.gets
    puts "[#{client}] Message: #{message}"
    client.puts message
  end
end
```

```
server = TCPServer.new("localhost", 3000)
while client = server.accept?
  spawn handle_client(client)
end
```

Run server application:

```
$ crystal server.cr
```

In another terminal run a client, that connects to the port `3000`:

```
$ nc localhost 3000
hello
hello
world
world
```

Amazing. A small example with a lot of possibilities. From this point, it could be upgraded to a simple chat server, remote execution of commands and etc.

### TCP network analyzer

I would like to know, how TCP is working on a new connection and try to understand all this Voodoo. From previous years, I heard about `tcpdump` and `wireshark` tools for this needs. I’d like to play with `wireshark`.

```
$ brew cask install wireshark
```

Run **Wireshark** and add a filter to trace all TCP packets from port `3000`: `tcp.port == 3000`. In the terminal, start the server and client in the background.

![Initial connection](/assets/2018-11-25-debug-crystal-tcp-server-1_QiiPK1kJpNVn1z0kzPl7Uw.png)

There are 3 handshake packets and last packet to change [TCP Window size](https://en.wikipedia.org/wiki/TCP_tuning#Window_size) from 65,535 to 407,776.

### Send and Receive data

Let’s send 2 bytes from the client to the server, and get a response. **Wireshark** shows 6 packets captured. Hm, I expected only 4.

![Send and Receive packages](/assets/2018-11-25-debug-crystal-tcp-server-1__KjzMmxLgE7Sc3GiG9e3rQ.png)

First package `TCP 78 57464 → 3000 [PSH, ACK] Seq=1 Ack=1 Win=407776 Len=2` has data: `310a` that represents character `1` and new line. Next package from the server `TCP 76 3000 → 57464 [ACK] Seq=1 Ack=3 Win=407776 Len=0` confirms to the client, that the package with `Seq=1` was received. 4 other packets related to the server’s response with the same behavior. But why 4? After analyzing the first package’s data, found that it send just character `1` and in next send package there is a new line byte.

I would like to send a message in one packet. Started investigation how TCP communication implemented in Crystal. Maybe after reading the sources will find a solution.

According to the [doc](https://crystal-lang.org/api/0.27.0/TCPServer.html#accept%3F-instance-method) and [sources](https://github.com/crystal-lang/crystal/blob/e9a1158e40280eb27657c2938aaf7a690a614a31/src/socket/tcp_server.cr#L103) `accept?` returns [TCPSocket](https://crystal-lang.org/api/0.27.0/TCPSocket.html) object. From the class documentation, there is a sample code how to send data to a socket. Let’s change `puts` to `<<` with one string.

```
def handle_client(client)
  puts "Connected client: #{client}"
  while message = client.gets
    puts "[#{client}] Message: #{message}"
    client << "#{message}\n"
  end
end
```

After restart server and client applications, start the capture of packages again:

![Capture Send and Receive packets with Wireshark](/assets/2018-11-25-debug-crystal-tcp-server-1_WszteJTtMYxV7kXXaj13Yg.png)

Now it looks better — we transfer a few bytes less. In the same interpolation uses `String.build` that is a good approach by [the doc](https://github.com/crystal-lang/crystal-book/blob/master/guides/performance.md#use-string-interpolation-instead-of-concatenation). `IO.puts` is slower, according to [the source](https://github.com/crystal-lang/crystal/blob/c9d1eef8fde5c7a03a029d64c8483ed7b4f2fe86/src/io.cr#L226). It invokes 2 `puts` one with string and another on an empty line.

### Close connection

When a client is closing the connection, it sends `FIN,ACK` packet and the server send back `ACK` packet, as confirmation for closing socket.

![Open and close connection](/assets/2018-11-25-debug-crystal-tcp-server-1_HbH-lzpglsLeZAucb9CcqQ.png)

The sockets changed state from `ESTABLISHED` to `CLOSE_WAIT` and waits for the packet `FIN`. For **MacOS** there is command to check all connection by port: `lsof -nP -iTCP:3000`. One socket is closed and disappears, but connection from the server to the exited client still exists with state `CLOSE_WAIT` . It means we did not close correctly connection from the server side. After modification of the client handler function:

```
def log(message, src)
  puts "[#{Time.now}] [#{src}] #{message}"
end
```

```
def handle_client(client)
  log "Client is connected: #{client}", "server"
  while message = client.gets
    log "Message: #{message}", client
    client << "#{message}\n"
  end
  log "Closing client's connection: #{client}", "server"
  client.close
end
```

![Properly closed all connections](/assets/2018-11-25-debug-crystal-tcp-server-1_0Ijw_g3tQCcW3ugBwPm9BQ.png)

### Conclusion

Now we learned how to build a server application, without big problems and time read Crystal implementation of different methods and classes from the documentation and sources. Something new for me was to use **Wireshark** to understand TCP communication between applications. All source code you can find in [Github](https://github.com/miry/samples/blob/master/experiments/1-tcp-listen/server.cr).

![That’s all Folks](/assets/2018-11-25-debug-crystal-tcp-server-1_iifqfnqorqkZVMpCyq1BjA.png)

**Michael Nikitochkin** *is a Lead Software Engineer. Follow him on [LinkedIn](https://www.linkedin.com/in/michaelnikitochkin/) or [GitHub](https://github.com/miry).*

> If you enjoyed this story, we recommend reading our [latest tech stories](https://jtway.co/latest) and [trending tech stories](https://jtway.co/trending).

### References

https://www.linuxjournal.com/files/linuxjournal.com/linuxjournal/articles/023/2333/2333s2.html

https://crystal-lang.org/api/0.27.0/TCPServer.html

https://crystal-lang.org/docs/guides/concurrency.html

https://wiki.wireshark.org/DisplayFilters

https://github.com/miry/samples/blob/master/experiments/1-tcp-listen/server.cr


