+++
author = "Chris Suttles"
categories = ["OpenStack", "summit"]
date = 2017-05-25T12:02:07Z
description = ""
draft = false
cover = "/images/2017/09/openstack-summit-boston-2.jpeg"
slug = "openstack-summit-boston-tuesday-sessions-pm"
tags = ["OpenStack", "summit"]
title = "OpenStack Summit Boston - Tuesday Sessions PM"
aliases = ["/openstack-summit-boston-tuesday-sessions-pm/"]

+++


#### Containers Networking Using Kuryr: A Hands-On Lab

- Sudhir Kethamakka
- Amol Chobe
- [https://www.openstack.org/summit/boston-2017/summit-schedule/events/18385/containers-networking-using-kuryr-a-hands-on-lab](https://www.openstack.org/summit/boston-2017/summit-schedule/events/18385/containers-networking-using-kuryr-a-hands-on-lab)

* Kuryr is a part of the [OpenStack Big Tent](http://drbacchus.com/bigtent/)
* Kuryr is a bridge between container frameworks and Neutron
* Supports [CNI](https://github.com/containernetworking/cni) and [CNM](https://thenewstack.io/container-networking-landscape-cni-coreos-cnm-docker/)
 * Docker
 * Kubernetes
 * Rkt
 * CoreOS

![](/content/images/2017/05/Photo-May-09--2-19-49-PM.jpg)

The lab focused on the CNM model. Both CNM and CNI support are plugins in Kuryr, so it's possoble to extend Kuryr to support other network models by writing a plugin at this layer. Kuryr ties Docker to Neutron, and gets you all the good Neutron things in you docker networks (Security Groups, FWaaS, LBaaS, etc).

* Kuryr has plans to converge with Magnum, so Kubernetes pods/clusters spun up under Magnum will be able to leverage Kuryr and use Neutron as well.
 * [https://etherpad.openstack.org/p/magnum-kuryr](https://etherpad.openstack.org/p/magnum-kuryr)
 * [https://blueprints.launchpad.net/kuryr/+spec/containers-in-instances](https://blueprints.launchpad.net/kuryr/+spec/containers-in-instances)
* Kuryr support pluggable IPAM as well

During the course of the lab everyone spun up an all-in-one OpenStack VM, with Kuryr and Docker pre-configured. We then created networks in OpenStack and Docker, and verified they existed in both, as well as verifying ports allocated in Neutron and Security Groups were applied. Overall, I really enjoyed this lab and it was the best lab I attended during the summit.

#### Cinder node Resiliency (Handling Media, Network and Software Failures)

- Dhruv Bhatnagar
- Nirendra Awasthi
- [https://www.openstack.org/videos/boston-2017/cinder-node-resiliency-handling-media-network-and-software-failures](https://www.openstack.org/videos/boston-2017/cinder-node-resiliency-handling-media-network-and-software-failures)

This lightning talk was a poorly veiled advertisement for Veritas software, under the guise that it would help improve Cinder resiliency. The design was poor, or poorly explained, or both, and it was a big disappointment.

Essentially, you run the Veritas software on all Cinder nodes, and use it to create redundancy in case of failure by peering nodes and syncing them, at the cost of significant complexity increase, IO load increase, and significantly reducing your usable space. They use PIT snapshots of VMS and deltas of data on Cinder nodes to achieve redundancy, but they did not have a good explanation of tracking state, or recovery under poor conditions (how much data is lost between snapshots), and the recovery process included manual steps like "repoint things at the peer Cinder node".

It included this gem "There's no data loss in this architecture, pretty much."

When I talk about data, I want certainty, on explicit terms, not phrases like "pretty much". This was without a doubt the worst lightning talk I attended.

#### FWaaS v2: A New Beginning

- Sridar Kandaswamy
- Yushiro FURUKAWA
- Chandan Dutta Chowdhury
- [https://www.openstack.org/videos/boston-2017/fwaas-v2-a-new-beginning](https://www.openstack.org/videos/boston-2017/fwaas-v2-a-new-beginning)

This talk included two cores and on contributor. They discussed the differences between FWaaS v1, which debuted in Havana release, and the new FWaaS v2, as well as the future plans for FWaaS v2.

FWaaS v1: 

 * Havana release deployed on all routers
 * Refactored for Kilo to use only user specified routers
 * Router ports was proposed, but never made it into FWaaS v1

Code base was refactored again for FWaaS v2.

FWaaS v2:

 * Router Port association
 * Interest in L2 Ports, but not ready yet
  * Targeted for Ocata, but did not make it. Retargeted for Pike
 * Multiple layers of enforcement, applied after Security Groups
 * API available since Newton

Primitives are:

 * Group (made of one or more policies)
 * Policy (ingress or egress, made of one or more rules)
 * Rule (traffic specs and actions like: tcp any any deny, etc)

![](/content/images/2017/05/Photo-May-09--3-50-25-PM.jpg)

The data model for L2 and L3 is the same, and their planned approach for L2 is therefore very similar to the approach for L3, using iptables and ipsets in a network namespace.

Currently they are working on meeting Neutron stadium project requirements:

* CI/CD
* Stable backports
* Neutron integrations
* Client libraries

We saw some slides walking through L2 filtering in detail, and they explained the default firewall group, which is similar to the default security group, and is applied when no other firewall group is present (possibly after all applied firewall groups as well, this wasn't clear to me from the presentation). They were debugging some port update issues, and co-existence with Security Groups at the time of the summit, and they are targeting Horizon support for Pike.


#### Easier OpenStack Deployment: Novice Installation Using Kolla

- Michał Jastrzębski
- [https://www.openstack.org/summit/boston-2017/summit-schedule/events/18177/easier-openstack-deployment-novice-installation-using-kolla](https://www.openstack.org/summit/boston-2017/summit-schedule/events/18177/easier-openstack-deployment-novice-installation-using-kolla)

This lab was billed as follows: 

> In this session, we will provide participants the opportunity to utilize our guide and complete a full production grade deployment of OpenStack using Kolla.

What I got was very different. Many students had trouble with the lab content, myself included. With a room full of students pounding the lab servers, the people who could connect had terrible latency, and many simply could not reach their lab servers. I was fortunate; I read head and got started on the lab during the presentation, but I was quickly sidelined by lab instructions that did not work as expected. I worked through some of the issues, as did other students, some submitting pull requests to fix the lab content. I gave it a go for about an hour, and when it became clear the whole class was stalling, I cut my losses and left.  The repo for the lab has been updated 8 days ago, as of this writing, and the pull requests submitted during the lab have been integrated, so maybe it works now. Here's the repo: [https://github.com/inc0/kolla-ansible-workshop](https://github.com/inc0/kolla-ansible-workshop)

