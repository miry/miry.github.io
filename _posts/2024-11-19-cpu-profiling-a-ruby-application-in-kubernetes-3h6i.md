---
url: https://dev.to/miry/cpu-profiling-a-ruby-application-in-kubernetes-3h6i
canonical_url: https://dev.to/miry/cpu-profiling-a-ruby-application-in-kubernetes-3h6i
title: CPU Profiling a Ruby Application in Kubernetes
slug: cpu-profiling-a-ruby-application-in-kubernetes-3h6i
description: In this article, I'll guide you through the steps to obtain CPU profiling
  traces from a running container in Kubernetes.
tags:
- ruby
- profiling
- programming
- kubernetes
author: Michael Nikitochkin
username: miry
---

![Cover image](/assets/2024-11-19-cpu-profiling-a-ruby-application-in-kubernetes-3h6i-cover_image-https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Flu9okrqilodwp0pqp7cf.jpeg)

# CPU Profiling a Ruby Application in Kubernetes


> **TL;DR**  In this article, I'll guide you through the steps to obtain CPU profiling traces from a running container in Kubernetes.

## Preparation

I won’t delve into the purpose of CPU profiling here but will focus on the instructions. For profiling, I’ll use [rbspy]. I initially tried [Pyroscope], but encountered issues with reporting data in a local Ruby example. Additionally, its Ruby gem support seemed outdated, so I switched back to [rbspy].

Profiling containers isn't straightforward - it requires some preparatory steps, including updating _Pod_ security settings to enable `SYS_PTRACE`.

### Updating Pod Security Settings

Here’s an example of the changes needed in the deployment configuration:

```patch
      containers:
        - name: web
          image: docker.io/rails/rails:master
          command:
            - bundle
            - exec
            - rails
            - server
+          securityContext:
+            capabilities:
+              add:
+                - SYS_PTRACE
```

Once deployed, you can begin debugging.  By default, containers run as non-root users and often lack the required profiling tools, which limits the usefulness of `kubectl exec`[^1]. Instead, you can use `kubectl debug`[^2] to hijack a node or pod with a custom container running in privileged mode [^3].

### Using `kubectl debug`

The `kubectl debug` command can add a container to the node or attach it to a target pod:

```shell
$ kubectl debug -it web-84cd66cb44-n82jt --image=alpine --target=web

$ kubectl debug node/backend4x-gw73h -it --image=alpine
```

There is a container image exists for [rbspy] and could be used like:

```shell
$ kubectl debug -it web-84cd66cb44-n82jt -c debugger --image=rbspy/rbspy:0.27.0-musl --target=web --profile=sysadmin
```

However, this approach didn’t work well in my Kubernetes cluster, and I didn’t have time to investigate further. Consider this a homework assignment for you. 

### An Alternative Approach

Here’s what I did instead:

```shell
$ kubectl debug -it web-84cd66cb44-n82jt -c debugger --image=alpine --target=web --profile=sysadmin
# wget -qO- https://github.com/rbspy/rbspy/releases/download/v0.27.0/rbspy-x86_64-unknown-linux-musl.tar.gz | tar xvz
# mv rbspy-x86_64-unknown-linux-musl/rbspy /usr/bin/rbspy
```

Even after adding `SYS_PTRACE`, I couldn’t get [rbspy] to work without adding `--profile=sysadmin`. Additionally, using `-c debugger` assigns a static container name, which simplifies automation.

## Recording the Profile

Once [rbspy] is installed, you’re ready to start profiling. Most applications run with `PID 1`, but if other scripts are running in the container, you may need to identify the correct _PID_ using `ps ax`. After identifying the _PID_, start collecting data (replace 1 with _PID_):

```shell
# rbspy record --pid 1 --raw-file /raw.gz --format flamegraph -f /flamegraph.svg
```

## Exporting the Data

Once profiling is complete, you need to copy the reports from the container. Use `kubectl cp`[^4] to achieve this:

```shell
$ kubectl cp -c debugger web-84cd66cb44-n82jt:/flamegraph.svg flamegraph.svg
$ kubectl cp -c debugger web-84cd66cb44-n82jt:/raw.gz raw.gz
```

## Generating Reports

While you already have `flamegraph.svg`, you can generate other formats, such as `speedscope`, from the raw data:

```shell
$ rbspy report -f speedscope -i raw.gz -o speedscope.out
```

You can then upload the file to [speedscope.app](https://www.speedscope.app/) to view the flamegraph over time.

![](/assets/2024-11-19-cpu-profiling-a-ruby-application-in-kubernetes-3h6i-8roj3eqb7c7blwqc2w30.jpg)

## Summary

[rbspy] simplifies profiling for running applications. However, in cloud environments with distributed loads, a more automated solution like continuous profiling (e.g., [Pyroscope]) is preferable. Currently, an external memory profiler isn’t available, but the Ruby community has introduced a new tool, [vernier], which could be useful for continuous profiling.

## References

[rbspy]: https://rbspy.github.io/  
[vernier]: https://github.com/jhawthorn/vernier  
[Pyroscope]: https://pyroscope.io/

[^1]: [`kubectl exec`](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_exec/)
[^2]: [`kubectl debug`](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_debug/)
[^3]: [Kubernetes: Debug Running Pods](https://kubernetes.io/docs/tasks/debug/debug-application/debug-running-pod/)
[^4]: [`kubectl cp`](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_cp/)



