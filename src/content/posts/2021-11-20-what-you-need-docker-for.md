---
template: blog-post
title: What you need docker for
slug: /WhatYouNeedDockerFor
date: 2021-11-27 12:08
description: Docker why you still need 
---
# What you need docker for

## A not so silent revolution

//TODO: differentiate between docker sw and docker company

Back in 2013, when Docker came out it changed everything. It made everything we knew about development crumble. In a world where Heroku seemed the next big tech giant, it made Heroku pail. It took concepts that have been available in the Unix worlds for several decades (namespaces and control groups) and used them to allow independent applications to run close to each other without dependencies on the underlying hardware and OS.

When it first came out it initially seemed to many that it was going to be the next fad, but history showed otherwise. But what does Docker actually do?

## Docker explained

Docker builds on a key feature of the Linux feature, the LXC, which provides namespace segregation and control groups allowing to create a shippable package (a container) that runs as an independent process in the target machine.
This is the killing feat that Docker brought to the world: it allows to define a package without having to care about how it will be run. It gives you guarantees in terms of process isolation, the ability for this package to see and communicate with other packages, the tools to build new packages, execute them, and manage their lifecycle. 

### What's the docker stack

People tend to mix up Docker with containers. Docker is one way of building containers (being fair to Docker, it was what made containers accessible to a greater audiencie), but it's nor the only way, nor the only piece of software involved in doing it.

The act of dealing with the **low level stuff** (creating and running containerized processes) docker is usually tributed with, is actually performed by a lesser known component known as [runc][1]. 

There is then an additional layer built on top of runc, [containerd][2], which allows to handle networking and storage, deals with the images lifecycle and, through runc, run the containers.

What does docker bring then? It allows to run containerc via a developer friendly command interface.

### That's it? Why does Docker matter at all?

Looking at this landscape, it might seem that Docker is not that relevant as it might have seemed. And it's true, Docker is not the cornerstone technology it used to be. But what has changed?

If we start looking deeper, we will see that containerd was donated to the open source community by Docker, meaning that Docker (the company) decided to [extract part the technology[3] it built and make it an open source software. This means that without Docker, there wouldn't be a containerd.

What about runc? Well, it turns out that Docker donated it's own runtime to the Open Container Initiative ([OCI][4]) which served as the [basis for runc][5].

## Why did Docker give up on it's key components?

Docker claimed it did it to ensure that both runc and containerd had the opportunity to be developed independently,, without any specific influence from any particular company. Personally, I think this has been a strategic decision from the Docker team to allow:
1. two technologies that Docker relies on to be maintained by the community (hence removing the need of Docker investing a substantial amount of development time on them)
2. spread the usage of runc and containerd (by having more projects depend on them), hence making the docker stack the de facto standard

I believe both panned out allright for Docker.

## Docker architecture

The architecture for Docker looks like this:

![Docker architecture](../images/container-ecosystem-docker.drawio.png "Docker architecture")

In a typical scenario, where the container is launched:

1. Docker creates the container from an image, and calls containerd
2. Containerd calls containerd-shim, asking it to run the container
3. Containerd calls runc, asking it to run the container
4. Runc runs the container

What is containerd-shim? containerd-shim is acting as the middle manager between for all communication between the container manager (containerd) and the container runtime (runc) providing key features to each:
- it allows the container runtime to survive a container manager restart
- it shields the container manafer from specifities of the container runtime (e.g. propagating the exit and error code of running containers and keeping a container's stdin to allow attaching to the container)

## Docker centrality

Over the years, some critics have been raised on the Docker container implementation.

This lead to alternative ways of defining, building and managing containers. The most successful alternative has probably been RKT ([now defunct](Ending and archiving the rkt project)).

Back when RKT was all the rage, a request to Kuebernetes to support RKT, lead Kubernetes to create an abstraction called CRI or Container Runtime Interface to allow multuple containers runtime in Kubernetes. This allowed Kubernetes to be able to support Docker, Containerd, CRI-O (the first replacement build for the CRI interface) and others. Ultimately, Kubernetes removed the support for the Docker runtime, but this action had no impact as Docker images are OCI compliant, and all OCI compliant images look the same to Kubernetes.

## OK, so why do I still care about Docker?

After having revolutionized the industry, Docker Inc. straggled to find a viable business model.

After having seen Kubernetes take over as the platform of choice and the failure of initiatives such as Docker Enterprise, it seemed that that was it for Docker. The last attempt Docker has made is to ask for a 5$/user/month fee for developers using it in a commercial environment.

What are people getting for 5$/month? Nowadays Docker is focusing on building dev tools. The UI provided by Docker makes extremely easy for developer to monitor the status of containers running locally and for their company to enforce CI and SecOps rules.

## Is it worth it?

[1]: https://github.com/opencontainers/runc "Runc source code"
[2]: https://containerd.io/ "Containerd website"
[3]: https://www.docker.com/blog/containerd-joins-cncf/ "containerd joins the Cloud Native Computing Foundation"
[4]: https://opencontainers.org/ "https://opencontainers.org/"
[5]: https://segmentfault.com/a/1190000040223946/en "Runc 1.0 release"
[6]: https://github.com/rkt/rkt/issues/4024 "Ending and archiving the rkt project"
[7]: https://kubernetes.io/blog/2020/12/02/dont-panic-kubernetes-and-docker/ "https://kubernetes.io/blog/2020/12/02/dont-panic-kubernetes-and-docker/"
[8]: https://javadoc.io/static/com.amazonaws/amazon-kinesis-client/1.12.0/com/amazonaws/services/kinesis/multilang/MultiLangDaemon.html "MultiLangDaemon source code"
[9]: https://docs.aws.amazon.com/streams/latest/dev/shared-throughput-kcl-consumers.html#:~:text=interface%20called%20the-,MultiLangDaemon,-.%20This%20daemon%20is "Multilang deamon explained on AWS docs"
[10]: https://github.com/vmware/vmware-go-kcl "Golang Kinesis consumer source code"
