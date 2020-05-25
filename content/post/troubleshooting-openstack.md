+++
author = "Chris Suttles"
categories = ["OpenStack"]
date = 2017-03-07T13:56:21Z
description = ""
draft = false
cover = "/images/2017/10/COMPUTER-REPAIR.jpg"
slug = "troubleshooting-openstack"
tags = ["OpenStack"]
title = "Troubleshooting OpenStack"

+++


#### Everything begins with Keystone

This is nothing new, but it's important to point out. Nothing works if Keystone is not set up properly. The easiest way to check if Keystone and your client are set up correctly is to run:

{{< highlight markdown >}}
openstack token issue
{{< / highlight >}}

An often overlooked part of Keystone is the service catalog. Keystone is responsible for authentication and authorization in OpenStack, but it's also responsible for the service catalog. If you run the following, you'll see the contents of the service catalog:

{{< highlight markdown >}}
openstack endpoint list
{{< / highlight >}}

It's important that this is accurate. The URLs listed in the service catalog must be accurate, because if they are wrong, your services will fail to communicate. In many cases, you supply these URLs in config files, and those should always match the service catalog as well.

##### Focus on the component

If Keystone is working, it's time to look closer at the component that is giving you trouble. Here are some general steps that are helpful regardless of the component:

* Do the URLs in the config match the service catalog?
* Is the port available and listening as expected?
 * `ss -ntap` is useful here
 * Make sure you are listening on the right interface, or all interfaces. `::` is better than `0.0.0.0`, as it includes both ipv4 and ipv6.
* Can you connect to the port from the remote host if it is listening?
 * `echo 'blah'| nc $dest $dport` or `telnet $dest $dport` are helpful here
 * Don't forget to check on name resolution here. `dig +short +search $dest a` and `dig +short +search $dest aaaa` are helpful here.
* Enable verbose or debug logging, reproduce the error, and watch the logs.
* Can the service communicate with the database backend and the message transport, and are those services healthy?
* Most components have a status command, which shows the status of the services that make that component. What does that command say?
 * `openstack compute service status`
 * `openstack network agent list`
* Does your problem include every operation, or only certain types? If you can list volumes, but not create volumes, your next steps will be different than if you cannot list and cannot create volumes, for example.

Every component is different, so steps will vary widely, but all of these steps will lead you to more specific troubleshooting. It's also crucial to understand how components in OpenStack communicate. There's a very simple guideline.

* Components communicate with other components via API   
  * example: cinder to keystone
* Components communicate with other services in the same component over message queue, via rpc.cast or rpc.call
  * example: nova-api to nova-scheduler
* Components communicate with the database for storage of persistent state data
  * example: When a node is created, it's status is stored in the nova db (other things happen too, but that's outside the scope of this example)

So, if you are looking into failures within nova, and only nova, you might be interested to look into message queues for nova, and double check that all the nova services can access whatever transport you are using for message queueing (typically rabbitmq).

Also, here's a very handy reference for how the process of requesting a VM works:

https://www.slideshare.net/mirantis/openstack-architecture-43160012

