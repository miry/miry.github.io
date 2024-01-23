---
url: https://medium.com/notes-and-tips-in-full-stack-development/ubuntu-server-16-06-on-raspberry-pi-3-via-terraform-93dccaef5ddb
canonical_url: https://medium.com/notes-and-tips-in-full-stack-development/ubuntu-server-16-06-on-raspberry-pi-3-via-terraform-93dccaef5ddb
title: Ubuntu Server 16.04 on Raspberry Pi 3 via Terraform
subtitle: In this tutorial you will find how to setup latest Ubuntu Classic server
  16.04 and automate process as much as possible. Solve two issues…
slug: ubuntu-server-16-06-on-raspberry-pi-3-via-terraform
description: ""
tags:
- raspberry-pi
- terraform
- ubuntu
- arm
- tutorial
author: Michael Nikitochkin
username: miry
---

![](/assets/2017-07-02-ubuntu-server-16-06-on-raspberry-pi-3-via-terraform-1_wpJwYpRHBA4L2Idsn_17Uw.jpeg)

# Ubuntu Server 16.04 on Raspberry Pi 3 via Terraform

In this tutorial you will find how to setup latest Ubuntu Classic server 16.04 and automate process as much as possible. Solve two issues related to upgrade packages and kernel.

***Hardware and software requirements:***

* Raspberry Pi 3

* A microSD card with adapter

* A monitor with an HDMI interface

* A USB keyboard

* Ethernet cable with Internet connection

* Terraform ([Getting Started](https://www.terraform.io/intro/getting-started/install.html))

* (Optional) Terraform example of resources [repo](https://github.com/jetthoughts/infrastructure/blob/master/clusters/maas/ubuntu.tf)

# Install Ubuntu

First step is to download the latest [Ubuntu Classic Server 16.04](http://www.finnie.org/software/raspberrypi/ubuntu-rpi3/ubuntu-16.04-preinstalled-server-armhf+raspi3.img.xz) image from the [download page](https://ubuntu-pi-flavour-maker.org/download/) of **Ubuntu Pi Flavour Maker**.

There are many solutions to burn an image on a microSD you can find on official [ubuntu page](https://developer.ubuntu.com/core/get-started/installation-medias), but I suggest to use [Etcher](https://etcher.io/) from [Resin.io](https://resin.io/) team. It is easy and faster for all OS.

# First boot and steps

1. Connect the device to local network and boot it.

1. If you have already DHCP server setup in the local network, than you would get the IP address through booting output. Otherwise you need to manually login to the device and setup network. Default login and password is **ubuntu**. It would ask you to change the password after first login.

1. Update authorize keys of the user to access the host without password. Some solutions: `ssh-copy-id ubuntu@<ip>`. Or import Github’s public keys:

```
# Updated 2017.07.05 Thanks to https://www.reddit.com/user/xeoomd for the url
# from comment https://www.reddit.com/r/devops/comments/6l6uhm/ubuntu_server_1606_on_raspberry_pi_3_via_terraform/djsf1ai/
# Replace GTIHUB_USER with your username in Github
curl -s https://github.com/GTIHUB_USER.keys >> ~/.ssh/authorized_keys

# First version
# curl -s https://api.github.com/users/GTIHUB_USER/keys | grep key | sed 's/.*"key":\s*"\([^\"]*\)"/\1/g' >> ~/.ssh/authorized_keys 
```
> *[ssh-github-authorized-keys.sh view raw](https://gist.githubusercontent.com/miry/13a5de77ae88a259fdc0005d30a4ab4c/raw/cd432c02bb093bcefca0827b6d8c941d47dbe83b/ssh-github-authorized-keys.sh)*

# Automation

I choose [Terraform](https://www.terraform.io/), because it is easy to read and fast learning. The main purpose of Terraform to work with Cloud/Hosting providers and setup resources via API calls. To provision hosts I use `null_resource`. It allows to connect to the host and run different scripts. More information you can find in official documentation [Provisioners: null_resource](https://www.terraform.io/docs/provisioners/null_resource.html) and [Provisioner Connections](https://www.terraform.io/docs/provisioners/connection.html).

### Upgrade packages

So you have kernel version `4.4.0–1009-raspi2`. There are few bugs that blocks simple upgrading:

* [Could not boot into RPi3 with kernel 4.4.0–1038-raspi2](https://bugs.launchpad.net/ubuntu/+source/linux-raspi2/+bug/1652270)

* [linux-firmware-raspi2 conflicts with linux-firmware over brcmfmac43430-sdio.bin](https://bugs.launchpad.net/ubuntu/+source/linux-firmware-raspi2/+bug/1691729)

The upgrade packages, create a resource:

```
variable "server_ip" {
  default = "10.0.0.2"
}

# Terraform documentation
#  * Provisioner null_resource: https://www.terraform.io/docs/provisioners/null_resource.html
#  * Provisioner Connections: https://www.terraform.io/docs/provisioners/null_resource.html

# Ubuntu issues:
# https://bugs.launchpad.net/ubuntu/+source/linux-raspi2/+bug/1652270
# https://bugs.launchpad.net/ubuntu/+source/linux-raspi2/+bug/1652270/comments/44
# https://bugs.launchpad.net/ubuntu/+source/linux-firmware-raspi2/+bug/1691729
resource "null_resource" "upgrade-packages" {
  connection {
    type = "ssh"
    user = "ubuntu"
    host = "${var.server_ip}"
  }

  provisioner "remote-exec" {
    inline = [
      "sudo apt-mark hold linux-raspi2 linux-image-raspi2 linux-headers-raspi2",
      "sudo dpkg-divert --divert /lib/firmware/brcm/brcmfmac43430-sdio-2.bin --package linux-firmware-raspi2 --rename --add /lib/firmware/brcm/brcmfmac43430-sdio.bin",
      "sudo apt-get update && sudo apt-get upgrade -y && sudo apt-get dist-upgrade -y",
      "sudo apt-mark unhold linux-raspi2 linux-image-raspi2 linux-headers-raspi2",
      "sudo apt-get update && sudo apt-get upgrade -y && sudo apt-get dist-upgrade -y",
      "sudo sed 's/device_tree_address.*/device_tree_address=0x02008000/g; s/^.*device_tree_end.*//g;' -i /boot/firmware/config.txt",
      "sudo reboot"
    ]
  }
}
```
> *[raspi3-ubuntu-upgrade.tf view raw](https://gist.githubusercontent.com/miry/940c4e3b809bd2a0ab2470a612b786a3/raw/ed3fd7e4cbb30b46628bf5d924392c0889361a14/raspi3-ubuntu-upgrade.tf)*

Where `var.server_ip` is your Raspberry Pi’s ip address. To apply changes run command:

```
$ terraform apply -target=null_resource.upgrade-packages
```

### Change Hostname

By default the server’s hostname is `ubuntu`. It could be changed via resource:

```
variable "server_ip" {
  default = "10.0.0.2"
}

variable "server_hostname" {
  default = "node01"
}

# Ubuntu reference for hostnamectl: http://manpages.ubuntu.com/manpages/trusty/man1/hostnamectl.1.html
resource "null_resource" "set-hostname" {
  connection {
    type = "ssh"
    user = "ubuntu"
    host = "${var.server_ip}"
  }

  provisioner "remote-exec" {
    inline = [
      "sudo hostnamectl set-hostname ${var.server_hostname}",
      "echo '127.0.0.1 ${var.server_hostname}' | sudo tee -a /etc/hosts",
      "sudo reboot"
    ]
  }
}

```
> *[set-hostname.tf view raw](https://gist.githubusercontent.com/miry/bf52ed5507b758a6d27762edea303992/raw/0b708553e2756abf98451b3267448e07b019bdf0/set-hostname.tf)*

Update variables `server_ip` and `server_hostname` to your needs and apply changes:

```
$ terraform apply -target=null_resource.set-hostname
```

### Wifi

After upgrade the packages the wifi device was disappeared. Ubuntu issue:

* [Problem with WiFi Ubuntu MATE 15.10 on Pi3](https://www.raspberrypi.org/forums/viewtopic.php?f=28&t=141834)

In same thread you can find the solution. After fix the problem you need to setup connection to your WiFi. Also you can find more information how to setup Raspberry Pi with WiFi in [How to setup your Raspberry Pi 2/3 with Ubuntu 16.04, without cables (headlessly)](https://medium.com/a-swift-misadventure/how-to-setup-your-raspberry-pi-2-3-with-ubuntu-16-04-without-cables-headlessly-9e3eaad32c01).

```
variable "server_ip" {
  default = "10.0.0.2"
}

variable "wlan_ssid" {
  default = "free wifi hotspot"
}

variable "wlan_psk" {
  default = "changeme"
}

# https://www.raspberrypi.org/forums/viewtopic.php?f=28&t=141834
# https://medium.com/a-swift-misadventure/how-to-setup-your-raspberry-pi-2-3-with-ubuntu-16-04-without-cables-headlessly-9e3eaad32c01
# https://wiki.debian.org/WiFi/HowToUse
resource "null_resource" "wifi" {
  depends_on = ["null_resource.upgrade-packages"]

  connection {
    type = "ssh"
    user = "ubuntu"
    host = "${var.server_ip}"
  }

  provisioner "remote-exec" {
    inline = [
      "sudo apt-get install -y wireless-tools wpasupplicant",
      "cd /lib/firmware/brcm/",
      "sudo wget https://github.com/RPi-Distro/firmware-nonfree/raw/master/brcm80211/brcm/brcmfmac43430-sdio.bin",
      "sudo wget https://github.com/RPi-Distro/firmware-nonfree/raw/master/brcm80211/brcm/brcmfmac43430-sdio.txt",

      "echo \"allow-hotplug wlan0\niface wlan0 inet dhcp\nwpa-conf /etc/wpa_supplicant/wpa_supplicant.conf\" | sudo tee /etc/network/interfaces.d/10-wlan.cfg",
      "echo \"network={\\nssid=\\\"${var.wlan_ssid}\\\"\\npsk=\\\"${var.wlan_psk}\\\"\\n}\" | sudo tee /etc/wpa_supplicant/wpa_supplicant.conf",
      "sudo reboot"
    ]
  }
}
```
> *[wifi.tf view raw](https://gist.githubusercontent.com/miry/93de0e5b0653aa53e3e0ec99cb112445/raw/ccfeec46fdef7274c51e2d256b78bfcceecfe9aa/wifi.tf)*

```
$ terraform apply -target=null_resource.wifi
```

# Summary

Now you can access to your Raspberry Pi via *WiFi* or *Ethernet*. You can modify terraform resource to install `avahi` to detect the IP addresses of the device. Another solution to scan local network for hosts with open 22 port via:

```
$ nmap -sV -p 22  10.0.0.0/24
```

In the output would get a detail information about your local network, and choose ip similar to:

```
Nmap scan report for 10.0.0.2
Host is up (0.012s latency).
```

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

![That’s all Folks](/assets/2017-07-02-ubuntu-server-16-06-on-raspberry-pi-3-via-terraform-1_iifqfnqorqkZVMpCyq1BjA.png)

**Michael Nikitochkin** *is a Lead Software Engineer. Follow him on [LinkedIn](https://www.linkedin.com/in/michaelnikitochkin/) or [GitHub](https://github.com/miry).*

> If you enjoyed this story, we recommend reading our [latest tech stories](https://jtway.co/latest) and [trending tech stories](https://jtway.co/trending).


