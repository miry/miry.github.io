---
url: https://medium.com/notes-and-tips-in-full-stack-development/war-with-ads-and-trackers-d3787e34750e
canonical_url: https://medium.com/notes-and-tips-in-full-stack-development/war-with-ads-and-trackers-d3787e34750e
title: War With Ads and Trackers
subtitle: My story begins with wish to see pure content of a new page in 2 seconds
  many many years ago. As young padawan I tried with small stuff —…
slug: war-with-ads-and-trackers
description: ""
tags:
- ad-blocking
- gist
author: Michael Nikitochkin
username: miry
---

# War With Ads and Trackers

My story begins with wish to see pure content of a new page in 2 seconds many many years ago. As young *padawan* I tried with small stuff — **Disabled Flash and Java**. In that period most sites have animated flash elements.

Next **second step** was **Install AdBlock Addon**. I tried multiple solutions. One of the best was [**Ghostery**](https://www.ghostery.com/) at least on that period. It was a bit different from others. It can detect social networks integrations and other trackers. I used it for some period.

I was growing up. Started use services to keep secrets and financial information. That is potential could be available from the browser addons. Besides I found that not all pages load correctly with enabled ad blocker addons. **Third my step** to use **DNS** from the my favorite router [**MikroTik**](https://mikrotik.com/) and programmatically add new rules.

```
/ip dns static add address=127.0.0.1 ttl=600w name=".*\\.vk\\.com"
/ip dns static add address=127.0.0.1 ttl=600w name="rs.mailer.ru"
/ip dns static add address=127.0.0.1 ttl=600w name="googleads.g.doubleclick.net"
/ip dns static add address=127.0.0.1 ttl=600w name="pagead2.googlesyndication.com"
/ip dns static add address=127.0.0.1 ttl=600w name="www.googletagservices.com"
/ip dns static add address=127.0.0.1 ttl=600w name="a.pr.dotua.org"
/ip dns static add address=127.0.0.1 ttl=600w name="google-analytics.com"
/ip dns static add address=127.0.0.1 ttl=600w name=".*\\.google-analytics\\.com"
/ip dns static add address=127.0.0.1 ttl=600w name="mc.yandex.ru"
/ip dns static add address=127.0.0.1 ttl=600w name="share.yandex.ru"
/ip dns static add address=127.0.0.1 ttl=600w name="vk.com"
/ip dns static add address=127.0.0.1 ttl=600w name="yandex.st"
/ip dns static add address=127.0.0.1 ttl=600w name="ad.admixer.net"
/ip dns static add address=127.0.0.1 ttl=600w name="counter.marketgid.com"
/ip dns static add address=127.0.0.1 ttl=600w name="ads.adfox.ru"
/ip dns static add address=127.0.0.1 ttl=600w name="fbstatic-a.akamaihd.net"
/ip dns static add address=127.0.0.1 ttl=600w name="content.adfox.ru"
/ip dns static add address=127.0.0.1 ttl=600w name="an.yandex.ru"
/ip dns static add address=127.0.0.1 ttl=600w name="log.pinterest.com"
/ip dns static add address=127.0.0.1 ttl=600w name="r.mailer.ru"
/ip dns static add address=127.0.0.1 ttl=600w name="rs.mailer.ru"
/ip dns static add address=127.0.0.1 ttl=600w name=".*\\.addthis\\.com"
/ip dns static add address=127.0.0.1 ttl=600w name=".*\\.adexprt\\.com" 
/ip dns static add address=127.0.0.1 ttl=600w name=".*\\.exoclick\\.com"
/ip dns static add address=127.0.0.1 ttl=600w name="mr.mention.net"
```

It was good solution, it supports wildcard domain names, Web interface and Telnet. I setup VPN tunnel to the router and route traffic through it. It works awesome. As **Ghostery** idea, block most of trackers and analytics domains. One minus of this solution: you could not block specific place on the page as **AdBlocker Addon**. If a service wrap all ads elements through its domain, this solution was useless.

Fourth solution came after I change the office and could not control a router in the new place. To keep similar idea to have own file with rules. Only `/etc/hosts` came to my mind. This solution does not have sync between hosts and any UI. So here is a sync script :

```
# Download current state of the gist. Concatenate the output of the hosts and gist
{ curl -s "https://gist.githubusercontent.com/miry/2a7635beaa7078e0d7af/raw/hosts" & cat /etc/hosts ; } | sort | uniq | grep '127.0.0.1' | grep -v -e '(^\s*$\|localhost\|broadcasthost\|#)' | tee /tmp/hosts | gist -u 2a7635beaa7078e0d7af -f hosts

# /etc/hosts should put blocked part between 2 lines of `#ADBLOCKER`
{ sed '/\#ADBLOCKER/,/\#ADBLOCKER/d' /etc/hosts;  echo '#ADBLOCKER';  cat /tmp/hosts; echo '#ADBLOCKER' ; } | sudo tee /etc/hosts
```

[Explainshell](http://explainshell.com/explain?cmd=%7B+curl+-s+%22https%3A%2F%2Fgist.githubusercontent.com%2Fmiry%2F2a7635beaa7078e0d7af%2Fraw%2Fhosts%22+%26+cat+%2Fetc%2Fhosts+%3B+%7D+%7C+sort+%7C+uniq+%7C+grep+%27127.0.0.1%27+%7C+grep+-v+-e+%27%28%5E%5Cs*%24%5C%7Clocalhost%5C%7Cbroadcasthost%5C%7C%23%29%27+%7C+tee+%2Ftmp%2Fhosts+%7C+gist+-u+2a7635beaa7078e0d7af+-f+hosts) for this command. In few words:

* Get all blacklisted domains from the `/etc/hosts`

* Download recent blacklisted domains from the [gist](https://gist.github.com/miry/2a7635beaa7078e0d7af)

* Merge 2 streams

* Update gist and local `/etc/hosts`

*Run latest sync command:*

```
$ curl https://gist.githubusercontent.com/miry/2a7635beaa7078e0d7af/raw/sync.sh  | bash
```

### Small benchmarks:

![](/assets/2017-03-20-war-with-ads-and-trackers-1_dSCS73Py-JrJujQyLTppHA.png)

35 seconds require to open main page. My CPU fan’s noise increased, because of number of JS programming.

![](/assets/2017-03-20-war-with-ads-and-trackers-1_1qWzBUMQIHYC4wiu4efIhw.png)

With blacklisted domains it show 10x times faster, and no waste of CPU energy or networking.

I am still looking for perfect simple solution to block ads for my phones and laptops without extra services. I understand the services/publishers that want to make money on old school advertisements — it is simple and cheaper.

> Let’s save the energy and build Green services without Ads.


