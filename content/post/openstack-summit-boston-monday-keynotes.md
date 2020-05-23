+++
author = "Chris Suttles"
categories = ["OpenStack", "summit"]
date = 2017-05-10T12:41:25Z
description = ""
draft = false
cover = "/images/2017/09/openstack-summit-boston-7.jpeg"
slug = "openstack-summit-boston-monday-keynotes"
tags = ["OpenStack", "summit"]
title = "OpenStack Summit Boston - Monday Keynotes"

+++


##### GEIX / GE Healthcare
They shared their success stories with OpenStack, which included reducing the time required for some medical imaging from over 1 day to minutes. This was in partnership with a research endeavor they sponsor.

##### Taking OpenStack to the Network Edge (Verizon)
Verizon previewed their OpenStack in a box CPE, which was 1U, LTE capable with four antennas, and runs OpenStack in containers. It's part of their globally, massively distributed OpenStack footprint.

They talked about their focus on:
 
* Automation
* Self Healing
* Orchestration

We then got a demo of CPE provisioning and maintenance of sites, in a heavily customized interface, which included circuit management and geolocation. 

###### US Army Cyber School
The US Army education representative spoke about their BBH (BroadBand Handrail) program, which is an agile framework for publishing courseware via CI and teaching students about OpenStack, and focuses on infrastructure as code and courseware as code.
Their Alpha environment, which they plan to expand is currently:

* ~2000 cores (started as 3 commodity boxes on a table!)
* 36TB RAM 

We saw a live demo where a change was made to courseware source, which was in a language similar to markdown. This triggered the CI build pipeline to re-render the courseware as PDF, and we saw the infrastructure in OpenStack re-deploy the related heat template as well.

###### Mirantis
This started with an animated video that was a bit of a mashup of comic books and sci-fi. I didn't care for it. The basic premise was that "cloud city" was dying and being terrorized by the evil "Doc Locke", a villain who traps people in his "black box". Mirantis, of course, saves the day. After this intro, which I would call mediocre at best, [Boris](https://www.linkedin.com/in/borisrenski/) walks out on stage holding a "black box" like the one depicted in the video.

Boris then spoke, at great length, about the term private cloud, and why it has negative connotations. He suggested instead, that we all adopt the term "managed open cloud" instead. I believe this presentation carried on longer than the allotted time and put the keynotes behind schedule, but I didn't really check the time, so perhaps that is not the case.

Boris then got on track and made some very good points (and some obvious ones), including:

* Clouds costs money
* Software costs money
* Vendors want to own you with their ecosystem
* Vertical design patterns that rely on open source all the way up the stack are better than stitching together vendor products
* Even a single vendor product dependency in your stack dramatically reduces your flexibility and forces you toward a waterfall release model, which is bad
* Open source and open design is important
* Lag of vendors prevents quick upgrades

Lastly, Mirantis/Boris announced a partnership with Fujitsu. A Fujitsu exec spoke briefly, and then together, she and Boris crushed the black box he brought out on stage.

###### AT&T / DirecTV

These two presenters spoke about the importance of decoupling product from infrastructure. They also provided some details on their OpenStack project, which runs their entertainment workload as microservices Using OpenStack. They chose OpenStack instead of containers because of the CPU and storage requirements of their workload.

Their environment is something like this:

* Mitaka release
* CI/CD
* Baremetal (for rendering workload)
* Hybrid cloud includes Kubernetes and Docker for microservices

Next, we saw a demo, which included streaming DirecTV channels without any special device. It was pointed out that HD frames that made the video stream of the channel we watched were moments ago only jumbo frames flying through this system, rendered in real time to become our video stream.

###### eBay - Scaling Kubernetes on OpenStack
Things kicked off with the "big money slide", which demonstrated eBay's considerable OpenStack footprint.
Their multi-tenant OpenStack environment includes:

* ~167,100 VMs
* 13PB storage

This environment runs 95% of all eBay traffic.

They talked about their first use of Kubernetes in 2015. That environment is now:

* ~22k cores
* 6 Availability zones
* Powered by OpenStack

It runs stateless applications, also many stateful applications, including the following:

* ELK (elasticsearch,logstash, kibana)
* Kafka
* NoSQL

Next they talked about their project for managing Kubernetes across multiple providers (including OpenStack), [TessMaster](http://www.informationweek.com/cloud/infrastructure-as-a-service/ebay-container-management-tool-works-with-kubernetes/d/d-id/1327782) (or Tess.io). They described it as roughly a Kubernetes for Kubernetes. This was followed by a live demo, where Kubernetes clusters were expanded from the web interface. They touted support for multiple groups, as well as multiple regions and datacenters. The announced that it will be open sourced within a couple of months.

###### RedHat

There were two RedHat segments back to back, and the first was a fireside chat with [James Whitehurst](https://www.redhat.com/en/about/company/management/james-whitehurst). He talked about his book ["The Open Organization"](https://www.redhat.com/en/explore/the-open-organization-book), as well as where RedHat is going, while touching on the direction of the OpenStack community. He said that cultural challenges are often the most difficult and slow problems to solve, and he emphasized the importance of staying close to upstream for success. This chat was cut very short, because at this point of the morning the keynotes were about 20 minutes off schedule.

Next was a talk by the [RedHat CTO Chris Wright](https://www.linkedin.com/in/chris-wright-b733851/). He highlighted the increased number of OpenStack installations in production in this year's user survey (65%), and he talked about the increased adoption of RedHat's OpenStack offering, complete with the customer slide full of logos.

He also spoke about [MOC (Massachusetts Open Cloud)](https://info.massopencloud.org/), which is a research collaboration project sponsored by RedHat. Highlights for this installation of OpenStack include:

* Supported by only three staff members
* 700TB of Ceph based storage

He also spoke about hybrid cloud, and using OpenStack with Kubernetes or OpenShift.

