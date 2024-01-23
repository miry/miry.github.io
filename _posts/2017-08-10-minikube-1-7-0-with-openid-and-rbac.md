---
url: https://medium.com/notes-and-tips-in-full-stack-development/minikube-1-7-0-with-openid-and-rbac-2d3327414fff
canonical_url: https://medium.com/notes-and-tips-in-full-stack-development/minikube-1-7-0-with-openid-and-rbac-2d3327414fff
title: Minikube 1.7.0 with OpenID and RBAC
subtitle: In the article Kubernetes 1.6.1 Authentication via Google I explained how
  to create Google Application and generate kubernetes token. I…
slug: minikube-1-7-0-with-openid-and-rbac
description: ""
tags:
- kubernetes
- minikube
- openid-connect
- devops
- google-cloud-platform
author: Michael Nikitochkin
username: miry
---

# Minikube 1.7.0 with OpenID and RBAC

In the article [Kubernetes 1.6.1 Authentication via Google](https://jtway.co/kubernetes-auth-google-with-rbac-60a74787e6a5) I explained how to create Google Application and generate kubernetes token. I found that people have problem and to help them to test OpenID token authorization I want to present [Minikube](https://github.com/kubernetes/minikube) solution. I assume this solution should remove problems with different host environments and networking issues.

I replaced [Step 2](https://jtway.co/kubernetes-auth-google-with-rbac-60a74787e6a5#5a7f) part in the article with `minikube` command, before you start please finish first [Step 1](https://jtway.co/kubernetes-auth-google-with-rbac-60a74787e6a5#89c2) and [Step 3](https://jtway.co/kubernetes-auth-google-with-rbac-60a74787e6a5#aac8).

Here is replacement of `Step 2` and short version of [Step 4](https://jtway.co/kubernetes-auth-google-with-rbac-60a74787e6a5#ccaf):

```
$ minikube start \
      --extra-config=apiserver.Authorization.Mode=RBAC \
      --extra-config=apiserver.Authentication.OIDC.IssuerURL=https://accounts.google.com \
      --extra-config=apiserver.Authentication.OIDC.UsernameClaim=email \
      --extra-config=apiserver.Authentication.OIDC.ClientID="123123123.apps.googleusercontent.com"
```

```
$ kubectl get no
NAME       STATUS    AGE       VERSION
minikube   Ready     8m        v1.7.0
```

```
$ kubectl create clusterrolebinding cluster-admin-minikube --clusterrole=cluster-admin --user="user@exmaple.com"
```

```
$ kubectl get no --user="user@exmaple.com"
```

I found this approach easy to implement and good for testing applications on local environment with more close to production configurations and permissions.

![](/assets/2017-08-10-minikube-1-7-0-with-openid-and-rbac-1_191jZ4P6L-uGpX-7Fs1zGg.png)

**Michael Nikitochkin** *is a Lead Software Engineer. Follow him on [LinkedIn](https://www.linkedin.com/in/michaelnikitochkin/) or [GitHub](https://github.com/miry).*

> If you enjoyed this story, we recommend reading our [latest tech stories](https://jtway.co/latest) and [trending tech stories](https://jtway.co/trending).


