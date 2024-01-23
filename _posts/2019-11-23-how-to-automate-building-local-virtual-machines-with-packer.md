---
url: https://medium.com/notes-and-tips-in-full-stack-development/how-to-automate-building-local-virtual-machines-with-packer-a238ba6b49c7
canonical_url: https://codeberg.org/miry/samples/src/branch/master/experiments/3-packer-images/README.md
title: How to automate building local virtual machines with Packer
subtitle: We will learn how to create a disk image base on a remote iso disk image.
  Auto install packages and configure a virtual machine with a…
slug: how-to-automate-building-local-virtual-machines-with-packer
description: Tutorial how to use packer with Qemu
tags:
- packers
- qemu
- automation
- linux
- continuous-integration
author: Michael Nikitochkin
username: miry
---

![Photo by Stefano Intintoli on Unsplash](/assets/2019-11-23-how-to-automate-building-local-virtual-machines-with-packer-1_pMnhyb-qARBFLnRkPObL8Q.jpeg)

# How to automate building local virtual machines with Packer

We will learn how to create a disk image base on a remote `iso` disk image. Auto install packages and configure a virtual machine with a small configuration file. Setup another disk image, this time, the source would have a different format.

Before we continue, all the sources are available in [Github](https://github.com/miry/samples/tree/master/experiments/3-packer-images/).

There are multiple ways to build a virtual machine for experiments. One of the ways is to use [**Packer**](https://www.packer.io/). [**Packer**](https://www.packer.io/) has a big [list](https://www.packer.io/docs/builders/index.html) of integrations: cloud providers, virtual machines, and containers.

### Preparation

* Install [**Packer**](https://www.packer.io/) command line in your environment. There are multiple [options;](https://www.packer.io/intro/getting-started/install.html)

* In this tutorial, I am going to use [**Qemu**](https://www.qemu.org/), but you can port the examples to any other [Packer builders](https://www.packer.io/docs/builders/index.html).

### **Step 1 — Building a base image**

In this step, I show you how to install **Centos** with preinstalled packages.

**Build configuration file**

Before start, visit the documentation page of [**Packer Qemu builder**](https://www.packer.io/docs/builders/qemu.html). There is a basic example with all the required options. I created a modified version and saved to [centos.json](https://github.com/miry/samples/tree/master/experiments/3-packer-images/)

```
{
  "variables": {
    "centos_password": "centos",
    "version": "1908"
  },

  "builders": [
    {
      "vm_name": "centos-packer.qcow2",
      "iso_urls": [
        "iso/CentOS-7-x86_64-Minimal-{{ user `version` }}.iso",
        "http://mirror.de.leaseweb.net/centos/7/isos/x86_64/CentOS-7-x86_64-Minimal-{{ user `version` }}.iso"
      ],
      "iso_checksum_url": "http://mirror.de.leaseweb.net/centos/7/isos/x86_64/sha256sum.txt",
      "iso_checksum_type": "sha256",
      "iso_target_path": "iso",
      "output_directory": "output-centos",
      "ssh_username": "centos",
      "ssh_password": "{{ user `centos_password` }}",
      "ssh_wait_timeout": "20m",
      "http_directory": "http",
      "boot_command": [
        "<tab> text ks=http://{{ .HTTPIP }}:{{ .HTTPPort }}/ks.cfg<enter><wait>"
      ],
      "boot_wait": "2s",
      "shutdown_command": "echo '{{ user `centos_password` }}' | sudo -S /sbin/halt -h -p",
      "type": "qemu",
      "headless": true,
      "memory": 4096,
      "cpus": 4
    }
  ]
}
```

Most important changes in settings:

* Use [iso_urls](https://www.packer.io/docs/builders/qemu.html#iso_urls), instead of [iso_url](https://www.packer.io/docs/builders/qemu.html#iso_url).
It tries to download one by one from the provided list. It allows reusing files that already downloaded.

* Set [headless](https://www.packer.io/docs/builders/qemu.html#headless) to `true` in order to hide **Qemu** window. For debugging purposes, you can change it to `false`.

* Important option [http_directory](https://www.packer.io/docs/builders/qemu.html#http_directory).

[**Packer**](https://www.packer.io/) helps us to run a `HTTP` webserver for the automation **Linux** setup process.
With this option, I specified where is the root web dir located.
In the case of **Centos**, we rely on [**Kickstart2**](https://docs.centos.org/en-US/centos/install-guide/Kickstart2/) to automate the installation process.
It works in tandem with `boot_command`, where we specify `IP` and
port of our `HTTP` webserver and the path to [**Kickstart2**](https://docs.centos.org/en-US/centos/install-guide/Kickstart2/) config file `ks.cfg`.

To make it works, create a new file `[ks.cfg](https://github.com/miry/samples/blob/master/experiments/3-packer-images/http/ks.cfg)` under catalog `http`:

```
install
cdrom
lang en_US.UTF-8
keyboard us
unsupported_hardware
network --bootproto=dhcp
firewall --disabled
authconfig --enableshadow --passalgo=sha512
selinux --permissive
timezone UTC
bootloader --location=mbr
text
skipx
zerombr
clearpart --all --initlabel
autopart
auth  --useshadow  --enablemd5
firstboot --disabled
reboot
rootpw MAGICROOTPASSWORD
user --name=centos --groups=centos --password=centos
```

```
%packages --ignoremissing --excludedocs
@Base
@Core
@Development Tools
-@Graphical Internet
openssl-devel
readline-devel
zlib-devel
kernel-devel
wget
curl
%end
```

```
%post
yum install -y epel-release
yum update -y
yum install -y httping iftop iperf3 ivpsadm net-tools nmap-ncat ntp sysstat tcpdump telnet tmux tree vim wireshark yum-plugin-fastestmirror yum-utils zsh
```

```
sudo systemctl enable ntpd
```

```
# sudo
echo "centos ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
```

```
# ssh key
mkdir /home/centos/.ssh
cat <<EOF >> /home/centos/.ssh/authorized_keys
<Your SSH public key>
EOF
```

```
chmod 0700 -R /home/centos/.ssh
chown centos:centos -R /home/centos/.ssh
sed -i "s/^.*requiretty/#Defaults requiretty/" /etc/sudoers
%end
```

Change passwords and authorize your **SSH** public key. It is ready to start building a new image.

### Run building centos image task

You should have two configurations files — one for builder specification, second is for auto-provisioning.

```
$ packer build centos.json
==> qemu: Retrieving ISO
==> qemu: Trying iso/CentOS-7-x86_64-Minimal-1908.iso
...
==> Builds finished. The artifacts of successful builds are:
--> qemu: VM files in directory: output-centos
```

The image’s path is `output-centos/centos-packer.qcow2`.

### Testing the base image

Run `**Qemu**` with attached our disk image as root volume.

```
$ qemu-system-x86_64 -name centos-packer \
                    -netdev user,id=user.0,hostfwd=tcp::4141-:22 \
                    -device virtio-net,netdev=user.0 \
                    -drive file=output-centos/centos-packer.qcow2,if=virtio,cache=writeback,discard=ignore,format=qcow2 \
                    -machine type=pc,accel=kvm \
                    -smp cpus=4,sockets=4 \
                    -m 4096M \
                    -display sdl
```

You can check logs installation:

```
$ sudo less /var/log/anaconda/ks-script-*.log
```

### Step 2 — Building a docker image

In this step, you are going to learn how to create a new disk image base on the existing one.
Like the previous step, create a packer config `[docker.json](https://github.com/miry/samples/tree/packer-build-automate/experiments/3-packer-images/)`:

```
{
  "builders": [
    {
      "vm_name": "docker-packer.qcow2",
```

```
      "disk_image": true,
      "iso_url": "output-qemu/centos-packer-1908.qcow2",
      "iso_checksum_type": "none",
      "output_directory": "output-docker-image",
      "ssh_username": "centos",
      "ssh_password": "centos",
      "ssh_pty": true,
      "ssh_wait_timeout": "2m",
      "shutdown_command": "echo 'centos' | sudo -S /sbin/halt -h -p",
      "type": "qemu",
      "headless": true,
      "memory": "8192",
      "cpus": 4
    }
  ],
  "post-processors": null,
  "provisioners": [
    {
      "type": "shell",
      "scripts": [
        "{{template_dir}}/docker.sh"
      ]
    }
  ]
}
```

There 2 main options are different from the first config: [disk_image](https://www.packer.io/docs/builders/qemu.html#disk_image) and [provisioners](https://www.packer.io/docs/provisioners/index.html).
In the `provisioners` section, you should specify the provision script. Create a sample [docker.sh](https://github.com/miry/samples/tree/packer-build-automate/experiments/3-packer-images/) file, mentioned in the config:

```
#!/usr/bin/env bash
set -euo pipefail
```

```
sudo yum install -y docker
sudo systemctl enable docker
```

### Run building docker image task

After all config and provision script is ready, run:

```
$ packer build docker.json
...
* A checksum type of 'none' was specified. Since ISO files are so big,
a checksum is highly recommended.
==> qemu: Retrieving ISO
==> qemu: Trying output-centos/centos-packer.qcow2
...
==> qemu: Connected to SSH!
==> qemu: Provisioning with shell script: docker.sh
...
    qemu: Complete!
...
==> Builds finished. The artifacts of successful builds are:
--> qemu: VM files in directory: output-docker
```

The result saved to `output-docker/docker-packer.qcow2`.

```
$ qemu-system-x86_64 -name docker-packer \
              -netdev user,id=user.0,hostfwd=tcp::4141-:22 \
              -device virtio-net,netdev=user.0 \
              -drive file=output-docker/docker-packer.qcow2,if=virtio,cache=writeback,discard=ignore,format=qcow2 \
              -machine type=pc,accel=kvm \
              -smp cpus=4,sockets=4 \
              -m 4096M \
              -display sdl
```

Log in to the instance and check access to docker service

```
$ ssh centos@localhost -p 4141 -i <path/to/ssh.key>
...
[centos@localhost ~]$ sudo docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

### Conclusion

We learn how to use samples from the Packer documentation page. We build a local machine with auto-provisioned packages, users, and configs. It is learned how to create another disk image base on an existing one.

**What can you do next?** You can create different images for Kubernetes clusters: `etcd`, `master`, and `node`. Build a disk image for your small experiments, then rebuild it from scratch. Test your provisioning scripts for cloud instances locally. Try to build images for other virtual machines like [**VMware**](https://www.packer.io/docs/builders/vmware.html) or [**VirtualBox**](https://www.packer.io/docs/builders/virtualbox.html).

For more on **Packer**, see more [guides on Packer.io](https://www.packer.io/guides/index.html).

![That’s all folks!](/assets/2019-11-23-how-to-automate-building-local-virtual-machines-with-packer-1_iifqfnqorqkZVMpCyq1BjA.png)

**Michael Nikitochkin** *is a Lead Software Engineer. Follow him on [LinkedIn](https://www.linkedin.com/in/michaelnikitochkin/) or [GitHub](https://github.com/miry).*

> *If you enjoyed this story, we recommend reading our [latest tech stories](https://jtway.co/latest) and [trending tech stories](https://jtway.co/trending).*

### References

https://www.packer.io/docs/builders/qemu.html

[https://docs.centos.org/en-US/centos/install-guide/Kickstart2/](https://docs.centos.org/en-US/centos/install-guide/Kickstart2/)

https://github.com/miry/samples/blob/master/experiments/3-packer-images/


