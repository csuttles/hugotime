+++
author = "Chris Suttles"
categories = ["OpenStack", "IPv6"]
date = 2017-04-03T13:11:16Z
description = ""
draft = false
cover = "/images/2017/09/glance.png"
slug = "openstack-image-bootstrap-on-ipv6"
tags = ["OpenStack", "IPv6"]
title = "OpenStack image bootstrap on IPv6"

+++


In my last post, I detailed a few things I experienced moving the Ocata release of OpenStack to an IPv6 only environment. While the major hurdles are complete, there are still more things to consider

## Image Automation

We build images using Jenkins and Packer, and push those images into glance, which stores them in a Swift backend. That's all still going smooth, but where we ran into trouble is the automation of configuration. When nodes are deployed, they are based on the images we build, and those images include a cloud-init assisted Chef bootstrap, driven by the metadata service. All of that works fine on a dual stacked environment, but when we moved to only IPv6, we ran into trouble. This design assumes that clients will run dual stack and that they can obtain an IPv4 address and contact the metadata service, and then use cloud-init to configure the hostname and IPv6 networking. 

## IPv6 Does Not Support the Metadata Service

After moving to IPv6 only, I quickly discovered that hosts were not getting configured correctly or bootstrapping with Chef. I started to research and investigate our images, and tried a few things with cloud-init before finding the bad news. 
> [There is no provisions for an IPv6-based metadata service similar to what is provided for IPv4. In the case of dual stacked guests though it is always possible to use the IPv4 metadata service instead.](https://docs.openstack.org/liberty/networking-guide/adv-config-ipv6.html)

This is bad news for me, because that last sentence is exactly how we were handling client configuration in dual stacked guests. This means that guests which are IPv6 only are out of luck.

A little more investigation led me to this request for enhancement on Launchpad: [[RFE] Support metadata service with IPv6-only tenant network](https://bugs.launchpad.net/neutron/+bug/1460177)

This seemed very promising, until I reached the most recent update.

> Comments on bug report seems to hint this effort has been abandoned. Please resume, if you intend to proceed

Since resuming the work there is not trivial, and there are other ways for us to solve this problem, we are going to go a different direction. Our current plan is to [leverage config-drive instead](https://docs.openstack.org/user-guide/cli-config-drive.html)

---

###### Update:

We ended up resolving our issues with slight changes to the builds (enabled DHCP for v4 and v6 on firstboot), and using config-drive for metadata.

* We set `force_config_drive=true` in nova.conf to enable config-drive whether the user specifies it or not.
* We were already using dhcp for ipv4, so we added `dhcpv6-stateful` for the `--ipv6_ra_mode` and `--ipv6_address_mode` options to have Neutron manage DHCPv6 as well.

This got images pulling dynamic addresses regardless of ip version, and the VMs could then read and set hostnames from the config-drive data. This is enough to get them to bootstrap in chef, where we finish configuration of the VM and change addressing from dynamic to static.

