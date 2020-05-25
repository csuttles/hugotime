+++
author = "Chris Suttles"
categories = ["OpenStack", "IPv6"]
date = 2017-03-26T09:58:31Z
description = ""
draft = false
cover = "/images/2017/09/ipv6-ipv4.jpg"
slug = "ocata-and-ipv6"
tags = ["OpenStack", "IPv6"]
title = "Ocata and IPv6"
aliases = ["/ocata-and-ipv6/"]

+++


Since I took the lead on the OpenStack deployment in my workplace, IPv6 has been a major goal of our deployment. IPv6 is a first class citizen in our infrastructure, and the other environments that OpenStack coexists with are already running IPv6 only. This made getting OpenStack to run IPv6 only a major milestone, which was finally accomplished during our move to Ocata release.

##### Standards and SSL Termination

While running client workload with IPv4 or dual stack is acceptable, or even required in some cases (vendor packaged VMs are notoriously guilty here), the standard for deploying infrastructure is high. IPv6 only is required. SSL/TLS is required for all API endpoints, and certificates must be issued from our PKI and trusted. No snake oil certificates or self-signed monkey business is allowed.

###### HAProxy

I tackled the SSL/TLS requirement with HAProxy. It's very capable of handling the level of traffic we do over the OpenStack APIs, and it's easy to provide our internal certificates for each endpoint. HAProxy binds to the IPv6 address, and "load balances" to a single backend, the localhost, where OpenStack API services actually bind. This approach is outlined with different software in the examples on openstack.org (nginx, pound, stud, and apache), but the architecture is the same one outlined in the [OpenStack Security Guide](https://docs.openstack.org/security-guide/secure-communication/secure-reference-architectures.html). 

##### Client Side

In previous releases, I ran into problems client side with a few things defaulting to the A record instead of the AAAA as they should, [according to the proposed standards](https://tools.ietf.org/html/rfc6724#section-10.3) 
I only hit one snag on the client side in Ocata release. In [cinder.conf](https://docs.openstack.org/ocata/config-reference/block-storage/samples/cinder.conf.html), you need to specify the following parameter using [the bracket presentation format](https://tools.ietf.org/html/rfc4038#section-5.1):

`iscsi_ip_address = [$my_ip]`

Without the brackets, I ran into problems provisioning VMs, and saw that during drive mapping, when `iscsiadm -m discovery -t sendtargets -p target_IP:port` was run, the IPv6 address was not interpreted correctly. I believe what happens is that iscsiadm doesn't parse the port correctly. Regardless of what's happening at the iscsiadm level, when the discovery failed, that ultimately would exhaust all retries in Cinder. Exceptions were thrown, the volume that couldn't be discovered was deleted, and the VM would go to an ERROR state, because block allocation failed.

I didn't dig into iscsiadm too much, because I'm pretty sure I have already seen this issue bugged, and specifying the bracket format fixed the issue with Cinder, which was really my goal.

##### Server Side

I had a problem with orphaned A records causing RabbitMQ to fail to start because it tried to bind to an IPv4 address that was not present, based on a gethostbyname() call. I removed the orphaned A record (the hosts were previously dual stack), and once the TTL expired, RabbitMQ started with only the IPv6 address, as it should have done up front. I probably could have worked around this with rabbitmq.config, but I didn't want to automate extra config if I could avoid it, and cleaning up clutter from our IPAM is a better choice than automating around orphaned DNS records. Our IPv4 space is at a premium anyway, so giving that back was a better choice in several ways. 

##### Minor changes

Other than the hiccup with cinder and iscsi mentioned above, and the  the move to IPv6 with Ocata was remarkably smooth. I did some code mods and changed `0.0.0.0` to `::` throughout our code base used for deployment, which fixed most things that are not running TLS termination in HAProxy. We are currently deploying to bare metal via Ansible, but we are in the process of migrating to Chef, because it is the company standard for config management and supports many internal tools that are useful. After just a couple deployments to test servers, all the issues were resolved and I was finally able to announce that we successfully moved to IPv6 only for OpenStack's core services. We do still see IPv4 addresses on machines running neutron DHCP agents for IPv4 subnets, but that's expected and accepted.

