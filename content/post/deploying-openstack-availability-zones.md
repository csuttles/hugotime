+++
author = "Chris Suttles"
categories = ["OpenStack"]
date = 2017-04-12T13:28:37Z
description = ""
draft = false
cover = "/images/2017/09/openstack-availability-zones.jpg"
slug = "deploying-openstack-availability-zones"
tags = ["OpenStack"]
title = "Deploying OpenStack Availability Zones"
aliases = ["/deploying-openstack-availability-zones/"]

+++


We added availability zones to our deployment for a few reasons.

#### Fault domain isolation

The foremost of these was to isolate fault domains within our environment. Spreading workload across availability zones allows us to ensure that the applications and services provided by the OpenStack workload are resilient.

#### Influencing placement

Availability zones afford us more granular control of scheduling, without modifying configs or changing the default scheduler hints or filters. This allows us to be specific enough about placement to steer things in an intelligent way, while avoiding the burden of placing everything manually. We can specify a specific host (this is actually possible with just the default 'nova' zone), or we can specify the AZ and let the schedulers and placement API make the decisions within that AZ, which is quite useful. This allows us a lot of flexibility and makes planning for growth a little easier, since we specified default and fallback AZs.  

#### Localized storage traffic

We are using cinder with multiple back-ends, and one of those back-ends is LVM. With the LVM back-end in cinder, a logical volume is created on the storage host running the cinder-volume service, and that volume is exported as an iSCSI target for consumption by the initiator where nova/libvirt is running. While this works, it has some drawbacks in our environment, since the network is shared by other things. While I have made attempts to isolate block storage to a separate network, it just does not make sense with our datacenter design, and the teams that are stakeholders (other than me) will simply not bend to accommodate this idea. This leaves us with the classic challenges presented by block storage on a shared network. The most significant of these challenges for us was ToR (Top of Rack) uplink bandwidth. Our datacenter network is fast, but also oversubscribed from ToR to ToR, and from edge to core, as are most datacenters, because bandwidth costs money. Using availability zones, and setting `cross_az_attach = False` in the `[cinder]` stanza of nova.conf allows us to keep block traffic from traversing those precious uplinks, which alleviates the most problematic form of congestion we face due to block traffic traversing the same network as other traffic.

#### References

I found the following Mirantis article to be incredibly useful when considering our AZ deployment: 

[The first and final words on OpenStack availability zones](https://www.mirantis.com/blog/the-first-and-final-word-on-openstack-availability-zones/)

I also found this portion of the OpenStack documentation useful for both testing and influencing placement:

[Select hosts where instances are launched](https://docs.openstack.org/admin-guide/cli-nova-specify-host.html)

