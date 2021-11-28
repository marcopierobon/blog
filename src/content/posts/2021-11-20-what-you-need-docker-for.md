---
template: blog-post
title: What you need docker for
slug: /WhatYouNeedDockerFor
date: 2021-11-27 12:08
description: What you need Docker for
---
# What you need docker for

## What is docker

Everybody has heard about Docker. Not everyone might know that Docker was created by Docker Inc. Although it made sense when the Docker technology was created (in 2013), it still confounds people to this day.

## A not so silent revolution

Back in 2013, when Docker was launched, it sent shockwaves through the entire dev community. It made everything we knew about development and shipping crumble. In a world where Heroku seemed the next big tech giant, it made Heroku pail in comparison. It took concepts that had been available in the Unix worlds for several decades (namespaces and control groups) and used them to allow independent applications to run close to each other and without caring on dependencies on the underlying hardware and OS.

But what does Docker actually do?

## Docker explained

Docker builds on a key feature of the Linux feature, the LXC, which provides namespace segregation and control groups allowing to create a shippable package (a container) that runs as an independent process in the target machine.
This is the killing feat that Docker brought to the world: it allows to define a package without having to care about how it will be run. It gives you guarantees in terms of process isolation, the ability for this package to see and communicate with other packages, the tools to build new packages, execute them, and manage their lifecycle. 

### What's the docker stack

People tend to mix up Docker with containers. Docker is one way of building containers (being fair to Docker, it was what made containers accessible to a greater audience), but it's nor the only way, nor the only piece of software involved in doing it.

The act of dealing with the **low level components** (creating and running containerized processes) that Docker is usually tributed with, is actually performed by a lesser known component known as [runc][1]. 

There is then an additional layer built on top of runc, [containerd][2], which allows handling networking and storage, deals with the images lifecycle and, through runc, run the containers.

What does docker bring then? It allows running containerd commands via a developer friendly command interface.

### That's it? Does Docker still matter at all?

Looking at this landscape, it might seem that Docker nowadays is not that relevant as it was. And it's true, Docker is not the cornerstone technology it used to be. But what has changed? To answer this, we need to understand how Docker operates.

### Docker architecture

The high level Docker architecture is:

![Docker architecture](../images/container-ecosystem-docker.drawio.png "Docker architecture")

In a typical scenario, where the container is launched:

1. Docker creates the container from an image, and calls containerd
2. Containerd calls containerd-shim, asking it to run the container
3. Containerd calls runc, asking it to run the container
4. Runc runs the container

What is containerd-shim? containerd-shim is acting as the middle manager between for all communication between the container manager (containerd) and the container runtime (runc) providing key features to each:
- it allows the container runtime to survive a container manager restart
- it shields the container manager from specificities of the container runtime (e.g. propagating the exit and error code of running containers and keeping a container's stdin to allow attaching to the container)

How were these components created? And by whom?

If we start digging deeper, we will see that containerd was donated to the open source community by Docker. This means that Docker (the company) decided to [extract part the technology][3] it built to make it an independent open source software. This means that without Docker, there wouldn't be a containerd.

What about runc? Well, it turns out that Docker donated its own runtime to the Open Container Initiative ([OCI][4]) which served as the [basis for runc][5].

### Why did Docker give up on it's key components?

Docker claimed it did it to ensure that both runc and containerd had the opportunity to be developed independently, without any specific influence from any particular company. Personally, I think this has been a strategic decision from the Docker Inc. team to allow:
1. two technologies that Docker relies on to be maintained by the community (hence removing the need of Docker investing a substantial amount of development time on them)
2. spread the usage of runc and containerd (by having more projects depend on them), hence making the docker stack the de facto standard

I believe both (alleged) goals were achieved, but in doing so Docker Inc. lost the upper hand on the virtualization ecosystem.


## Docker centrality

Over the years, some critics have been raised on the Docker container implementation.

This lead to alternative ways of defining, building and managing containers. The most successful alternative has probably been RKT ([now defunct](Ending and archiving the rkt project)).

Back when RKT was all the rage, a request to Kubernetes to support RKT, lead Kubernetes to create an abstraction called CRI or Container Runtime Interface to allow multiple containers runtime in Kubernetes. This allowed Kubernetes to be able to support Docker, Containerd, CRI-O (the first replacement build for the CRI interface) and others. Ultimately, Kubernetes removed the support for the Docker runtime, but this action had no impact as Docker images too are OCI compliant, and all OCI compliant images look the same to Kubernetes.

### OK, so why do I still care about Docker?

Despite having revolutionized the industry, Docker Inc. straggled to find a viable business model for years.

After having seen Kubernetes take over as the running platform of choice and the failure of initiatives such as Docker Enterprise and Docker Hub, it seemed that that was it for Docker. The last attempt Docker has made is to charge for a $5/user/month fee for developers using Docker for Desktop it in a commercial environment (for companies of 250+ employees).

What are people getting for 5$/month? Nowadays, Docker is focusing on building dev tools to work with containers rather than on the actual container technology. The UI provided by Docker makes extremely easy for developer to monitor the status of containers running locally and for their company to enforce CI practices and SecOps rules. It also has a feature, called Development Environments Preview, that allows team members to switch to someone else dev environment without having to change local branch (and hence avoiding potential merge conflicts).

This change in the Docker for Desktop license, only affects the UI components (not the underlying docker daemon). For this reason, developers using Linux distributions are not affected (Docker for Desktop is available only for Windows and OSX).

### Is it worth paying for?

It depends. Not having it will force developers to interact to containers using the command line only, and monitoring of local containers will not be as easy as it is now. So, finding alternatives for the existing ways of work and train the team to learn them, might cost way more than $5/user/month.

### What are the alternatives?

As discussed, when it comes to running containers the heavy lifting is done by containerd and runc, so nothing else is strictly required.

Having said that, even for developers used to running docker from the command line, the experience would not be the same, as both containerd and runc operate at a lower level of abstraction compared to Docker.

### Enter Podman

[Podman][8] is a docker alternative that was built with the idea of having a CLI completely compatible with Docker. This was done with the idea of winning users over and came handy now when people are looking for a docker replacement without a steep learning curve.

Other alternatives are: 
- [Rancher Desktop][10]: Rancher Desktop is an open-source desktop application for Mac and Windows. It provides Kubernetes and container management. You can choose the version of Kubernetes you want to run. You can build, push, pull, and run container images. The container images you build can be run by Kubernetes immediately without the need for a registry. On macOS Rancher Desktop leverages a virtual machine to run containerd and Kubernetes. Windows Subsystem for Linux v2 is leveraged for Windows systems.
- [Lima][9] ("Linux-on-Mac" or "containerd for Mac") is a docker replacement built for OSX that is also used by [Rancher Desktop][10] when it's run on OSX
- [Buildah][11] is a tool that allows to build container images, with a greater level of control than the one allowed by Docker. Buildah does not deal with the container lifecycle, and for this is usually used in conjunction with Podman.

### Docker vs Podman

As mentioned, the CLI commands are the same. Here come a few examples:

#### Pull images

*Docker*
```sh
docker pull docker.io/library/httpd
```

*Podman*
```sh
podman pull docker.io/library/httpd
```

#### List images

*Docker*
```sh
docker images
```

*Podman*
```sh
podman images
```

#### Run a container

*Docker*
```sh
docker run -dt -p 8080:80/tcp docker.io/library/httpd
```

*Podman*
```sh
podman run -dt -p 8080:80/tcp docker.io/library/httpd
```

#### Run a container

*Docker*
```sh
docker run -dt -p 8080:80/tcp docker.io/library/httpd
```

*Podman*
```sh
podman run -dt -p 8080:80/tcp docker.io/library/httpd
```

#### List running containers

*Docker*
```sh
docker ps
```

*Podman*
```sh
podman ps
```

## Conclusions

Docker came and revolutionized the way developers work and their concept of virtualization. It didn't manage to capitalize on a significant competitive advantage, and now it's backing company, Docker Inc., is looking to find alternative ways to become profitable.

The $5/user/month can make sense to a lot of big companies that have their teams very used to the experience that Docker for Desktop provides when working on Windows or OSX systems. For everyone else, there are plenty of open source and free alternatives to pick from.

[1]: https://github.com/opencontainers/runc "Runc source code"
[2]: https://containerd.io/ "Containerd website"
[3]: https://www.docker.com/blog/containerd-joins-cncf/ "containerd joins the Cloud Native Computing Foundation"
[4]: https://opencontainers.org/ "https://opencontainers.org/"
[5]: https://segmentfault.com/a/1190000040223946/en "Runc 1.0 release"
[6]: https://github.com/rkt/rkt/issues/4024 "Ending and archiving the rkt project"
[7]: https://kubernetes.io/blog/2020/12/02/dont-panic-kubernetes-and-docker/ "https://kubernetes.io/blog/2020/12/02/dont-panic-kubernetes-and-docker/"
[8]: https://podman.io/ "Podman website"
[9]: https://github.com/lima-vm/lima "Lima source code"
[10]: https://rancherdesktop.io/ "Rancher Desktop website"
[11]: https://buildah.io/ "Buildah website"