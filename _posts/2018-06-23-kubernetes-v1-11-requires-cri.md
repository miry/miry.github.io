---
url: https://medium.com/notes-and-tips-in-full-stack-development/kubernetes-v1-11-requires-cri-8a181f621bb7
canonical_url: https://medium.com/notes-and-tips-in-full-stack-development/kubernetes-v1-11-requires-cri-8a181f621bb7
title: Kubernetes v1.11 requires CRI
subtitle: Handle errors on upgrade of kubernetes cluster from v1.11-beta.2 to v1.11-rc.1
slug: kubernetes-v1-11-requires-cri
description: On upgrade of kubernetes cluster from v1.11-beta.2 to v1.11-rc.1, there
  is a missed crictl file.
tags:
- kubernetes
- cri
- kubeadm
author: Michael Nikitochkin
username: miry
---

# Kubernetes v1.11 requires CRI

Today I upgraded my kubernetes cluster from `v1.11-beta.2` to `v1.11-rc.1`, discovered that a general warning message now becomes `ERROR`:

```
$ kubeadm init --config /etc/kubernetes/kubeadm.yml
I0623 08:00:06.826671    1703 feature_gate.go:230] feature gates: &{map[]}
[init] using Kubernetes version: v1.11.0-rc.1
[preflight] running pre-flight checks
I0623 08:00:06.907111    1703 kernel_validator.go:81] Validating kernel version
I0623 08:00:06.907182    1703 kernel_validator.go:96] Validating kernel config
[preflight] Some fatal errors occurred:
  [ERROR FileExisting-crictl]: crictl not found in system path
```

After a small investigation I found [github pull request](https://github.com/kubernetes/kubernetes/pull/64836/files) that adds a new package to arsenal, before I required to install via golang.

Welcome to `yum install -y cri-tools`. Updated terraform modules to preinstall CRI: [https://github.com/jetthoughts/infrastructure/commit/a783f4dd4382772cce0b86c20bdc2193d4fafa0f](https://github.com/jetthoughts/infrastructure/commit/a783f4dd4382772cce0b86c20bdc2193d4fafa0f).

![](/assets/2018-06-23-kubernetes-v1-11-requires-cri-1_191jZ4P6L-uGpX-7Fs1zGg.png)

**Michael Nikitochkin** *is a Lead Software Engineer. Follow him on [LinkedIn](https://www.linkedin.com/in/michaelnikitochkin/) or [GitHub](https://github.com/miry).*

> If you enjoyed this story, we recommend reading our [latest tech stories](https://jtway.co/latest) and [trending tech stories](https://jtway.co/trending).


