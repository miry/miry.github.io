---
url: https://medium.com/notes-and-tips-in-full-stack-development/rotate-kubernetes-certificates-with-kubeadm-v1-11-6a9bbb0a7da3
canonical_url: https://medium.com/notes-and-tips-in-full-stack-development/rotate-kubernetes-certificates-with-kubeadm-v1-11-6a9bbb0a7da3
title: Rotate Kubernetes certificates with Kubeadm v1.11
subtitle: The way to update expired certificates for Kubernetes masters
slug: rotate-kubernetes-certificates-with-kubeadm-v1-11
description: TL;DR Use kubeadm tool on your local machine to rotate certificates
tags:
- docker
- kubernetes
- kubeadm
- certificate
- devops
author: Michael Nikitochkin
username: miry
---

# Rotate Kubernetes certificates with Kubeadm v1.11

![Photo by Anna Jiménez Calaf on Unsplash](/assets/2018-11-19-rotate-kubernetes-certificates-with-kubeadm-v1-11-1_6x3XczXohZ_8s3IOR_MaWQ.jpeg)

After building my first successful immutable cluster, I faced the problem of expired masters’ certificates. I have already mentioned how to generate new certificates in my previous article [**Generate Kubernetes client certificates**](https://jtway.co/generate-kubernetes-client-certificates-64782f7a58d5), so here is a quick recap:

```
$ docker run --rm -v $(pwd)/pki:/etc/kubernetes/pki \
             miry/kubernetes:v1.11.4 \
             /bin/kubeadm alpha phase certs all
[certificates] Generated ca certificate and key.
[certificates] Generated apiserver certificate and key.
[certificates] apiserver serving cert is signed for DNS names 
[certificates] Generated sa key and public key.
[certificates] Generated front-proxy-ca certificate and key.
[certificates] Generated front-proxy-client certificate and key.
...
[certificates] valid certificates and keys now exist in "/etc/kubernetes/pki"
```

If you run this command for same files, it would not update any existing files.

```
$ docker run --rm -v $(pwd)/pki:/etc/kubernetes/pki \
             miry/kubernetes:v1.11.4 \
             /bin/kubeadm alpha phase certs all
[certificates] Using the existing ca certificate and key.
[certificates] Using the existing apiserver certificate and key.
...
[certificates] valid certificates and keys now exist in "/etc/kubernetes/pki"
```

And would return the exception if there are any expired certificates:

```
$ docker run --rm -v $(pwd)/pki:/etc/kubernetes/pki \
             miry/kubernetes:v1.11.4 \
             /bin/kubeadm alpha phase certs all
[certificates] Using the existing ca certificate and key.
failure loading apiserver certificate: the certificate has expired
```

Check expiration date of certificates:

```
$ openssl x509 -noout -text -in pki/apiserver.crt
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 123123123123123(0x123123123123)
    Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN=kubernetes
        Validity
            Not Before: Nov 16 16:58:58 2017 GMT
            Not After : Nov 16 16:58:58 2018 GMT
...
```

To make the `kube-apiserver` process requests from current `kubelet` we need to update *apiserver certificate and key* along with *front-proxy-ca certificate and key*, while *ca certificate and key* as well as *sa key* should be the same. `kubelet` client certificates could also be the same, as `kubelet(`starting from version 1.8) does a rotation of certificates in the background.

```
$ rm pki/{apiserver*,front-proxy-client*}
$ docker run --rm -v $(pwd)/pki:/etc/kubernetes/pki \
             miry/kubernetes:v1.11.4 \
             /bin/kubeadm alpha phase certs all
[certificates] Using the existing ca certificate and key.
[certificates] Generated apiserver certificate and key.
[certificates] Using the existing apiserver-kubelet-client certificate and key.
...
[certificates] Generated front-proxy-client certificate and key.
```

Once new certificates are generated, let’s check the validation date:

```
$ openssl x509 -noout -text -in pki/apiserver.crt
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 32423234234234 (0x34535345345345345)
    Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN=kubernetes
        Validity
            Not Before: Nov 16 16:58:58 2017 GMT
            Not After : Nov 26 17:52:04 2019 GMT
        Subject: CN=kube-apiserver
```

Next, you should upload the certificates to master machines and restart the `kube-apiserver` process. Also, regenerate `admin.conf` via:

```
$ docker run --rm -v $(pwd)/pki:/etc/kubernetes \ 
             miry/kubernetes:v1.11.4 \ 
             /bin/kubeadm alpha phase kubeconfig admin
```

![](/assets/2018-11-19-rotate-kubernetes-certificates-with-kubeadm-v1-11-1_iifqfnqorqkZVMpCyq1BjA.png)

**Michael Nikitochkin** *is a Lead Software Engineer. Follow him on [LinkedIn](https://www.linkedin.com/in/michaelnikitochkin/) or [GitHub](https://github.com/miry).*

> If you enjoyed this story, we recommend reading our [latest tech stories](https://jtway.co/latest) and [trending tech stories](https://jtway.co/trending).


