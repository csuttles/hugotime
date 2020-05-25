+++
author = "Chris Suttles"
categories = ["OpenStack"]
date = 2017-09-02T23:27:20Z
description = ""
draft = false
cover = "/images/2017/09/openstack-pike.jpg"
slug = "openstack-pike-released"
tags = ["OpenStack"]
title = "OpenStack Pike Released"
aliases = ["/openstack-pike-released/"]

+++


There's a lot of posts on this already: 

* [https://www.openstack.org/news/view/340/openstack-pike-delivers-composable-infrastructure-services-and-improved-lifecycle-management](https://www.openstack.org/news/view/340/openstack-pike-delivers-composable-infrastructure-services-and-improved-lifecycle-management)
* [https://www.mirantis.com/blog/53-things-to-look-for-in-openstack-pike/](https://www.mirantis.com/blog/53-things-to-look-for-in-openstack-pike/)
* [https://www.theregister.co.uk/2017/09/01/openstack_pike_release_emphasises_microservices_and_scale/](https://www.theregister.co.uk/2017/09/01/openstack_pike_release_emphasises_microservices_and_scale/)

Here's what I'm excited about in this release.

## Python3

There are a lot of good things in the Pike release, but I am very excited to see the move to Python3.5. I stopped writing Python2.x a while ago, after reading about async in [Dave Beazley's](http://www.dabeaz.com/) excellent Python column in [:login;](https://www.usenix.org/publications/login). I'm fortunate to work in a place that really embraces Python's new features, and fully supports Python3, so this change is huge for me, because I don't have to write Python2 anymore at all.

## Etcd

At the Forum in Boston, the user and developer communities decided to use etcd v3 as the distributed lock management solution for OpenStack, and integrations are starting to appear in the Pike release. This is exciting for me because I was in the room for some of those discussions, and I agree this is a huge step forward. While there are other things, like [Consul](https://www.consul.io/docs/commands/lock.html), that could be used for this purpose, etcd is not a bad choice, and makes sense given the desire for greater composability. If people are already running kubernetes, then they are likely running etcd.

## Improved Quota Handling

This is another one where I was in the room for the beginnings of the discussion in Boston. I think [the change in quote handling](https://specs.openstack.org/openstack/nova-specs/specs/pike/approved/cells-count-resources-to-check-quota-in-api.html) is a huge improvement over the previous implementation.

## CellsV2

We are using CellsV2, which on Ocata, means everything is in a single cell. While our current footprint is small compared to large installs like CERN, the addition of support for multiple cells in CellsV2 gives us a lot more flexibility in the design as we expand.

## Summary

There's a lot more, and it's worth digging in to the articles above to get all the details. There's RDO and SuSE packages available as I write this post, and I've been running a Pike beta via Kolla for about a month or so. The Canonical release to apt is also available, so regardless of your platform choice, Pike is here, and worth checking out!

