---
url: https://medium.com/notes-and-tips-in-full-stack-development/generate-kubernetes-client-certificates-64782f7a58d5
canonical_url: https://medium.com/notes-and-tips-in-full-stack-development/generate-kubernetes-client-certificates-64782f7a58d5
title: Generate Kubernetes client certificates
subtitle: TL;DR Use kubeadm tool on your local machine to generate certificates
slug: generate-kubernetes-client-certificates
description: ""
tags:
- kubernetes
- kubeadm
- ssl-certificate
- macos
author: Michael Nikitochkin
username: miry
---

# Generate Kubernetes client certificates

> TL;DR Use kubeadm tool on your local machine to generate certificates

![Ladies Stretch Circle. Photo by: Sarah Pflug](/assets/2018-06-08-generate-kubernetes-client-certificates-1_yBh37WtfHS_xdNNZyQKvbg.jpeg)

Security is the most important part of any Kubernetes cluster. In our days all my Kubernetes cluster uses OpenID authorization. It makes easy to set up many standalone clusters with `kubeadm` and shares with a team only `ca.crt`. One of the limits OpenID authorization — it is access to services via a browser without `kubectl proxy` or `kubectl port-forward`. There are a plenty of tools to make working web services with OpenID authorization.

The most common service that people will use in a cluster is the Kubernetes Dashboard UI. I am going to describe how to generate a new client certificate to access it, without libraries and hacks like “[kubernetes hard way](https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/master/docs/04-certificate-authority.md#the-kubelet-client-certificates)”.

You require certificates from your master on a local machine. Because I am fun of `kubeadm` and that it helps to build a cluster in the easiest way, I want to introduce one of the best features — “[alpha phase](https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-alpha/)”. Usually, I use `kubeadm` directly on a server. This feature can build a High Available cluster. I prepare all certificates and keys and distribute in all master instances before I have a cluster.

### Kubeadm for MacOS users

For some reason, Google developers don’t build `kubeadm` for `Darwin`. It is only one thing, that makes me crazy. Thanks to Docker/Containers we have always solution to change ARCH, and I released repo `miry/kubernetes` with basic binaries. [Dockerfile](https://github.com/jetthoughts/docker-images/tree/master/kubernetes) and [docker hub tags](https://hub.docker.com/r/miry/kubernetes/tags/). Basic command it would looks like:

```
$ docker run -v $(pwd)/pki:/etc/kubernetes/pki -it miry/kubernetes:v1.11.0-alpha.2 /bin/kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"11+", GitVersion:"v1.11.0-alpha.2", GitCommit:"ed9b25c90241b2b8a1fa10b96381c57f99ca952a", GitTreeState:"clean", BuildDate:"2018-05-02T12:02:56Z", GoVersion:"go1.10.1", Compiler:"gc", Platform:"linux/amd64"}
```

### Generate Admin user client certificate

I like people who created `kubeadm` . It makes so easy

```
$ kubeadm alpha phase kubeconfig admin --cert-dir ./pki --kubeconfig-dir ./
[kubeconfig] Wrote KubeConfig file to disk: "./admin.conf"
```

It is a valid `kubeconfig` that works for your cluster. Sometime ago I wrote script how to extract certificates from `kubeconfig`:

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

```
$ ruby 01_extract_crt.rb -s ./admin.conf -d ./
$ ls -1
01_extract_crt.rb
admin.conf
ca.crt
kubecfg.crt
kubecfg.key
pki
```

For MacOs users need to convert `kubecfg.crt` and `kubecfg.key` to PKCS 12 format via:

```
$ openssl pkcs12 -export -clcerts -inkey kubecfg.key -in kubecfg.crt -out kubecfg.p12 -name "kubernetes-admin-client"
# it would ask for encryption password, required when you need to import it to Keychain
$ open kubecfg.p12
```

For Firefox users, you need to import certificates to your browser in Settings page. For other people, open in a browser next page: `https://<kubernetes master ip address>:6443/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/` It would ask for client certificate pick the one that has current date.

### Generate User client certificate

Except generation of main certificates and `admin` user, `kubeadm alpha phase` also has functions to generate user client certificates.

```
$ kubeadm alpha phase kubeconfig user --client-name="foo" --cert-dir ./pki > user_foo.conf
$ ruby 01_extract_crt.rb -s ./user_foo.conf -d ./
$ openssl pkcs12 -export -clcerts -inkey kubecfg.key -in kubecfg.crt -out kubecfg.p12 -name "kubernetes-user-foo-client"
$ open kubecfg.p12
```

# Summary

`kubeadm` is a good knife in your arsenal. If you choose to build kubernetes with another tools, it still could help you to generate certificates and keys. Who is interested in Terraform, I shared it part of kubernetes master module: [certificates](https://github.com/jetthoughts/infrastructure/tree/master/modules/k8s_master/certificates)

![](/assets/2018-06-08-generate-kubernetes-client-certificates-1_191jZ4P6L-uGpX-7Fs1zGg.png)

**Michael Nikitochkin** *is a Lead Software Engineer. Follow him on [LinkedIn](https://www.linkedin.com/in/michaelnikitochkin/) or [GitHub](https://github.com/miry).*

> If you enjoyed this story, we recommend reading our [latest tech stories](https://jtway.co/latest) and [trending tech stories](https://jtway.co/trending).


