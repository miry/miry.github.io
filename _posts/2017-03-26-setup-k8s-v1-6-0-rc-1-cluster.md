---
url: https://medium.com/notes-and-tips-in-full-stack-development/setup-k8s-v1-6-0-rc-1-cluster-b4cc7749973f
canonical_url: https://medium.com/notes-and-tips-in-full-stack-development/setup-k8s-v1-6-0-rc-1-cluster-b4cc7749973f
title: Setup K8S v1.6.0-rc.1 cluster
subtitle: The light/simple kubernetes cluster could be done with kubeadm tool. In
  current day 2016.03.26, kubernetes v1.6.0 is not released. To…
slug: setup-k8s-v1-6-0-rc-1-cluster
description: ""
tags:
- kubernetes
- docker
- tutorial
- amazon-web-services
author: Michael Nikitochkin
username: miry
---

# Setup K8S v1.6.0-rc.1 cluster

The light/simple kubernetes cluster could be done with [kubeadm](https://kubernetes.io/docs/getting-started-guides/kubeadm/) tool. In current day 2016.03.26, kubernetes v1.6.0 is not released. To deploy a release candidate for playing I met with few issues.

Where and What is my playground:

* Cloud: AWS (or any host providers)

* OS: CentOS Linux release 7.3.1611 (Core)

* Docker: v1.12.6

All my machines I setup using packer(to build images) and terraform. For the article I would omit those tools.

# Prepare the packages

For stable kubernetes builds I usually use original repo `yum.kubernetes.io` from [https://kubernetes.io/docs/getting-started-guides/kubeadm/](https://kubernetes.io/docs/getting-started-guides/kubeadm/). Here is the first problem: I could not find release candidate builds in the repo. I suppose it should be somewhere, but it was not my day to find it. Found the beautiful release builder [https://github.com/kubernetes/release](https://github.com/kubernetes/release). Small changes in the [kubelet.spec](https://github.com/kubernetes/release/blob/master/rpm/kubelet.spec) to provide exactly my required build version:

```
%global KUBE_VERSION 1.6.0-rc.1
%global KUBE_VERSION_MAJOR 1.6.0
```

then replaced all `Version: %{KUBE_VERSION}` with `Version: %{KUBE_VERSION_MAJOR}` . Because of [https://github.com/kubernetes/release/issues/290](https://github.com/kubernetes/release/issues/290)

```
$ cd ./rpm
$ ./docker-build.sh
```

And Wuala: we have nice rpms in the output folder:

```
$ ls output/x86_64
kubeadm-1.6.0-0.x86_64.rpm kubelet-1.6.0-0.x86_64.rpm repodata kubectl-1.6.0-0.x86_64.rpm kubernetes-cni-0.3.0.1-0.07a8a2.x86_64.rpm
```

Upload the packages to [https://packagecloud.io/miry/kubernetes](https://packagecloud.io/miry/kubernetes).

# Node setup

All nodes (master and slaves) in the cluster should have same versions of kubernetes files.

```
# https://packagecloud.io/miry/kubernetes/install
$ curl -s https://packagecloud.io/install/repositories/miry/kubernetes/script.rpm.sh | sudo bash
$ sudo setenforce 0 || true # Required for K8S
$ yum install -y docker kubelet kubeadm kubectl kubernetes-cni
```

There is changes in the `systemd` service file for `kubelet`.

```
$ cat <<EOF > /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
[Service]
Environment="KUBELET_KUBECONFIG_ARGS=--kubeconfig=/etc/kubernetes/kubelet.conf --require-kubeconfig=true"
Environment="KUBELET_SYSTEM_PODS_ARGS=--pod-manifest-path=/etc/kubernetes/manifests --allow-privileged=true"
Environment="KUBELET_NETWORK_ARGS=--network-plugin=cni --cni-conf-dir=/etc/cni/net.d --cni-bin-dir=/opt/cni/bin"
Environment="KUBELET_DNS_ARGS=--cluster-dns=10.96.0.10 --cluster-domain=cluster.local"
Environment="KUBELET_AUTHZ_ARGS=--authorization-mode=Webhook --client-ca-file=/etc/kubernetes/pki/ca.crt"
Environment="KUBELET_EXTRA_ARGS=--cgroup-driver=systemd"
ExecStart=
ExecStart=/usr/bin/kubelet \$KUBELET_KUBECONFIG_ARGS \$KUBELET_SYSTEM_PODS_ARGS \$KUBELET_NETWORK_ARGS \$KUBELET_DNS_ARGS \$KUBELET_AUTHZ_ARGS \$KUBELET_EXTRA_ARGS
EOF
```

The difference to original, is that I added `--cgroup-driver=systemd` to `kubelet`. Docker and Kubernetes should have same `cgroup`.

Enable and run services:

```
$ sudo systemctl enable docker && sudo systemctl start docker
$ sudo systemctl enable kubelet && sudo systemctl start kubelet
```

# Master

The kubeadm is still in development and as I expected there are changes in the flags and options.

```
$ kubeadm init --apiserver-advertise-address=$NODE_PRIVATE_IP --apiserver-cert-extra-sans="my.example.com" --kubernetes-version="v1.6.0-rc.1"
```

Besides changes in the flags, now `kubeadm` waits until node would be alive and checking for `DNS` pod. That is strange, because the pod could not work without any [Network addons](https://kubernetes.io/docs/admin/addons/). Open a new connection I started to install network addons. But here also have some changes. So

1. For k8s the network addon installation file is different, because of changes in the daemon set.

1. Because the `kubeadm` was not finished the config file is not located in the default path.

1. By default, there is an Authorization option enabled and the non secure API port 8080 is not available even for localhost.

I use next command to install an application(network addon): `kubectl — kubeconfig=/etc/kubernetes/admin.conf apply -f [https://raw.githubusercontent.com/projectcalico/canal/master/k8s-install/kubeadm/1.6/canal.yaml](https://raw.githubusercontent.com/projectcalico/canal/master/k8s-install/kubeadm/1.6/canal.yaml)` and `kubeadm` finished the process in the first session.

To access from local computer copy the file `/etc/kubernetes/admin.conf` to `~/.kube/config`. Verify that it works: `kubectl get nodes`. If it does not work, check that the IP address is correct and the port 6443 reachable.

# Slave

In the slave joins also changed a bit. Now you need always provide the discovery port. Kubernetes API server secure port now is 6443. The typical join would be:

```
$ kubeadm join --token="${k8s_token}" ${master_ip}:6443
```

# Dashboard

I thought most hard work finished, and continued playing with k8s. Install dashboard as usual without any problems:

```
$ kubectl create -f https://rawgit.com/kubernetes/dashboard/master/src/deploy/kubernetes-dashboard.yaml
```

Checked that kube config is valid and run small ruby and bash script to import client certificate to the Mac OS Keychain:

```
require 'optparse'
require 'yaml'
require 'base64'

options = {
  config_path: File.join(ENV['HOME'], '.kube', 'config'),
  write_dir: File.join(ENV['HOME'], '.kube')
}

OptionParser.new do |opts|
  opts.banner = "Usage: extract_crt.rb [options]"

  opts.on('-s', '--source FILE_PATH', 'Path to the kube config') { |v| options[:config_path] = v }
  opts.on('-d', '--destination DIR', 'Path to directory where save key and certs') { |v| options[:write_dir] = v }

end.parse!

kube_path = options[:write_dir]
file_config = File.read options[:config_path]
config = YAML.load file_config

ca = Base64.decode64 config["clusters"][0]["cluster"]["certificate-authority-data"]
File.open(File.join(kube_path, 'ca.crt'), File::CREAT|File::TRUNC|File::RDWR, 0644) do |f|
  f.write(ca)
end

client_crt = Base64.decode64 config["users"][0]["user"]["client-certificate-data"]
File.open(File.join(kube_path, 'kubecfg.crt'), File::CREAT|File::TRUNC|File::RDWR, 0644) do |f|
  f.write(client_crt)
end

client_key = Base64.decode64 config["users"][0]["user"]["client-key-data"]
File.open(File.join(kube_path, 'kubecfg.key'), File::CREAT|File::TRUNC|File::RDWR, 0644) do |f|
  f.write(client_key)
end

```
> *[01_extract_crt.rb view raw](https://gist.githubusercontent.com/miry/9fbb8947510294c25285bda2a6e11900/raw/c4fef7a1bedf55ec4a088c8a6d42fc8ab6013b80/01_extract_crt.rb)*

```
#!/bin/bash
# Would ask for password to encrypt the key
openssl pkcs12 -export -clcerts -inkey ~/.kube/kubecfg.key -in ~/.kube/kubecfg.crt -out ~/.kube/kubecfg.p12 -name "kubernetes-client"
open ~/.kube/kubecfg.p12
```
> *[02_convert_to_p12.sh view raw](https://gist.githubusercontent.com/miry/9fbb8947510294c25285bda2a6e11900/raw/81b387a2d8ebab275f20fc492ddf462d0a30bd64/02_convert_to_p12.sh)*

Open in the browser `https://<master-ip>:6443/ui` and got the error page: Permission denied to get list of pods. Investigated in the ServiceAccounts a bit and came to [https://blog.kcluster.io/setup-role-based-access-control-rbac-and-audit-logs-for-kubernetes-clusters-9e4db5b67020#.ghvjfwjhr](https://blog.kcluster.io/setup-role-based-access-control-rbac-and-audit-logs-for-kubernetes-clusters-9e4db5b67020#.ghvjfwjhr) Because of Authentication, there is only 1 admin user.

```
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1alpha1
metadata:
  name: kube-system-admin
rules:
  - apiGroups: ["*"]
    resources: ["*"]
    verbs: ["*"]
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1alpha1
metadata:
  name: kube-system-service-account-role-binding
subjects:
  - kind: ServiceAccount
    name: default
    namespace: kube-system
roleRef:
  kind: ClusterRole
  name: kube-system-admin
```
> *[roles.yml view raw](https://gist.githubusercontent.com/miry/420660824519636dda4784c5608bfff2/raw/38d9842533c3856a70b3cc6c91049a28133e0bb6/roles.yml)*

Granted permissions to the default service account of UI:

```
$ kubectl apply --validate=false -f https://gist.githubusercontent.com/miry/420660824519636dda4784c5608bfff2/raw/c18299a83b0dce78dbb8b942002494fb5bfefe50/roles.yml
```

After this command the UI should work properly.

![That’s all Folks](/assets/2017-03-26-setup-k8s-v1-6-0-rc-1-cluster-1_iifqfnqorqkZVMpCyq1BjA.png)

**Michael Nikitochkin** *is a Lead Software Engineer. Follow him on [LinkedIn](https://www.linkedin.com/in/michaelnikitochkin/) or [GitHub](https://github.com/miry).*

> If you enjoyed this story, we recommend reading our [latest tech stories](https://jtway.co/latest) and [trending tech stories](https://jtway.co/trending).


