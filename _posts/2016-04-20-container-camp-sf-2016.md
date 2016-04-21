---
layout: post
title:  "Summary of Container Camp San Francisco 2016"
date:   2016-04-20 10:00:00
categories: blog
tags: [container, docker, rkt, orchestration, conferences, tech]
---

On my last day in San Francisco, I had the opportunity to attend [Container Camp](https://container.camp/). It was quite pleasing to be at a conference which was completely focussed on the container ecosystem. I felt the conference was divided into different themes - the morning starting with Container orchestration focussed talks & updates from different tools and OCI, afternoon focussed on container security and evening seemed to be some random stuff. Here are some of the highlights and some points I found interesting from the conference.

### Container orchestration

[Alex Williams](https://twitter.com/alexwilliams) from [@thenewstack](https://twitter.com/thenewstack) gave a very good insight into the survey results about Container Orchestration. One thing that surprised me was that Kubernetes seemed to be the most popular tool used for container orchestration, followed by things I expected to see such as bash scripts, configuration management tools such as ansible, chef, etc. Also configuration management seemed to be lower in priority when looking for container orchestration tools or Containers as a Service. I couldn't find the presentation deck, but the New Stack folks should be publishing the full report soon is what Alex said.

[Casey Bisson](https://twitter.com/misterbisson) from Joyent had a wonderful slide deck, but the story he told and the way he approached it was even better. I really enjoyed his talk. I felt the talk was themed around a simple idea - You should put your orchestrator in your local development environment. The way he put forth this idea is by relating to the sci-fi movies and what we could learn from them. Another thing pointed out and a quote I've always remember telling others is that "Containers are not a solution to all your problems". Containerizing all things won't simply resolve all your problems. Running docker in dev is easy, but when you talk about running docker in production you need to look at a bunch of things as you can see in the image below.

![things you need in place for docker in production](/assets/2016-04-20-container-camp-sf-2016/docker-in-prod.jpg)

### Tooling Updates about Docker, OCI and Kubernetes

[Mano Marks](https://twitter.com/ManoMarks) from Docker spoke about the new things in Docker engine 1.13 such as native support for OSX and Windows available in [beta](https://beta.docker.com/) which has been possible through native VM support through xhyve and Hyper-V respectively. It also support OCI specification as it now runs on containerd and runc. Also supports other architectures such as arm, i686, etc.

[Brandon Philips](https://twitter.com/BrandonPhilips) from CoreOS spoke about the [Open Container Initiative (OCI)](https://www.opencontainers.org/) and how it started off with [appc](https://github.com/appc). This was an interesting story about the growth and adoption of the OCI for the image specification. [rkt](https://coreos.com/rkt/) already follows the specification, but it is good to see that docker has also joined in with the latest release. The talk also briefly covered the Container Network Interface (CNI) and Container Network Model (CNM). I need to read more about this, but what I understood was that both CNI and CNM are distinct mechanisms to control the networking of the containers. CNI seems to be a flexible json specification based approach which defines a network namespace and then passes control over to a CNI plugin which can basically define the bridges, maintain the network state, etc. CNM on the other hand defines a network and endpoints which can be attached to containers. He also posted his slide deck which you can find [here](https://speakerdeck.com/philips/container-standards-and-interfaces-an-update-1).

![CNI vs. CNM](/assets/2016-04-20-container-camp-sf-2016/cni-vs-cnm.jpg)

[Tim Hockin](https://twitter.com/thockin) from Google spoke about the updates in Kubernetes 1.3. After giving a really nice intro to Kubernetes in 45 seconds, he managed to do 3 demos (which was impressive!) highlighting the Multi-zone support, new deployment methods allowing rolling deploys and DaemonSets. I also liked the batched processing with Jobs in Kubernetes.

![Intro to Kubernetes](/assets/2016-04-20-container-camp-sf-2016/intro-to-kubernetes.jpg)

### Container Security

[Thomas Cameron](https://twitter.com/thomasdcameron) from RedHat gave an introduction to container security covering the different aspects of security including cgroups, the different [namespaces](https://lwn.net/Articles/531114/), linux capabilities and SELinux.

Cythia Thomas from [@midokura](https://twitter.com/midokura) spoke about container network security and [kuryr](https://github.com/openstack/kuryr) a Docker network plugin for Openstack.

### Other random topics

I could just attend one talk by [Jessie Frazelle](https://twitter.com/frazelledazzell) from Mesosphere who spoke about Containers vs. Sandboxes. The talk was daringly presented without any slides and just as a demo, where she used [binctr](https://github.com/jfrazelle/binctr) to create some statically linked binaries embedded in containers which you can run completely unprivileged. She used libcontainer running with unprivileged access utilizing the user namespaces to run the binaries. The learnings from this talk were pretty great and I especially liked a quote she made - ["cgroups are what processes use and namespaces are what processes see"](https://twitter.com/akshay_karle/status/721103156155785216) clearly explaining the difference between cgroups and namespaces.

There were also lightning talks and some other talks in the end of the day but I couldn't attend as I had to leave early to catch my flight home. In general I liked the conference, but I would've loved to stay back and mingle with the folks at the conference or having some open spaces would've been perfect.
