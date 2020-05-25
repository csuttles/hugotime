+++
author = "Chris Suttles"
categories = ["OpenStack", "summit"]
date = 2017-06-05T23:48:03Z
description = ""
draft = false
cover = "/images/2017/09/openstack-summit-boston.jpeg"
slug = "openstack-summit-boston-wednesday-sessions-pm"
tags = ["OpenStack", "summit"]
title = "OpenStack Summit Boston - Wednesday Sessions PM"
aliases = ["/openstack-summit-boston-wednesday-sessions-pm/"]

+++


#### Neutron Pain Points

- Kevin Benton
- [https://www.openstack.org/summit/boston-2017/summit-schedule/events/18801/neutron-pain-points](https://www.openstack.org/summit/boston-2017/summit-schedule/events/18801/neutron-pain-points)
- [https://etherpad.openstack.org/p/neutron-boston-painpoints](https://etherpad.openstack.org/p/neutron-boston-painpoints)
- [https://etherpad.openstack.org/p/pike-neutron-making-it-easy](https://etherpad.openstack.org/p/pike-neutron-making-it-easy)

* Octavia LBaaS
 * People were looking for an update on status, but noone was there to speak on it
 * Main contributor moved on, but project is still maintained, even though it was not represented in this session

* FWaaS v2
 * L3 supported since Newton
 * L2 was targeted for Ocata, now targeted for Pike
 * 6-9 contributors total, they solicited more
 * Sustainability billed as "OK"
 * Current focus was stadium project requirements (gate test coverage, etc)
 * No migration path for v1->v2 at time of meeting, will be added
 * "FWaaS v2 is security groups on steroids"

* Neutron does not know when tenants are created
 * This came up in discussion of default rule management and is worth understanding when planning tenants and automation

* Security Groups include anti-spoofing, and this is broken when SG is disabled for a tenant/network.

Issues!

* No 'default default' SG
 * One user was very vocal about wanting to be able to define the default security group rules so that when a new tenant is created the rules would suit their use case. This dominated the discussion and included suggestions like an increment of the API version when adding support, automating the fix at project creation time (the unanimous choice for everyone but the guy asking), and the gem "use the source luke".
* Port Security (anti-spoofing) and SG are quite intertwined and the disabling of SG disabled port security. Operators voiced the desire to control each independently.
* A migration path from v1->v2 is needed for operators on v1
* Yahoo operators asked for DHCP support of something other than dnsmasq (kea and ISC DHCP were suggested, kea being the favorite)
 * Looking for volunteers to help code and test
* MTU needs to be carried across namespaces (currently bugged)
* Operators should be able to disable DNS option in DHCP leases when no DNS on Neutron subnet
* Request for support for more than ICMP, TCP, UDP; some of this is there, and you can use protocol numbers, but there are no docs (use the source, Luke). Looking for contributors to write docs. Also some details/related stuff in wiki: [https://wiki.openstack.org/wiki/Neutron/TrafficProtection](https://wiki.openstack.org/wiki/Neutron/TrafficProtection)
* Operators expressed interest in removing group to group rules in the default ruleset. My notes here include the phrase "database explosion". I think the concern is that in large multi tenant environments, these rules can eat up a lot of space in the DB and are mostly unused and unnecessary

#### Kuberneterize Your Baremetal Nodes in OpenStack!

- Ken Savich
- Darin Sorrentino
- [https://www.openstack.org/videos/boston-2017/kuberneterize-your-baremetal-nodes-in-openstack](https://www.openstack.org/videos/boston-2017/kuberneterize-your-baremetal-nodes-in-openstack)

This session kicked off with a bunch of live polls:

* Who runs baremetal/ironic: 30%
* Who runs kubernetes on Openstack: 26%
* Who's heard of the term 'kuberneterize': 15%

The session then launched into the plan. The K8s master was to be a VM, and the minions to be bare metal;  OpenStack Platform 10 (RedHat's Newton release) was  to run atop. The OS was RHEL 7.3, Kubernetes was to be version 1.6.x and Docker version was to be 1.12. 

> "Everybody has a plan until they get punched in the face." - Mike Tyson

If you have been astute, you might have noticed my choices in the language used to describe this talk. The versions are all "to be" because the subject of this talk never succeeded at becoming anything more than a plan.

There's more to this talk if you care to watch it, but with all the content at the summit, a talk without a working demo was a disappointment, particularly after meeting Ken the day before; he was excited about the demo and said that he was working through some last minute details. Unfortunately, the pitfalls of bare metal prevented the demo's success. There's some good material in there if you are trying to do something similar, but I just thought this sounded neat and told Ken I would attend, so it didn't give me much value.

#### Neutron Diagnostic Tools

- Armando Migliaccio
- [https://www.openstack.org/summit/boston-2017/summit-schedule/events/18758/neutron-diagnostic-tools](https://www.openstack.org/summit/boston-2017/summit-schedule/events/18758/neutron-diagnostic-tools)
- [https://blueprints.launchpad.net/neutron/+spec/troubleshooting](https://blueprints.launchpad.net/neutron/+spec/troubleshooting)
- [http://lists.openstack.org/pipermail/openstack-dev/2016-May/094201.html](http://lists.openstack.org/pipermail/openstack-dev/2016-May/094201.html)

The goal of this session was to talk about a diagnostic check API for neutron. It's detailed in the linked spec, and is targeted for Pike release.

Proposed checks include:

* DHCP
* Tunnels
* Security Groups
* Dropped packets on hosts
* MTU mismatch
* conf validation
* Floating IPs
* IPv6 RA
* Metadata service (is it reachable)
* host to host reachability on overlapped IP addrs

What do operators currently do?

* Diff platform vs Neutron (so they match)
* Remove discovered orphaned objects through manual cleanup
* Users select the source of truth
* For Openvswitch and ML2, they gather all DHCP/IPaddrs and ping/check namespaces

This discussion was good, and I'm glad to see the community driving this, since this is a pretty important addition to Neutron. Manual checks are a poor way to maintain things of importance.

#### Common Networking Operations Across Kubernetes and OpenStack​ with Calico

- Mark Baker
- Karthik Prabhakar
- Larry Rensing
- [https://www.openstack.org/videos/boston-2017/common-networking-operations-across-kubernetes-and-openstack-with-calico](https://www.openstack.org/videos/boston-2017/common-networking-operations-across-kubernetes-and-openstack-with-calico)


I felt this was a stand out, and one of the best sessions of the summit. The presentation detailed using Kolla and Docker for the control plane, and LXD, Kubernetes or VMs for the data plane. Canonical touted the flexibility and compatibility of deploying the pair, either side by side, with Kubernetes on OpenStack and Openstack on Kubernetes. Tigera highlighted their contributions to CNI and Flannel.

The Calico approach is pure layer 3, and fits well with the [CLOS model](https://comeroutewithme.com/2015/02/28/why-is-clos-spineleaf-the-thing-to-do-in-the-data-center/). It's plugin driven, and supports CNI, Neutron, and a few other plugins. There's no overlay, routes are advertised by Felix and Bird/Bird6, and config/state is stored in [etcd](https://github.com/coreos/etcd). Security groups are mapped to profiles in etcd, which allows for dynamic adjustment, and ipsets are using in iptables for better performance. Policy and profiles can be extended to bare metal Linux nodes.

The demo for the presentation was very similar to the Helm demo, a workload VM was pinged continuously, and video was played from a webserver on it as well. The OpenStack release was upgraded to Newton, while Keystone was hammered with requests, and kubectl get pods was repeatedly run to show status. Rolling updates were applied pod by pod, and 0% packet loss occurred, and latency was not bad either (based on ping summary). No dropped frames were sen in the video. Database dependencies were handled via kolla-toolbox.

My QA notes include "can you have overlapping IP addresses". The answer was no.

