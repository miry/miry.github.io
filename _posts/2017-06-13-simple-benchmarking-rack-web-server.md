---
url: https://jtway.co/simple-benchmarking-rack-web-server-abe6a6f974f6
canonical_url: https://jtway.co/simple-benchmarking-rack-web-server-abe6a6f974f6
title: Simple Benchmarking Rack Web Server
subtitle: Before each project we check new versions of favorite Ruby web servers and
  choose which is the best.
slug: simple-benchmarking-rack-web-server
description: ""
tags:
- ruby
author: Michael Nikitochkin
username: miry
---

# Simple Benchmarking Rack Web Server

![][image_ref_MSpZT1VrZHNYUXAwWTZ2bnpUTjBkOFdBLmpwZWc=]

Before each project we check new versions of favorite Ruby web servers and choose which is the best.

Let’s use, for example, [Geminabox](http://tomlea.co.uk/posts/gem-in-a-box/) application to distribute gems in the local network for our company. [Geminabox](http://tomlea.co.uk/posts/gem-in-a-box/) is a very tiny Rack application and it is easy to configure. You can get the sample [here](https://github.com/miry/geminabox_web_server).

There are a lot of Ruby Web Servers. More details you can get from Digital Ocean article [A Comparison of (Rack) Web Servers for Ruby Web Applications](https://www.digitalocean.com/community/tutorials/a-comparison-of-rack-web-servers-for-ruby-web-applications). I excluded **Passenger** because it requires to compile Nginx module and **Thin** — it could not use all CPU cores.

# Test environment:

* Hardware: MacMini Late 2012 (16 GB 1333 MHz DDR3, 2.5 GHz Intel Core i5)

* OS: OS X 10.9.4 (13E28)

* Ruby versions: ruby-2.2.0-dev, jruby-1.7.9

* Benchmark tool: [Siege](http://linux.die.net/man/1/siege)

* Web servers: [Puma](http://puma.io), [Unicorn](http://unicorn.bogomips.org), [TorqueBox](http://torquebox.org)

# Benchmarking

Before we go we should install all ruby versions and required gems. I use [rbenv](http://rbenv.org) to manage multiple ruby versions in one host.

```
$ rbenv install 2.2.0-dev
$ rbenv shell 2.2.0-dev
$ gem install bundler puma unicorn 
$ rbenv install jruby-1.7.9
$ rbenv shell jruby-1.7.9
$ gem install bundler puma torquebox
```

Prepare the application:

```
$ git clone http://github.com/miry/geminabox_web_server
$ cd geminabox_web_server
```

# Unicorn + Ruby

Run Rack application:

```
$ rbenv shell 2.2.0-dev
$ sudo WEB_CONCURRENCY=5 unicorn -c config/unicorn.rb
```

*Siege report for 5 concurrencies:*

```
Transactions:		          50 hits
Availability:		      100.00 %
Elapsed time:		       15.83 secs
Data transferred:	       30.93 MB
Response time:		        0.86 secs
Transaction rate:	        3.16 trans/sec
Throughput:		        1.95 MB/sec
Concurrency:		        2.72
Successful transactions:          50
Failed transactions:	           0
Longest transaction:	        1.20
Shortest transaction:	        0.58
```

*Siege report for 25 concurrencies:*

```
Transactions:		         250 hits
Availability:		      100.00 %
Elapsed time:		       63.15 secs
Data transferred:	      154.66 MB
Response time:		        5.49 secs
Transaction rate:	        3.96 trans/sec
Throughput:		        2.45 MB/sec
Concurrency:		       21.73
Successful transactions:         250
Failed transactions:	           0
Longest transaction:	        6.58
Shortest transaction:	        1.36
```

# Puma + Ruby

# With 5 workers

Run Rack application:

```
$ rbenv shell 2.2.0-dev
$ sudo puma config.ru -t 4:32 -p 80 -w 5
```

CPU was loaded at near 360%.

*Siege report for 5 concurrencies:*

```
Transactions:		          50 hits
Availability:		      100.00 %
Elapsed time:		       15.30 secs
Data transferred:	       30.93 MB
Response time:		        0.81 secs
Transaction rate:	        3.27 trans/sec
Throughput:		        2.02 MB/sec
Concurrency:		        2.65
Successful transactions:          50
Failed transactions:	           0
Longest transaction:	        1.64
Shortest transaction:	        0.46
```

*Siege report for 25 concurrencies:*

```
Transactions:		         250 hits
Availability:		      100.00 %
Elapsed time:		       61.02 secs
Data transferred:	      154.66 MB
Response time:		        4.78 secs
Transaction rate:	        4.10 trans/sec
Throughput:		        2.53 MB/sec
Concurrency:		       19.59
Successful transactions:         250
Failed transactions:	           0
Longest transaction:	       16.57
Shortest transaction:	        0.52
```

# Puma + Ruby

# Without workers

Run Rack application:

```
$ rbenv shell 2.2.0-dev
$ sudo bundle exec puma config.ru -t 4:32 -p 80
```

CPU was loaded at 100% only

*Siege report for 5 concurrencies:*

```
Transactions:		          50 hits
Availability:		      100.00 %
Elapsed time:		       21.18 secs
Data transferred:	       30.93 MB
Response time:		        1.41 secs
Transaction rate:	        2.36 trans/sec
Throughput:		        1.46 MB/sec
Concurrency:		        3.34
Successful transactions:          50
Failed transactions:	           0
Longest transaction:	        2.37
Shortest transaction:	        0.38
```

*Siege report for 25 concurrencies:*

```
Transactions:		         250 hits
Availability:		      100.00 %
Elapsed time:		      111.67 secs
Data transferred:	      154.66 MB
Response time:		       10.55 secs
Transaction rate:	        2.24 trans/sec
Throughput:		        1.38 MB/sec
Concurrency:		       23.62
Successful transactions:         250
Failed transactions:	           0
Longest transaction:	       11.19
Shortest transaction:	        7.34
```

# Puma + JRuby

Run Rack application:

```
$ rbenv shell jruby-1.7.9
$ sudo bundle exec puma config.ru -t 4:32 -p 80
```

For first requests a response time was huge. But after the second experiment, it was decreased. During the experiments the CPU was loaded at near 350%.

*Siege report for 5 concurrencies:*

```
Transactions:		          50 hits
Availability:		      100.00 %
Elapsed time:		       10.61 secs
Data transferred:	       30.93 MB
Response time:		        0.44 secs
Transaction rate:	        4.71 trans/sec
Throughput:		        2.92 MB/sec
Concurrency:		        2.09
Successful transactions:          50
Failed transactions:	           0
Longest transaction:	        0.66
Shortest transaction:	        0.25
```

*Siege report for 25 concurrencies:*

```
Transactions:		         250 hits
Availability:		      100.00 %
Elapsed time:		       41.97 secs
Data transferred:	      154.66 MB
Response time:		        3.46 secs
Transaction rate:	        5.96 trans/sec
Throughput:		        3.68 MB/sec
Concurrency:		       20.61
Successful transactions:         250
Failed transactions:	           0
Longest transaction:	        5.83
Shortest transaction:	        0.35
```

# TorqueBox + JRuby

Run Rack application:

```
$ rbenv shell jruby-1.7.9
$ torquebox deploy
$ torquebox run
```

*Siege report for 5 concurrencies:*

```
Transactions:		          50 hits
Availability:		      100.00 %
Elapsed time:		       15.18 secs
Data transferred:	       30.93 MB
Response time:		        0.72 secs
Transaction rate:	        3.29 trans/sec
Throughput:		        2.04 MB/sec
Concurrency:		        2.37
Successful transactions:          50
Failed transactions:	           0
Longest transaction:	        1.07
Shortest transaction:	        0.39
```

*Siege report for 25 concurrencies:*

```
Transactions:		         250 hits
Availability:		      100.00 %
Elapsed time:		       63.80 secs
Data transferred:	      154.66 MB
Response time:		        5.76 secs
Transaction rate:	        3.92 trans/sec
Throughput:		        2.42 MB/sec
Concurrency:		       22.59
Successful transactions:         250
Failed transactions:	           0
Longest transaction:	        7.48
Shortest transaction:	        2.52
```

# Conclusion

After current investigation, I see that [Puma](http://puma.io) is cross platform and fast web server. So if you have not decided what ruby to choose, [Puma](http://puma.io) will be the best solution. More about benchmarking and web servers:

* [The Ruby Web Benchmark Report](http://www.madebymarket.com/blog/dev/ruby-web-benchmark-report.html)

* [A Simple Webserver Comparison](https://gist.github.com/cespare/3793565)

* [Benchmarking TorqueBox](http://torquebox.org/news/2011/02/23/benchmarking-torquebox/)

* [High Performance Ruby Part 3: non-blocking IO and web application scalability](http://blog.gregweber.info/posts/2011-06-16-high-performance-rb-part3)


[image_ref_MSpZT1VrZHNYUXAwWTZ2bnpUTjBkOFdBLmpwZWc=]: data:image/jpeg;base64,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
