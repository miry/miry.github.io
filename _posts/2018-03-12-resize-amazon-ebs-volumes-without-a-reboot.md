---
url: https://medium.com/notes-and-tips-in-full-stack-development/resize-amazon-ebs-volumes-without-a-reboot-ca118b010b44
canonical_url: https://medium.com/notes-and-tips-in-full-stack-development/resize-amazon-ebs-volumes-without-a-reboot-ca118b010b44
title: REsize Amazon EBS volumes without a reboot
subtitle: In rare cases, it happens that disk is full, and usually, it happens not
  in right time. No worries, AWS supports hot expanding disk for EBS…
slug: resize-amazon-ebs-volumes-without-a-reboot
description: ""
tags:
- aws
- terraform
- be
- fx
author: Michael Nikitochkin
username: miry
---

# REsize Amazon EBS volumes without a reboot

![Photo by chuttersnap on Unsplash](/assets/2018-03-12-resize-amazon-ebs-volumes-without-a-reboot-1_KaQSz3BTS74PQcbsOpFg3A.jpeg)

In rare cases, it happens that disk is full, and usually, it happens not in right time. No worries, AWS supports hot expanding disk for EBS volumes attached to modern instances. Manual process split to 2 phase:

* change the volume size via Web console or command line;

* ssh to the instance and update a partition information.

I want to automate a bit of those and register my changes. Terraform is one of the tools that help me with this task.

What do we have: a volume formatted with XFS, connected to Linux and mounted somewhere. Let’s begin the work!

### Step 1: Create a terraform resource

Checking the document of the resource related to [EBS volume](https://www.terraform.io/docs/providers/aws/r/ebs_volume.html) and declare what do we know about the volume:

```
// volumes.tf
resource "aws_ebs_volume" "mysql" {
  availability_zone = "us-east-1a"
  size = 1000
  type = "gp2"
  tags {
    Name      = "mysql"
    Role      = "db"
    Terraform = "true"
    FS        = "xfs"
  }
}
```
> *[volumes.tf view raw](https://gist.githubusercontent.com/miry/acc89e8257184e040c88c1d5aed65e90/raw/8368f0914f7627983aba0352a29bf67bf18d88d9/volumes.tf)*

Import existing AWS resource to our state:

```
$ terraform import aws_ebs_volume.mysql vol-0123456789abcdef0
Import successful!
```

Check missed values from resources like tags and fix conflicts:

```
$ terraform plan -target=aws_ebs_volume.mysql
  ~ aws_ebs_volume.mysql
      tags.%:                    "1" => "2"
      tags.Name:                 "Created for Mysql" => "mysql"
      tags.Terraform:            "" => "true"
```

```
$ terraform apply -target=aws_ebs_volume.mysql
```

Update `size` field to `2000` and apply changes:

```
// volumes.tf
resource "aws_ebs_volume" "mysql" {
  availability_zone = "us-east-1a"
  size = 2000
  type = "gp2"
  tags {
    Name      = "mysql"
    Role      = "db"
    Terraform = "true"
    FS        = "xfs"
  }
}
```
> *[volumes.tf view raw](https://gist.githubusercontent.com/miry/aa6b80fd71223667e6698e58a11ff7be/raw/b2b0735b1e96da8faebb4ff1db3ada09f51cf4e0/volumes.tf)*

```
$ terraform apply -target=aws_ebs_volume.mysql
  ~ aws_ebs_volume.mysql
      size: "1000" => "2000"
```

### Step 2: Detect instance IP and volume disk

Search for instance with this volume. Create a [data resource](https://www.terraform.io/docs/providers/aws/d/instance.html) and set output of the instance id:

```
// volumes.tf
// ...
data "aws_instance" "mysql" {
  filter {
    name   = "block-device-mapping.volume-id"
    values = ["${aws_ebs_volume.mysql.id}"]
  }
}

output "instance_id" {
  value = "${data.aws_instance.mysql.id}"
}
```
> *[volumes.tf view raw](https://gist.githubusercontent.com/miry/e2a9d5c3410270b3001b94ba94cc7039/raw/4a8924df25718782d31387370680ef9817ceb114/volumes.tf)*

Update the state and check the results:

```
$ terraform refresh 
aws_ebs_volume.mysql: Refreshing state... (ID: vol-0123456789abcdef0)
data.aws_instance.mysql: Refreshing state...
```

```
Outputs:
instance_id = i-0123456789abcdef0
```

It is time to get the device name or mount point of our volume inside the instance. Here is one of the example how to get this information:

```
// volumes.tf
// ...
locals {
  mount_point = "${data.aws_instance.mysql.ebs_block_device.0.device_name}"
}

```
> *[volumes.tf view raw](https://gist.githubusercontent.com/miry/c5914d0b44eca51f2d227a970aa8d523/raw/c09db94a7b1e8c5dedbaa5ecc1099a3457fd7093/volumes.tf)*

If you manage volumes with OpsWorks, then it could be done by tags:

```
// volumes.tf
// ...
locals {
  mount_point = "${aws_ebs_volume.mysql.tags["opsworks:mount_point"]}"
}
```
> *[volumes.tf view raw](https://gist.githubusercontent.com/miry/311dd314cd445255650a81c7fef9d2da/raw/1917f6c7eafee25ba737810c761d7811d55de0a3/volumes.tf)*

### Step 3: Execute a script

The last step is to update the partition to use whole new disk size. The resource has 2 blocks - connection manifest and script:

```
// volumes.tf
// ...
resource "null_resource" "expand_disk" {
  connection {
    type        = "ssh"
    user        = "ubuntu"
    private_key = "${file("~/.ssh/id_rsa")}"
    host        = "${data.aws_instance.mysql.public_ip}"
  }
  
  provisioner "remote-exec" {
    inline = [
      "sudo lsblk",
      "sudo xfs_growfs ${local.mount_point}",
    ]
  }
}
```
> *[volumes.tf view raw](https://gist.githubusercontent.com/miry/eb5224a8c55958735a403780495ce53e/raw/0952ba95a154c2c594da186d976630f14f0c4414/volumes.tf)*

And last our command is to execute it:

```
$ terraform apply -target=null_resource.expand_disk
```

### Resume

In this article I presented a small case, in the real world these resources would be bigger and have some dependencies on other resources. I will add tricky small exercise on your shoulders how to run the script on each change of volume’s size. Here is the last version of the document:

```
resource "aws_ebs_volume" "mysql" {
  availability_zone = "us-east-1a"
  size = 2000
  type = "gp2"
  tags {
    Name      = "mysql"
    Role      = "db"
    Terraform = "true"
    FS        = "xfs"
  }
}

data "aws_instance" "mysql" {
  filter {
    name   = "block-device-mapping.volume-id"
    values = ["${aws_ebs_volume.mysql.id}"]
  }
}

output "instance_id" {
  value = "${data.aws_instance.mysql.id}"
}

locals {
  mount_point = "${data.aws_instance.mysql.ebs_block_device.0.device_name}"
}

resource "null_resource" "expand_disk" {
  connection {
    type        = "ssh"
    user        = "ubuntu"
    private_key = "${file("~/.ssh/id_rsa")}"
    host        = "${data.aws_instance.mysql.public_ip}"
  }
  
  provisioner "remote-exec" {
    inline = [
      "sudo lsblk",
      "sudo xfs_growfs ${local.mount_point}",
    ]
  }
}
```
> *[volumes.tf view raw](https://gist.githubusercontent.com/miry/51083c39541a2b034ad863616c79bbf6/raw/785e19b844776043f2218b15924020bbe6b4c732/volumes.tf)*

P.S.: Of course, you can accomplish the same by:

```
$ aws ec2 modify-volume --region us-east-1 --volume-id vol-11111111111111111 --size 2000 --volume-type gp2 --iops 100
$ ssh -t 58.18.28.18 "sudo xfs_growfs /ebs" # For XFS volume
$ ssh -t 58.18.28.18 "sudo resize2fs /dev/xdvn" # For EXT4 volume
```

![](/assets/2018-03-12-resize-amazon-ebs-volumes-without-a-reboot-1_191jZ4P6L-uGpX-7Fs1zGg.png)


