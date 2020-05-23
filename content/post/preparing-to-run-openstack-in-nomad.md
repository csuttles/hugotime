+++
author = "Chris Suttles"
categories = ["OpenStack", "containers"]
date = 2017-07-31T13:10:57Z
description = ""
draft = false
cover = "/images/2017/09/nomad-logo.jpg"
slug = "preparing-to-run-openstack-in-nomad"
tags = ["OpenStack", "containers"]
title = "Preparing to Run OpenStack in Nomad"

+++


###### Docs

The start of this journey begins with [docs](https://www.nomadproject.io/intro/), and [understanding the model for running containers in Nomad](https://www.nomadproject.io/docs/job-specification/index.html).  I ran through the examples and found it pretty easy once I got going.

###### Why

You might expect the "Why" to be the first step, and really, it is. We chose Nomad because IPv6 is a serious requirement for our installation, I work with a brilliant guy who happens to be a Nomad contributor. Also, Kubernetes does not support IPv6, and probably will not support it for longer than we can wait. Why OpenStack in containers? This question was answered [about a thousand times during the OpenStack Summit in Boston](https://www.openstack.org/summit/boston-2017/summit-schedule/global-search?t=containers).


###### Plan

We plan to use [Nomad](https://www.nomadproject.io/) for scheduling containers, and [Kolla](https://github.com/openstack/kolla) for building them. The initial rollout will be only stateless services, and we'll still use bare metal for the database and message queues. Once we get the container builds and stateless services handled, we'll tackle stateful services too.

###### Versions

Right now, we are running Ocata, and that is version 4.x.y in Kolla (ugh, yet another independent version number system). In the next few months we plan to move to Pike, which is 5.x.y in Kolla.

###### First steps

Right now, I am working on image builds. Once image builds are mostly sorted, I'll start working on developing Nomad job specs for each container, ideally via some automation hooked into the build process so this can all become a CI/CD pipeline. There's exciting times ahead!

