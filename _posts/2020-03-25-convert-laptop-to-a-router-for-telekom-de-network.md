---
url: https://medium.com/notes-and-tips-in-full-stack-development/convert-laptop-to-a-router-for-telekom-de-network-21bd2f9a0ef3
canonical_url: https://medium.com/notes-and-tips-in-full-stack-development/convert-laptop-to-a-router-for-telekom-de-network-21bd2f9a0ef3
title: Convert laptop to a router for Telekom.de network
subtitle: Set up a Linux machine as a WiFi router in a few steps. No luck with Mac
  OS devices.
slug: convert-laptop-to-a-router-for-telekom-de-network
description: ""
tags:
- linux
- telekom
- pppoe
- internet
- router
author: Michael Nikitochkin
username: miry
---

![Photo by Estée Janssens on Unsplash](/assets/2020-03-25-convert-laptop-to-a-router-for-telekom-de-network-0_PHf8wHoP_11uD31N.jpeg)

# Convert laptop to a router for Telekom.de network

Set up a Linux machine as a WiFi router in a few steps. No luck with Mac OS devices.

> TL;DR I spent half of the day to collect required information about Telekom and recover my knowledge to setup an Internet connection. Came to the command line(available in the Result section of the post).

### Requirements

**Telekom.de**

* **VLAN** with TAG ID **7**

* **PPPoE** for authorization
- PAP CHAP **Username**: Mitbenutzernummer (it could be 0001 or [<anschlusskennung>#<zugangsnummer>#<mitbenutzernummer>@t-online.de](mailto:002803261498#551009405383#0001@t-online.de))
- PAP-CAHP **Password**: Kennwort (8 digits)

[Deutsch guide](https://www.telekom.de/hilfe/downloads/octopus-f50-bng-adsl.pdf) how to configure a router for Telekom.de.

I have straightforward local requirements: Support mobile devices to connect the Internet via WiFi.

I am going to skip all my experiments with MacOS, AirPort to have PPPoE over VLAN working. Let’s focus on my small setup of the Linux machine (laptop) with Fedora 31.

### Create a test internet link

My first attempt to use **GUI** applications to set up a new **PPPoE** connection. In the **Gnome** **Settings**, there is no option to add **VLAN** connections or **PPPoE**.

I went hardcore and checked how to setup **VLAN** via shell:

```
$ sudo ip link add link enp0s31f6  name enp0s31f6.7 type vlan id 7
```

Where `enp0s31f6` is the name of my ethernet device and `enp0s31f6.7` is the name of the new device with **VLAN** tag **ID 7**.

Usually, I use a “*brute force*” method as the leading technic to solve any problem. I found all shell commands started with `pppoe`.

```
$ pppoe<TAB>
pppoe            pppoe-discovery  pppoe-server     pppoe-sniff      pppoe-status                    
pppoe-connect    pppoe-relay      pppoe-setup      pppoe-start      pppoe-stop
```

From the list, the command `pppoe-setup` looks promising. It asks for a username, password, DNS, Ethernet device (`enp0s31f6.7` we had just created it). There is a possibility to re-run the configuration command to update the settings.

My best guess was that the next command is `pppoe-start`.

Check the status `pppoe-status` and `ping 1.1.1.1` or `ping 8.8.8.8`. It should work. Review the changes in my network devices via `ifconfig` command line. I found `vlan7` and `ppp0`. The `ppp0` has an IP address.

In my environment, for some reason, the DNS setting was not applied. Probably because of the NetworkManager service.

### Network Manager

To make those devices available after reboot, moved the creation of VLAN to the Network Manager via **GUI** application `nm-connection-editor`. Created the connection called **VLAN** with *ID 7* and *Prio 0*. It setups a device with the name `vlan7`. It means we should update the existing **PPPoE** settings to use a new device.
It also supports **DSL/PPPoE** settings. I setup a connection for **PPPoE**. But now I don’t have any idea how to start this connection. So I stick with the `pppoe-start` solution. (If you know how to use the Network Manager connections pls let me know via comments.)
To make sure I have a clean environment, I rebooted the laptop. I checked that the device `vlan7` was created on boot.

```
$ ifconfig | grep vlan7
vlan7: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
```

Even when `pppoe-setup` asked for an option to setup a connection on *BOOT*, it does not work in my case. After the rebooting, a `ppp0` device was missing.

Run again `pppoe-setup` with the new device name `vlan7`. Everything else I kept the same. Next commands I run if I need the internet for the machine:

```
$ sudo pppoe-stop; sudo pppoe-start
```

### Share internet

After a successful internet link, it is time to allow other mobile devices to use the Internet. I checked a few examples, and thought would dive deep in the world of `iptables`. But found a simple solution in the response from [Stackexchange](https://unix.stackexchange.com/questions/234552/create-wireless-access-point-and-share-internet-connection-with-nmcli/384513#384513?newreg=0df41e8831974270bc6e81351844659b).

```
$ nmcli dev wifi hotspot ifname wlp4s0 ssid "Name" password "foorbar"
```

It appears that **GNOME** networking settings support **WiFi Hotspot**. Only has one problem — after a new login or reboot, the **Hotspot** converts to *general* **WiFi** *client*. Before this solution, I tested `[wifi-ap](https://docs.ubuntu.com/core/en/stacks/network/wifi-ap/docs/basic-ap-setup)` and `[hostapd](https://w1.fi/hostapd/)`.

### Summary

I have an excellent stable internet connection, thanks to **Telekom**. 
All my **IOS devices** connected to the **WiFi** and use the shared internet. The quality of my laptop’s **WiFi Hotspot** *is bellow average* **WiFi** routers.
With my limited knowledge, I configured **PPPoE** over **VLAN** and shared the internet via **WiFi**. There are a lot of steps that could be improved — for example, how to use **Network Manager** for **PPPoE**.

Some magic findings, Apple devices are not your best friends (AirPort use own **VLAN** id to split traffic), **macOS** supports the creation of **VLAN**, but it does not allow you to create a **PPPoE** connection over the **VLAN**.

In the end, thanks to my old laptop, which has **Linux** installed, I can create a simple **router** solution.
It would be much easy for *newbies* if the **GNOME’s Settings** can create **VLAN** and **PPPoE** connections.

> *P.S: My command to setup an Internet with WiFi Hotspot enabled*

```
$ sudo pppoe-stop; sudo pppoe-start; nmcli dev wifi hotspot ifname wlp3s0 ssid Kozak password “SecurityPassword”; echo “nameserver 8.8.8.8” | sudo tee /etc/resolv.conf
```

![](/assets/2020-03-25-convert-laptop-to-a-router-for-telekom-de-network-1_iifqfnqorqkZVMpCyq1BjA.png)

**Michael Nikitochkin** *is a Lead Software Engineer. Follow him on [LinkedIn](https://www.linkedin.com/in/michaelnikitochkin/) or [GitHub](https://github.com/miry).*


