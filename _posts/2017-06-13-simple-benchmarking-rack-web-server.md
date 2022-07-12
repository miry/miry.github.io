---
url: https://jtway.co/simple-benchmarking-rack-web-server-abe6a6f974f6
canonical_url: https://jtway.co/simple-benchmarking-rack-web-server-abe6a6f974f6
title: Simple Benchmarking Rack Web Server
subtitle: Before each project we check new versions of favorite Ruby web servers and choose which is the best.
slug: simple-benchmarking-rack-web-server
description: 
tags: ruby
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
