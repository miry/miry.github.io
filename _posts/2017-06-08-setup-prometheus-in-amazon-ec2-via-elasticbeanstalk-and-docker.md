---
url: https://medium.com/notes-and-tips-in-full-stack-development/setup-prometheus-in-amazon-ec2-via-elasticbeanstalk-and-docker-13ec426c21c0
canonical_url: https://medium.com/notes-and-tips-in-full-stack-development/setup-prometheus-in-amazon-ec2-via-elasticbeanstalk-and-docker-13ec426c21c0
title: Setup Prometheus in Amazon EC2 via ElasticBeanstalk and Docker
subtitle: What is ElasticBeanstalk? It is one of Amazon Services that help developers
  to deploy applications. Now any cloud hosting wants to be…
slug: setup-prometheus-in-amazon-ec2-via-elasticbeanstalk-and-docker
description: ""
tags:
- docker
- amazon
- aws
- elastic-beanstalk
- tutorial
author: Michael Nikitochkin
username: miry
---

# Setup Prometheus in Amazon EC2 via ElasticBeanstalk and Docker

![](/assets/2017-06-08-setup-prometheus-in-amazon-ec2-via-elasticbeanstalk-and-docker-1_niG5MrTEyuenlndM3UBuTA.jpeg)

What is [ElasticBeanstalk](http://aws.amazon.com/de/elasticbeanstalk/)? It is one of Amazon Services that help developers to deploy applications. Now any cloud hosting wants to be modern and support docker, [ElasticBeanstalk](http://aws.amazon.com/de/elasticbeanstalk/) is one of them. Developers replace chef recipes with `Dockerfile` and don’t even think why.

[Best practice of Docker](https://docs.docker.com/articles/dockerfile_best-practices/) runs only one process. We often require more than one process like Application, DB, Background Workers etc. Before March 24, 2015 [ElasticBeanstalk](http://aws.amazon.com/de/elasticbeanstalk/) did not support multi-container on one instance. The Multi-container Docker platform differs from the other platform that [ElasticBeanstalk](http://aws.amazon.com/de/elasticbeanstalk/) provides. It replaced the custom bash scripts with [Elastic Container Service](http://aws.amazon.com/ecs/details/) commands. And this is Awesome!

[Prometheus](http://prometheus.io) is a monitoring system. Why have I chosen [Prometheus](http://prometheus.io) to describe how [ElasticBeanstalk](http://aws.amazon.com/de/elasticbeanstalk/) works? Because it contains multiple modules written with the help of different technologies.

Let’s play.

1. Run Prometheus on local machine[¹](#a240).

1. Setup ElasticBeanstalk Application and Environment[²](#0af8).

1. Customize Prometheus[³](#074a).

1. Add Prometheus Dashboard(Rails app and Mysql)[⁴](#5356).

1. Add Nginx with HTTP Basic Auth[⁵](#27dd).

1. Add Prometheus Pushgateway[⁶](#e0ac).

# Run Prometheus on a local machine

If you are not familiar with Docker you can find more information in [Docker User Guide](https://docs.docker.com/userguide/).

[![](/assets/2017-06-08-setup-prometheus-in-amazon-ec2-via-elasticbeanstalk-and-docker-1_NVLl4oVmMQtumKL-DVV1rA.png)](https://asciinema.org/a/5diw0wwk6vbbovnqrk5sh1soy)

```
$ boot2docker start 
$ eval "$(boot2docker shellinit)"
$ docker run -d -p 9090:9090 prom/prometheus
$ open http://"$(boot2docker ip)":9090
```

You should see the Prometheus status page. Try to play with Prometheus graph and queries. By default Prometheus gets own metrics. It was really easy. Don’t forget to stop process after:

```
$ docker ps
$ docker stop <container id: first column>
```

![](/assets/2017-06-08-setup-prometheus-in-amazon-ec2-via-elasticbeanstalk-and-docker-1_ZswaY-h2TkRZjW50ayPcpg.png)

# Create ElasticBeanstalk Application

```
$ mkdir prometheus
$ cd prometheus
$ brew install aws-elasticbeanstalk
$ eb init
$ eb create dev-env
$ eb open
```

It would create a Sample web application `prometheus` provided by [ElasticBeanstalk](http://aws.amazon.com/de/elasticbeanstalk/). More information about `eb` commands you can find [here](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb3-cmd-commands.html) and [installing guid](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb-cli3-install.html). There are 2 tiers: **Web** and **Worker**. I prefer use **Worker**, even for web applications that do not require to have the same IP address. Web tier creation and assigning `Elastic Load Balancer` or `Elastic IP` depends on your choice.

After that you can configure the application via web interface. Save configuration from environment settings menu. To access saved configurations create a command `config` .

```
$ eb config list
$ eb config get <configname>
$ vim .elasticbeanstalk/saved_configs/<configname>.cfg.yml
$ eb config put <configname>
```

It is a useful feature if you want to put configuration in a repo and create new environments based on those configurations. The local copy of config is located in `.elasticbeanstalk/saved_configs`

```
$ eb create other-env --cfg <configname>
```

Sample config:

```
EnvironmentConfigurationMetadata:
  DateModified: '1431853640000'
  DateCreated: '1431853640000'
AWSConfigurationTemplateVersion: 1.1.0.0
EnvironmentTier:
  Name: Worker
  Type: SQS/HTTP
SolutionStack: 64bit Amazon Linux 2015.03 v1.4.0 running Multi-container Docker 1.6.0 (Generic)
OptionSettings:
  aws:autoscaling:launchconfiguration:
    IamInstanceProfile: aws-elasticbeanstalk-ec2-role
    SecurityGroups: <somegroup>
    RootVolumeSize: null
    EC2KeyName: xxxxxxx
    InstanceType: m3.medium
  aws:elasticbeanstalk:environment:
    EnvironmentType: SingleInstance
```
> *[sample-config.yaml view raw](https://gist.githubusercontent.com/marchi-martius/e7e4a0c35bf638a39011f6a3fb724475/raw/d8205062a4d6a1b802fd6201927d24e5dc5c9985/sample-config.yaml)*

OK, we have an empty directory, EB application and Environment with sample application. The Multi-Container Docker application required the `Dockerrun.aws.json`. In this file we describe all our containers and how they are linked. More info about format of [Multi Container Docker Configuration](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/create_deploy_docker_v2config.html).

This is `Dockerrun.aws.json` to build and run `prometheus` like we do on local machine:

```
{
  "AWSEBDockerrunVersion": "2",
  "containerDefinitions": [
    {
      "name": "prometheus-app",
      "image": "prom/prometheus",
      "essential": true,
      "memory": 512,
      "portMappings": [
        {
          "hostPort": 9090,
          "containerPort": 9090
        }
      ]
    }
  ]
}
```
> *[Dockerrun.aws.json view raw](https://gist.githubusercontent.com/marchi-martius/a221ddd6c6fdad953b47e6dc9aad5920/raw/6d9239547927aed7326ef97beb666354a64dc505/Dockerrun.aws.json)*

The config is clear, except the `essential`. This option required to mark this container like important, and if during deploy any containers with enabled `essential` exits with non zero, the deploy would be aborted.

Let’s do local test for this configuration. Yep, it is the one of most cool features from `eb` running containers in local env using the same `docker` and `docker-compose`.

```
$ eb local run
Creating elasticbeanstalk_prometheusapp_1...
Attaching to elasticbeanstalk_prometheusapp_1
prometheusapp_1 | prometheus, version 0.13.3 (branch: master, revision: 7af85f9)
prometheusapp_1 |   build user:       root
prometheusapp_1 |   build date:       20150520-12:38:18
prometheusapp_1 |   go version:       1.4.2
prometheusapp_1 | I0521 21:14:09.034372       1 storage.go:207] Loading series map and head chunks...
prometheusapp_1 | I0521 21:14:09.034417       1 storage.go:212] 0 series loaded.
prometheusapp_1 | W0521 21:14:09.034434       1 main.go:136] No remote storage URLs provided; not sending any samples to long-term storage
prometheusapp_1 | I0521 21:14:09.034455       1 targetmanager.go:64] Pool for job prometheus does not exist; creating and starting...
prometheusapp_1 | I0521 21:14:09.036597       1 web.go:107] listening on :9090
```
> *[prometheus-eb-run view raw](https://gist.githubusercontent.com/marchi-martius/6be4fde486eb6d0dc0580bad69fac954/raw/50bf3c5dd39b5378f1e94c0d2e2923b929936856/prometheus-eb-run)*

It works like a charm. It creates a file `.elasticbeanstalk/docker-compose.yml` that used to run containers. Here is more help to understand it: [Docker Compose](https://docs.docker.com/compose/).

```
prometheusapp:
  image: prom/prometheus
  ports:
  - 9090:9090
```
> *[docker-compose.yaml view raw](https://gist.githubusercontent.com/marchi-martius/22cae3581bc4ee526fc96558ec6555ed/raw/4fe83a37a9a9a7e1e3dbea3e57c0116ace3ccf08/docker-compose.yaml)*

This file would help us in future to debug. Next step deploy:

```
$ eb deploy
```

Wait for a few mins and check that all finished without `ERROR`.

# Customize Prometheus

We setup the sample prometheus with default configuration and settings. Let’s add custom config file, that is stored out of the container with default settings.

```
$ mkdir prometheus
$ cat > prometheus/prometheus.yml <<EOY
# my global config
global:
  scrape_interval:     15s # By default, scrape targets every 15 seconds.
  evaluation_interval: 15s # By default, scrape targets every 15 seconds.
  # scrape_timeout is set to the global default (10s).

  # Attach these extra labels to all timeseries collected by this Prometheus instance.
  labels:
      monitor: 'codelab-monitor'

# Load and evaluate rules in this file every 'evaluation_interval' seconds.
rule_files:
  # - "first.rules"
  # - "second.rules"

# A scrape configuration containing exactly one endpoint to scrape: 
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'

    # Override the global default and scrape targets from this job every 5 seconds.
    scrape_interval: 5s
    scrape_timeout: 10s

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    target_groups:
      - targets: ['localhost:9090']
EOY

```
> *[prometheus-custom-init.sh view raw](https://gist.githubusercontent.com/marchi-martius/3c9faa44e3d6688627241fb98ea7ebfa/raw/ace50cdbfc52553ba995cf402f0e34728a72ee83/prometheus-custom-init.sh)*

Check the default `CMD` for prometheus in [https://registry.hub.docker.com/u/prom/prometheus/dockerfile](https://registry.hub.docker.com/u/prom/prometheus/dockerfile). In current version it looks like:

```
CMD        [ "-config.file=/etc/prometheus/prometheus.yml", \
             "-storage.local.path=/prometheus", \
             "-web.console.libraries=/etc/prometheus/console_libraries", \
             "-web.console.templates=/etc/prometheus/consoles" ]
```
> *[prometheus-dockerfile-cmd view raw](https://gist.githubusercontent.com/marchi-martius/e4558a429c60a2007d55e1faf3bb6ba8/raw/f003f3fb02a74ded8cc9a1d6a5e92fb2888d01c3/prometheus-dockerfile-cmd)*

We need to change only one parameter `-config.file` to point to our file and others remain unchanged. Let’s suppose that our config file will be located in `/opt/prometheus/prometheus.yml`. In a pure docker it would look like:

```
$ docker run -p 9090:9090 -t prom/prometheus -config.file=/opt/prometheus/prometheus.yml -storage.local.path=/prometheus -web.console.libraries=/etc/prometheus/console_libraries -web.console.templates=/etc/prometheus/consoles

prometheus, version 0.14.0 (branch: stable, revision: 67e7741)
  build user:       root
  build date:       20150603-06:20:33
  go version:       1.4.2
WARN[0000] No remote storage URLs provided; not sending any samples to long-term storage  file=main.go line=126
INFO[0000] Loading configuration file /opt/prometheus/prometheus.yml  file=main.go line=220
ERRO[0000] Couldn't load configuration (-config.file=/opt/prometheus/prometheus.yml): open /prometheus/prometheus.yml: no such file or directory  file=main.go line=224
ERRO[0000] Note: The configuration format has changed with version 0.14. Please see the documentation (http://prometheus.io/docs/operating/configuration/) and the provided configuration migration tool (https://github.com/prometheus/migrate).  file=main.go line=225

```
> *[prometheus-custom-docker-run view raw](https://gist.githubusercontent.com/marchi-martius/af684223cbd77d5817a9c1bf1daeb8e5/raw/4e30b4d4402a130669215e335699cd67a7f4545c/prometheus-custom-docker-run)*

This file is missing. To share our local folder with docker container we need to add specific option. More information about in [Managing data in containers](https://docs.docker.com/userguide/dockervolumes/). Simple version is: `docker run -v /path/local/folder:/path/container/folder`.

```
$ docker run -d -v $(pwd)/prometheus:/opt -p 9090:9090 -t prom/prometheus -config.file=/etc/prometheus/prometheus.yml -storage.local.path=/prometheus -web.console.libraries=/etc/prometheus/console_libraries -web.console.templates=/etc/prometheus/consoles
$ open http://"$(boot2docker ip)":9090
```
> *[prometheus-share-dir view raw](https://gist.githubusercontent.com/marchi-martius/696ddfc8e37733d072293ca6de10a8f2/raw/8d257eb815496ef576964749d7131454612ab0d4/prometheus-share-dir)*

It should work. Let’s stop it via `docker stop <container id: first column>`, change config and start again. We should see the new config in the status page. After we need to change option `-storage.local.path` to point to host volume to store all data on local machine.

Back to the documentation of `Dockerrun.aws.json` in [Multi Container Docker Configuration](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/create_deploy_docker_v2config.html). There are `volumes` and `mountPoints` options that we will use for mounting folders and `command` to change the default container `CMD` with our.

Add to the root of the config a key `volumes`:

```
{
  "AWSEBDockerrunVersion": "2",

  "volumes": [
    {
      "name": "prometheus-conf",
      "host": {
        "sourcePath": "/var/app/current/prometheus"
      }
    },
    {
      "name": "prometheus-data",
      "host": {
        "sourcePath": "/data/prometheus"
      }
    }
  ],

  "containerDefinitions": [
//...
```
> *[prometheus-volumes.json view raw](https://gist.githubusercontent.com/marchi-martius/ac93d509f09dfc5a018b78cae7ecdca1/raw/e5816fe108449f2480d032549b6cf77f158caeb0/prometheus-volumes.json)*

In this example, I added 2 volumes/folders that would be shared with containers. One folder for Prometheus configuration folder, that we keep in the git repository, and `data` folder would be stored in EBS volume of an EC2 instances. So we will not lose the data after reboot or redeploy. EB use `/var/app/current` to store current application working directory. Our latest code during deploy is located there.

In the container section we need to use that registered folder to mount in a correct container folder:

```
  "containerDefinitions": [
    {
      "name": "prometheus-app",
      //...

      "mountPoints": [
        {
          "sourceVolume": "prometheus-conf",
          "containerPath": "/opt/prometheus"
        },
        {
          "sourceVolume": "prometheus-data",
          "containerPath": "/data"
        }
      ]
    }
  ]
}
```
> *[prometheus-mount-points.json view raw](https://gist.githubusercontent.com/marchi-martius/e510030757b7f1f3e7414bb08fbd4b11/raw/2c4ab55a19a934bed1b193eda4045eb2f0dbbde2/prometheus-mount-points.json)*

Added attribute `command` to the container option to use new config file and data folder:

```
"command": [
  "-config.file=/opt/prometheus/prometheus.yml",
  "-storage.local.path=/data",
  "-web.console.libraries=/etc/prometheus/console_libraries",
  "-web.console.templates=/etc/prometheus/consoles"
]
```
> *[prometheus-config-command.json view raw](https://gist.githubusercontent.com/marchi-martius/810372e50b325b50ac9eee875d9903f3/raw/8e693f02d43ce0b1c210524b4d60e7fb904f3f3d/prometheus-config-command.json)*

The result would be:

```
{
  "AWSEBDockerrunVersion": "2",
  "volumes": [
    {
      "name": "prometheus-conf",
      "host": {
        "sourcePath": "/var/app/current/prometheus"
      }
    },
    {
      "name": "prometheus-data",
      "host": {
        "sourcePath": "/data/prometheus"
      }
    }
  ],
  "containerDefinitions": [
    {
      "name": "prometheus-app",
      "image": "prom/prometheus",
      "essential": true,
      "memory": 512,
      "portMappings": [
        {
          "hostPort": 9090,
          "containerPort": 9090
        }
      ],
      "mountPoints": [
        {
          "sourceVolume": "prometheus-conf",
          "containerPath": "/opt/prometheus"
        },
        {
          "sourceVolume": "prometheus-data",
          "containerPath": "/data"
        }
      ],
      "command": [
        "-config.file=/opt/prometheus/prometheus.yml",
        "-storage.local.path=/data",
        "-web.console.libraries=/etc/prometheus/console_libraries",
        "-web.console.templates=/etc/prometheus/consoles"
      ]
    }
  ]
}
```
> *[prometheus-config-result.json view raw](https://gist.githubusercontent.com/marchi-martius/c163336e9e4c94ebd09132c3991a90e2/raw/d1694e78748157b2c70e5150357ccd865c18b648/prometheus-config-result.json)*

Testing on a local machine first via `eb local run` and you would see something similar to:

![](/assets/2017-06-08-setup-prometheus-in-amazon-ec2-via-elasticbeanstalk-and-docker-1_Jd6m2a3d3zc0rkOxR1cazg.png)

Refresh the prometheus status page, and it would use our custom config. There is some trick, EB detect volumes that pointed to `/var/app/current` and uses the current folder to search for folders, but it does not mount data folder and that’s why your data will be missed after each restart. To verify this you can check `.elasticbeanstalk/docker-compose.yml`:

```
prometheusapp:
  command:
  - -config.file=/opt/prometheus/prometheus.yml
  - -storage.local.path=/data
  - -web.console.libraries=/etc/prometheus/console_libraries
  - -web.console.templates=/etc/prometheus/consoles
  image: prom/prometheus
  ports:
  - 9090:9090
  volumes:
  - /Users/user/projects/personal/prometheus/prometheus:/opt/prometheus
```
> *[docker-compose-new.yaml view raw](https://gist.githubusercontent.com/marchi-martius/2ce2d3fa5310f0f97514f11f19731094/raw/faac46aef44aed13d805788667aecb4c131a5608/docker-compose-new.yaml)*

So this file is good to debug your configuration file. And you can use it with a pure docker composer tool.

Let’s deploy `eb deploy` and debug via `eb logs`.

# Add Prometheus Dashboard (Rails app and MySQL)

We have a lot of data, now we need to add simple dashboards. [PromDash](https://registry.hub.docker.com/u/prom/promdash/) is nice tiny tool with nice features.

`PromDash` is a small Rails application and it uses MySQL database. This is a good example how to setup to docker containers and link its.

Every Rails application require to specify environment variable `RAILS_ENV` to `production`. For PromSash we also require to specify the MySQL db url. Add a new container specification to our `Dockerrun.aws.json` for MySQL:

```
{
  "volumes":[
   {
      "name": "mysql-data",
      "host": {
        "sourcePath": "/data/mysql"
      }
    },

    //...
  ],
  "containerDefinitions": [
    {
      "name": "db",
      "image": "mysql",
      "essential": true,
      "memory": 512,
      "environment": [
        {
          "name": "MYSQL_ROOT_PASSWORD",
          "value": "secretpassword"
        }
      ],
      "mountPoints": [
        {
          "sourceVolume": "mysql-data",
          "containerPath": "/var/lib/mysql"
        }
      ]
    },
    //...

  ]
}
```
> *[prometheus-dockerrun-mysql.json view raw](https://gist.githubusercontent.com/marchi-martius/00bf33d0f7d47cd534bc8987850c1c76/raw/dba77a4e9aa9f4afd20dd7c84957991f88bde799/prometheus-dockerrun-mysql.json)*

After verification that it works `eb local run`. Here I used a new option `environment`. This option is obvious and simple. Next let’s add PromDash container:

```
    {
      "name": "rails-app",
      "image": "prom/promdash",
      "essential": true,
      "links": [
        "db"
      ],
      "memory": 256,
      "portMappings": [
        {
          "hostPort": 3000,
          "containerPort": 3000
        }
      ],

      "environment": [
        {
          "name": "RAILS_ENV",
          "value": "production"
        },
        {
          "name": "DATABASE_URL",
          "value": "mysql2://root:secretpassword@[HOST]/promdash"
        }
      ]
    }
```
> *[prometheus-config-promdash-container.json view raw](https://gist.githubusercontent.com/marchi-martius/2df7add0b6ddac1ad67b4fe77ea58a49/raw/cc2cef2d9c0e0d273bf496ed2baa9fe9bef395c2/prometheus-config-promdash-container.json)*

Here I have added a key `links` with an array of container names that will be used in this container. Docker container generates a new ip address on each run. And to figure out what is ip address of linked container is possible only by hostname and environment variables based on a link name. More information in [Linking containers together](https://docs.docker.com/userguide/dockerlinks/). The example how to get an ip address check [PromDash Dockerfile](https://registry.hub.docker.com/u/prom/promdash/dockerfile/).

After run `eb local run` there is a lot info from Mysql container and now we can check the page:

```
$ open http://"$(boot2docker ip)":3000
```

It will return *We’re sorry, but something went wrong.* Because the database is not configured and there are no tables. We need to add a migration script to run required commands on deploy and run. For rails we need to run `bin/rake db:create db:migrate` for initial setup DB. Add a new container based on same PromDash image, but instead of run Web server, going to run this command.

```
   {
      "name": "db-migration",
      "image": "prom/promdash",
      "essential": false,
      "memory": 128,
      "links": [
        "db"
      ],
      "environment": [
        {
          "name": "RAILS_ENV",
          "value": "production"
        },
        {
          "name": "DATABASE_URL",
          "value": "mysql2://root:password@[HOST]/promdash"
        }
      ],
      "command": [
        "./bin/rake",
        "db:create",
        "db:migrate"
      ]
    },
```
> *[prometheus-config-db-migrate.json view raw](https://gist.githubusercontent.com/marchi-martius/722877c22bbcc43f3b68989c96b73355/raw/e2bff178394d656e04321a024c50d9950a9c42a0/prometheus-config-db-migrate.json)*

Added an option `essential: false` to mark this container as should not kill all other container after it finished working. The next problem we would come across, because the mysql initializes few seconds, and our rake task could not connect during setup, and we see `Mysql2::Error: Can't connect to MySQL server on '172.17.0.52' (111)`. It is ok. Let’s add our script to run rake task after some seconds.

`bin/delay.sh`:

```
#!/bin/sh
timeout=$1
shift
echo "Delayed command $@ by $timeout seconds"
sleep $timeout
exec $@
```
> *[delay.sh view raw](https://gist.githubusercontent.com/marchi-martius/680378d6a9e0d3efb1aeb79d42b9a4ba/raw/947a81cb68a9be74c483cb60aaf02d9240a5efcd/delay.sh)*

Next we should mount our bin folder and wrap rake command with `delay.sh`:

```
"volumes":{
  ...
    {
      "name": "helpers",
      "host":{
        "sourcePath": "/var/app/current/bin"
      }
    }
},
"containerDefinitions": [
  ...
    {
      "name": "db-migration",
      "image": "prom/promdash",
      "essential": false,
      "memory": 128,
      "links": [
        "db"
      ],
      "environment": [
        {
          "name": "RAILS_ENV",
          "value": "production"
        },
        {
          "name": "DATABASE_URL",
          "value": "mysql2://root:secretpassword@[HOST]/promdash"
        }
      ],

      "mountPoints": [
        {
          "sourceVolume": "helpers",
          "containerPath": "/opt/tools"
        }
      ],

      "command": [
        "sh",
        "/opt/tools/delay.sh",
        "10",
        "./bin/rake",
        "db:create",
        "db:migrate"
      ]
    }
]
```
> *[prometheus-config-delaysh.json view raw](https://gist.githubusercontent.com/marchi-martius/907666b60c866822c975bff0f87b54fe/raw/9df07c973fce097d9d7d45f374e0ea56cb37c179/prometheus-config-delaysh.json)*

Trying again `eb local run`, it looks like the migration has started, but quits afterwards. EB local does not support an option `essential: false` because it was related to [Docker Compose](https://docs.docker.com/compose/). Add new wrapper `bin/sleep.sh`:

```
#!/bin/sh
echo `$@`
if [ $? -eq 0 ]; then
    echo "Sleeping for 1 hour"
    sleep 3600
else
    echo "FAILED $@ exited with status $?"
fi
```
> *[sleep.sh view raw](https://gist.githubusercontent.com/marchi-martius/684174acd850c47d981137ea4d4170ec/raw/d43704c2cf8f98d73003b384ca92da09163f0552/sleep.sh)*

And update the `command`:

```
      "command": [
        "sh",
        "/opt/tools/delay.sh",
        "10",
        "/opt/tools/sleep.sh",
        "./bin/rake",
        "db:create",
        "db:migrate"
      ]
```
> *[prometheus-config-sleep.json view raw](https://gist.githubusercontent.com/marchi-martius/3a855d695cfe58ab0c8f3f89d3c18471/raw/8ab7478f3b3d4fe0bbdc82470aadd97592689bdc/prometheus-config-sleep.json)*

Start `eb local run` and it seems it works. Verify by: `open http://"$(boot2docker ip)":3000`. Starting deploying to verify our configuration on EB. (PS: Don’t forget to change the Security groups allow you access 3000 port and remove wrapper `delay.sh` and `sleep.sh`).

# Add Nginx with HTTP Basic Auth

To add nginx container you should go through similar steps as for rails-app. Nginx should open 2 ports: 80 for PromDash and 9090 for Prometheus app to access api from the promdash.

```
"volumes":[
  {
      "name": "nginx-proxy-conf",
      "host": {
        "sourcePath": "/var/app/current/proxy/conf.d"
      }
    },

...
],
"containerDefinitions":[
  ...
 {
      "name": "nginx-proxy",
      "image": "nginx",
      "essential": true,
      "memory": 128,
      "portMappings": [
        {
          "hostPort": 80,
          "containerPort": 80
        },
        {
          "hostPort": 9090,
          "containerPort": 9090
        }
      ],
      "links": [
        "rails-app",
        "prometheus-app"
      ],
      "mountPoints": [
        {
          "sourceVolume": "awseb-logs-nginx-proxy",
          "containerPath": "/var/log/nginx"
        },
        {
          "sourceVolume": "nginx-proxy-conf",
          "containerPath": "/etc/nginx/conf.d",
          "readOnly": true
        }
      ]
    }
...
```
> *[prometheus-config-nginx.json view raw](https://gist.githubusercontent.com/marchi-martius/1b7852f4115ed7499811519b22933af8/raw/b9b3621fc921676904148c3fc20eed6581bc040f/prometheus-config-nginx.json)*

Remove ports from the `prometheus-app` container and `rails-app`. Create site config `proxy/conf.d/default.conf`:

```
upstream rails_app {
    server rails-app:3000;
}

upstream prometheus_app {
    server prometheus-app:9090;
}

server {
	listen 80;
	server_name localhost;

	location / {
    auth_basic "password";
    auth_basic_user_file /etc/nginx/conf.d/htpasswd;
    proxy_set_header   X-Real-IP       $remote_addr;
    proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header   Host            $http_host;
    proxy_redirect     off;
    proxy_read_timeout 90;
    proxy_pass         http://rails_app;
  }
}

server {
	listen 9090;
	server_name localhost;

	location / {
    auth_basic "password";
    auth_basic_user_file /etc/nginx/conf.d/htpasswd;
    proxy_set_header   X-Real-IP       $remote_addr;
    proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header   Host            $http_host;
    proxy_redirect     off;
    proxy_read_timeout 90;
    proxy_pass         http://prometheus_app;
  }
}

```
> *[prometheus-nginx-proxy.conf view raw](https://gist.githubusercontent.com/marchi-martius/957a22a638e2499ec6a254472941e806/raw/48f666175df8060e9dd105be29f235facddf0273/prometheus-nginx-proxy.conf)*

As you see in the upstream configuration I use container names as host. Because docker during the links creates for each linked container a record in `/etc/hosts` with current ip address.

Also create a file `proxy/conf.d/htpasswd` to restrict access. [How To Set Up HTTP Authentication With Nginx](https://www.digitalocean.com/community/tutorials/how-to-set-up-http-authentication-with-nginx-on-ubuntu-12-10).

# Add Prometheus Pushgateway

Prometheus poll applications for metrics, but sometimes it is not possible to reach an application. There is [Push Gateway](https://github.com/prometheus/pushgateway). Container example:

```
"volumes":[
  ...
  {
      "name": "prometheus-gateway-data",
      "host": {
        "sourcePath": "/data/prometheus_gateway"
      }
    }

]

""
   {
      "name": "prometheus-gateway",
      "image": "prom/pushgateway",
      "essential": true,
      "memory": 1024,
      "mountPoints": [
        {
          "sourceVolume": "prometheus-gateway-data",
          "containerPath": "/data"
        }
      ],
      "command": [
        "-persistence.file",
        "/data/pushgateway.dat"
      ],
      "portMappings": [
        {
          "hostPort": 9091,
          "containerPort": 9091
        }
      ]
    },
```
> *[prometheus-config-pushgateway.json view raw](https://gist.githubusercontent.com/marchi-martius/03ffd222ada56b86e3e5a9d0441d4fb0/raw/13e1427ea0ed2ef970d588d04b831fc86ea6653a/prometheus-config-pushgateway.json)*

Restart an application on a local machine and it should open the port: `open http://"$(boot2docker ip)":9091` with current metrics from applications.

Prometheus application can communicate with gateway via `link`.

```
    {
      "name": "prometheus-app",
....
      "links": [
        "prometheus-gateway"
      ]
    },
....
```
> *[prometheus-config-pushgateway-link.json view raw](https://gist.githubusercontent.com/marchi-martius/3373a0a4a8848b4ec7deb0854c3196ee/raw/993cf09a3259e6ea62e78e0b954f879d9174638a/prometheus-config-pushgateway-link.json)*

After that we modify `prometheus/prometheus.yml` add to `scrape_configs` section:

```
  - job_name: 'prometheus-gateway'
    scrape_interval: '60s'

    target_groups:
      - targets: ['prometheus-gateway:9091']
```
> *[prometheus-scrape-configs.yaml view raw](https://gist.githubusercontent.com/marchi-martius/04fb690d2880cd39e58917ae0d46aeb8/raw/24b1d63c0228b32281d93625ac80659795b10ee0/prometheus-scrape-configs.yaml)*

Restart the local application and check the Prometheus status page. There you should see a new endpoint `[http://prometheus-gateway:9091/metrics`.](http://prometheus-gateway:9091/metrics.)

The final `Dockerrun.aws.json`:

```
{
  "AWSEBDockerrunVersion": "2",
  "volumes": [
    {
      "name": "nginx-proxy-conf",
      "host": {
        "sourcePath": "/var/app/current/proxy/conf.d"
      }
    },
    {
      "name": "mysql-data",
      "host": {
        "sourcePath": "/data/mysql"
      }
    },
    {
      "name": "prometheus-conf",
      "host": {
        "sourcePath": "/var/app/current/prometheus"
      }
    },
    {
      "name": "prometheus-data",
      "host": {
        "sourcePath": "/data/prometheus"
      }
    },

    {
      "name": "helpers",
      "host":{
        "sourcePath": "/var/app/current/bin"
      }
    },
    {
      "name": "prometheus-gateway-data",
      "host": {
        "sourcePath": "/data/prometheus_gateway"
      }
    }
  ],
  "containerDefinitions": [
    {
      "name": "db",
      "image": "mysql",
      "essential": true,
      "memory": 512,
      "environment": [
        {
          "name": "MYSQL_ROOT_PASSWORD",
          "value": "secretpassword"
        }
      ],
      "mountPoints": [
        {
          "sourceVolume": "mysql-data",
          "containerPath": "/var/lib/mysql"
        }
      ]
    },
    {
      "name": "prometheus-gateway",
      "image": "prom/pushgateway",
      "essential": true,
      "memory": 1024,
      "mountPoints": [
        {
          "sourceVolume": "prometheus-gateway-data",
          "containerPath": "/data"
        }
      ],
      "command": [
        "-persistence.file",
        "/data/pushgateway.dat"
      ],
      "portMappings": [
        {
          "hostPort": 9091,
          "containerPort": 9091
        }
      ]
    },


    {
      "name": "prometheus-app",
      "image": "prom/prometheus",
      "essential": true,
      "memory": 512,
      "mountPoints": [
        {
          "sourceVolume": "prometheus-conf",
          "containerPath": "/opt/prometheus"
        },
        {
          "sourceVolume": "prometheus-data",
          "containerPath": "/data"
        }
      ],
      "command": [
        "-config.file=/opt/prometheus/prometheus.yml",
        "-storage.local.path=/data",
        "-web.console.libraries=/etc/prometheus/console_libraries",
        "-web.console.templates=/etc/prometheus/consoles"
      ],
      "links": [
        "prometheus-gateway"
      ]
    },
    {
      "name": "rails-app",
      "image": "prom/promdash",
      "essential": true,
      "links": [
        "db"
      ],
      "memory": 256,
      "environment": [
        {
          "name": "RAILS_ENV",
          "value": "production"
        },
        {
          "name": "DATABASE_URL",
          "value": "mysql2://root:secretpassword@[HOST]/promdash"
        }
      ]
    },

    {
      "name": "db-migration",
      "image": "prom/promdash",
      "essential": false,
      "memory": 512,
      "links": [
        "db"
      ],
      "environment": [
        {
          "name": "RAILS_ENV",
          "value": "production"
        },
        {
          "name": "DATABASE_URL",
          "value": "mysql2://root:secretpassword@[HOST]/promdash"
        }
      ],

      "mountPoints": [
        {
          "sourceVolume": "helpers",
          "containerPath": "/opt/tools"
        }
      ],

      "command": [
        "sh",
        "/opt/tools/delay.sh",
        "20",
        "/opt/tools/sleep.sh",
        "./bin/rake",
        "db:create",
        "db:migrate"
      ]
    },
    {
      "name": "nginx-proxy",
      "image": "nginx",
      "essential": true,
      "memory": 128,
      "portMappings": [
        {
          "hostPort": 80,
          "containerPort": 80
        },
        {
          "hostPort": 9090,
          "containerPort": 9090
        }
      ],
      "links": [
        "rails-app",
        "prometheus-app"
      ],
      "mountPoints": [
        {
          "sourceVolume": "awseb-logs-nginx-proxy",
          "containerPath": "/var/log/nginx"
        },
        {
          "sourceVolume": "nginx-proxy-conf",
          "containerPath": "/etc/nginx/conf.d",
          "readOnly": true
        }
      ]
    }
  ]
}

```
> *[prometheus-config-final.json view raw](https://gist.githubusercontent.com/marchi-martius/8f8c744e5900bdca45cacb5a8e58cf7b/raw/4a1c7cc716b969e418f5da547603174c817c903b/prometheus-config-final.json)*

Sample project in [Github](https://github.com/miry/prometheus_on_eb)

* [Getting Started Using Elastic Beanstalk](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/GettingStarted.html)

![That’s all Folks](/assets/2017-06-08-setup-prometheus-in-amazon-ec2-via-elasticbeanstalk-and-docker-1_iifqfnqorqkZVMpCyq1BjA.png)

**Michael Nikitochkin** *is a Lead Software Engineer. Follow him on [LinkedIn](https://www.linkedin.com/in/michaelnikitochkin/) or [GitHub](https://github.com/miry).*

> If you enjoyed this story, we recommend reading our [latest tech stories](https://jtway.co/latest) and [trending tech stories](https://jtway.co/trending).


