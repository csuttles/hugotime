+++
author = "Chris Suttles"
categories = ["OpenStack", "IPv6", "availability"]
date = 2017-05-01T01:23:53Z
description = ""
draft = false
cover = "/images/2017/09/rabbitmq-clustering.jpg"
slug = "clustering-rabbitmq-on-ipv6-only-with-openstack-ocata"
tags = ["OpenStack", "IPv6", "availability"]
title = "Clustering RabbitMQ on IPv6 with OpenStack Ocata"

+++


As part of our efforts to improve the resilience of OpenStack within sites, we are moving to a multi controller architecture. I decided to tackle the underlying, stateful infrastructure services first. The first of those I looked into was RabbitMQ.

RabbitMQ supports clustering, and HA queues, as well as durable queues (queues that persist to disk). The setup seems fairly simple, but unfortunately many things still assume IP means IPv4, and that's exactly where this got interesting.

When I read through the [OpenStack HA guide](https://docs.openstack.org/ha-guide/shared-messaging.html), I saw that it's a small number of steps to get things working. The guide is a little outdated, since it still suggests using the `rabbit_hosts` parameter, but I decided to give it a try and see if I could follow it without modification.

###### IPv6, Erlang, RabbitMQ, and You

I tried following the guide, and the `rabbitmqctl join_cluster` command failed every time. I tried several suggested workarounds (adding the short hostname to `/etc/hosts` was one, for example), but nothing seemed to make any difference. I verified that the `erlang.cookie` was the same on all hosts. I checked to verify ports were listening. I checked for firewalls in my path and connected with `nc` and `telnet` with no problems to the destination port on the remote host. After all of this, I was no closer to a cluster. I decided to dig into the process a bit by wrapping it with strace (yes, there are better ways, but strace is easy, and I didn't care about the performance hit in this case). I ran `strace -f -e trace=network rabbitmqctl join_cluster <myhost>` and found that even though both hosts are IPv6 only, and that even dual stack hosts *should* try IPv6 first and then fall back to IPv4, the request was failing because all of the network activity was trying to use IPv4.

I searched, and found [this bug for TripleO](https://bugzilla.redhat.com/show_bug.cgi?id=1358311) which suggested I was not alone in finding difficulty clustering RabbitMQ with IPv6 only. I tried upgrading packages, as suggested in the bug, but that did not resolve the issue. Finally I dug through the RabbitMQ mailing list and found [this gem](https://groups.google.com/forum/#!msg/rabbitmq-users/zvarY4RkUlw/iuTmNSVxAgAJ;context-place=msg/rabbitmq-users/r-mirK1XFEI/9r_h7TIFBAAJ), wherein it is explained that the problem is in the underlying Erlang distribution and includes the needed config changes to tell the Erlang server that supports RabbitMQ to us IPv6 only.

I verified this solved my connectivity problems, and after some more testing I was able to remove the other workarounds I had tried. I pulled the short hostnames from `/etc/hosts` (actually Chef did that for me, so thanks Chef), and removed the packages from RabbitMQ team I installed manually, and reverted to the packages in my upstream apt repo (Ubuntu). I tested again and was able to join the cluster. It was one of those moments where I tend to hit the `enter|return` key a lot harder than necessary to indicate triumph.

###### Updated Syntax for Messaging

Now that I finally had a working cluster, since that connectivity issue was the last real hurdle and the guide was easy to follow after resolving it, I needed OpenStack to actually use it.
The HA guide still uses the deprecated `rabbit_host` syntax, and I needed to find a better way. I was able to find the `transport_url` equivalent [in the OpenStack Questions forum](https://ask.openstack.org/en/question/99978/how-to-set-transport_url-properly-to-use-ha-rabbitmq-cluster/). After creating the cluster, setting ha policy for all queues, and updating transport_url across controllers and compute nodes, and restarting services, I was able to execute the definitive test. I stopped RabbitMQ on the main controller, and created a VM. Once it launched successfully, I started RabbitMQ and repeated the process, stopping the other nodes, and observing the output of `rabbitmqctl cluster_status` and `rabbitmqctl list_queues` along the way.

The last task was to update our automation so this can be done in a repeatable way. I also verified that the RabbitMQ changes are backward compatible with the single node RabbitMQ for good measure, since we still deploy single controller instances for some testing needs.

