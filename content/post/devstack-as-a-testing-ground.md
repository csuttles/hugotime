+++
author = "Chris Suttles"
categories = ["OpenStack"]
date = 2017-03-20T01:11:01Z
description = ""
draft = false
cover = "/images/2017/09/devstack2.png"
slug = "devstack-as-a-testing-ground"
tags = ["OpenStack"]
title = "Devstack as a testing ground"

+++


In my last post, I ran into some challenges with Ocata. The placement API and cells v2 configuration was not yet documented (that documentation is actually still only in draft form), and this made configuring them more difficult than most OpenStack services.

This is where Devstack can be a very useful tool.

In the [post on ask.openstack](https://ask.openstack.org/en/question/102256/how-to-configure-placement-service-for-compute-node-on-ocata/) where I found some help before there was much available, there's this very helpful suggestion for configuring cells and placement on devstack: 

> For devstack please add enable_service placement-api in local.conf

This was really easy. It's a single line in local.conf, and after running stack.sh to deploy, I had a working Devstack with placement and cells support. This allowed me to dig through config files, and poke around the database to see what devstack configured.

Usually, documentation is complete and easy to follow, but in this case, devstack allowed me to answer some of my own questions and helped me to better understand the scarce information that was available on placement and cells at the time.

I have found Devstack very useful in other situations too. Having a disposable OpenStack instance can be a very valuable troubleshooting tool, and being able to reference the automated config, and logs is a great way to try things without breaking a larger install. 

While working on ipv6 support, I was new to ipv6 and also new to OpenStack. It was very helpful to be able to see neutron configs and network namespaces in a simple environment completely under my control. At work, the network is much more convoluted, and is not something I can modify without help. Having an isolated environment where I had carte blanche was invaluable in helping me understand how neutron works, and in understanding what I needed to configure for ipv6 support in my larger environment.

While I have mentioned Devstack in my previous post, ["Crawl, Walk, Run"](http://blog.highspeedlogic.org/getting-started-with-openstack/), I want to ephasize how valuable it is for ongoing troubleshooting. Even if you are not ready to contribute to OpenStack, Devstack can still be instrumental in your success. If you are ready to contribute, Devstack can help you run tempest against your local working copy, and really test your own changes before sending them to review, so you can have more confidence your changes will pass the gate/CI pipeline, and you can get your changes merged more easily. 

In short, Devstack is very useful; it's a valuable tool for development, and for operations, and should not be overlooked or dismissed once you start to get more familiar with OpenStack.

