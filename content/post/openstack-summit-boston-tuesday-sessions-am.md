+++
author = "Chris Suttles"
categories = ["OpenStack", "summit"]
date = 2017-05-20T01:39:59Z
description = ""
draft = false
cover = "/images/2017/09/openstack-summit-boston-3.jpeg"
slug = "openstack-summit-boston-tuesday-sessions-am"
tags = ["OpenStack", "summit"]
title = "OpenStack Summit Boston - Tuesday Sessions AM"
aliases = ["/openstack-summit-boston-tuesday-sessions-am/"]

+++


#### Learn about Keystone to Keystone federation and see it work with Horizon for Ocata

- Elvin Tubillara
- Nithya Renganathan
- [https://www.openstack.org/videos/boston-2017/learn-about-keystone-to-keystone-federation-and-see-it-work-with-horizon-for-ocata](https://www.openstack.org/videos/boston-2017/learn-about-keystone-to-keystone-federation-and-see-it-work-with-horizon-for-ocata)

The crux of this lightning talk was:

* Keystone instance "A" is source of identity
* Keystone instance "B" uses Keystone "A" as source ov identity via Shibboleth/SAML2.0
* Mappings are created for B->A. This only gets you identity, not ssh-keys or other Keystone components

For this to be successful, users must fully qualify their names, like `user@VagrantServiceProvider`

Keystone to Keystone is supported via:

* Python API
* Horizon
* CLI plugin

We got a demo where the presenters logged into horizon using `user@VagrantServiceProvider` style federated credentials. There was a drop down in the upper right of horizon where it was possibly to switch service providers.

Here's the official OpenStack docs on Keystone to Keystone federation: 

[https://docs.openstack.org/developer/openstack-ansible-os_keystone/ocata/configure-federation-wrapper.html](https://docs.openstack.org/developer/openstack-ansible-os_keystone/ocata/configure-federation-wrapper.html)

OpenStack docs on Federated Identity:

[https://docs.openstack.org/developer/keystone/federation/federated_identity.html](https://docs.openstack.org/developer/keystone/federation/federated_identity.html)

![](/content/images/2017/05/Photo-May-09--11-35-44-AM.jpg)

Also, here's a detailed from IBM on SSO, which covers a lot of the core concepts and terminology, even though it's not focused on Keystone to Keystone:

[https://www.ibm.com/developerworks/community/wikis/home?lang=en#!/wiki/W746177d414b9_4c5f_9095_5b8657ff8e9d/page/Keystone%20Horizon%20Single%20Sign%20On%20with%20SAML](https://www.ibm.com/developerworks/community/wikis/home?lang=en#!/wiki/W746177d414b9_4c5f_9095_5b8657ff8e9d/page/Keystone%20Horizon%20Single%20Sign%20On%20with%20SAML)


#### AT&T Cloud Evolution : Virtual to Container based (CI/CD)^2

- Milind Belhe
- Andrew Leasck
- Larry Rensing
- [https://www.openstack.org/videos/boston-2017/at-and-t-cloud-evolution-virtual-to-container-based-cicd2](https://www.openstack.org/videos/boston-2017/at-and-t-cloud-evolution-virtual-to-container-based-cicd2)

This lightning talk focused on AT&T's CI/CD pipeline evolution. Their biggest challenges included culture, lifecycle management, greenfield deployments and brownfield upgrades. They do patching based on OpenStack release candidates, via automated upstream syncs to the OpenStack code base. Once their images are built, they do a "simulated scale test" in their AVT lab, where they check for the types of problems that start to emerge at more than 300 nodes. They do a synthetic performance load test, and check for race conditions, as well as ensuring their master node does not suffer a DDOS during massive deployments. If all of this works, it generates a "golden config", which then traverses the same pipelines with fail gates. If that succeeds, it moves on to final verification test scripts. If these also pass, then the "golden config" is ready to be promoted to production.

![](/content/images/2017/05/Photo-May-09--11-51-15-AM.jpg)

![](/content/images/2017/05/Photo-May-09--11-54-32-AM.jpg)

Notable projects included in this presentation are:

* OpenStack Helm
 * [https://github.com/openstack/openstack-helm](https://github.com/openstack/openstack-helm)
 * [https://readthedocs.org/projects/openstack-helm/](https://readthedocs.org/projects/openstack-helm/)
* Armada
 * [https://github.com/att-comdev/armada](https://github.com/att-comdev/armada)
 * [https://wiki.opnfv.org/display/PROJ/Armada](https://wiki.opnfv.org/display/PROJ/Armada)

#### The Challenge of OpenStack Performance Optimization for 800 Nodes in Singe Region

- Hanchen Lin
- [https://www.openstack.org/videos/boston-2017/the-challenge-of-openstack-performance-optimization-for-800-nodes-in-singe-region](https://www.openstack.org/videos/boston-2017/the-challenge-of-openstack-performance-optimization-for-800-nodes-in-singe-region)

This talk focused on scaling at T2 Cloud, which is the largest public cloud in China. Some of the topics they covered regarding scale include:

* Using SSDs as caching layer in Ceph
* Overall architecture
![](/content/images/2017/05/Photo-May-09--12-11-46-PM.jpg)
![](/content/images/2017/05/Photo-May-09--12-13-40-PM.jpg)
* Conquering deadlocks in Galera with backend affinity (do not change backend within DB client session)
![](/content/images/2017/05/Photo-May-09--12-16-58-PM.jpg)
* Splitting MySQL reads and writes for better load balancing and performance
![](/content/images/2017/05/Photo-May-09--12-18-49-PM.jpg)
* Using local mode of RabbitMQ for better RPC performance during L2/RPC storms, also tune periodic task intervals
![](/content/images/2017/05/Photo-May-09--12-31-50-PM.jpg)
* Improving Keystone performance with Apache or NGINX ro the WSGI
![](/content/images/2017/05/Photo-May-09--12-34-53-PM.jpg)

While there was a lot of good technical content here, some of this information is dated, since it's focused on Liberty release and older versions of Galera. For example, local mode for RPC and tuning periodic tasks are still good tools, but in newer releases of openstack, the WSGI was already been moved under HTTP, so that part is basically irrelevant on Ocata.

