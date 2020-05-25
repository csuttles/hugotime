+++
author = "Chris Suttles"
categories = ["OpenStack", "summit"]
date = 2017-05-14T10:15:26Z
description = ""
draft = false
cover = "/images/2017/09/openstack-summit-boston-4.jpeg"
slug = "openstack-summit-boston-tuesday-keynotes"
tags = ["OpenStack", "summit"]
title = "OpenStack Summit Boston - Tuesday Keynotes"
aliases = ["/openstack-summit-boston-tuesday-keynotes/"]

+++


#### Home of Open {Composable} Infrastructure

- [Mark Collier](https://www.linkedin.com/in/markcollier/)
- [https://www.openstack.org/videos/boston-2017/home-of-open-composable-infrastructure](https://www.openstack.org/videos/boston-2017/home-of-open-composable-infrastructure)

Mark talked about hacking on Apple IIs and cutting class to play Doom, and how computing has evolved to become the foundation for important human achievements like science,  industry, going to mars, or curing disease. He talked about the using open source first for solving technical problems, and the  importance of AI, and machine learning. 

Some of the open source platforms he talked about included:

* Big Data: Apache Spark & Hadoop
* Realtime: Kafka
* CI/CD: Spinnaker, Jenkins, TravisCI, Zuul
* App Management: Kubernetes
* Cloud database: CockroachDB

He emphasized how these applications are all composable and cloud native, meaning they are intended to be used in a modular way, so there are well defined interfaces to connect them to other parts of infrastructure.

He noted that the summit marked the 7th year of OpenStack, and talked about how OpenStack is a collection of projects, not just the 'big tent', and he introduced new mascot icons for every project. He also touched on reducing complexity when possible, because it's important to make things easy to consume and compose. He also mentioned the [NIH](http://macro.media.mit.edu/share/NotInventedHere.pdf) syndrome, and how it is a trap that impedes innovation and success, and how tools are just tools. He followed this with communicating the need for distributed locking within OpenStack, and how the community has decided not to reinvent the wheel, and embrace [etcd](https://github.com/coreos/etcd) for this purpose.

#### Live Demo: Just Ironic and Neutron

- Julia Kreger 
- [https://www.openstack.org/summit/boston-2017/summit-schedule/events/18847/live-demo-just-ironic-and-neutron](https://www.openstack.org/summit/boston-2017/summit-schedule/events/18847/live-demo-just-ironic-and-neutron)

A Dell EMC rack stands on the stage. Julia walked out and demonstrated the status of the rack in both OpenStack and Kubernetes, where there were a few baremetal nodes powered down in OpenStack (running just Ironic and Neutron), and a Kubernetes node down. The nodes get powered on and brought up, respectively. This demo was only about 3 minutes.

#### Deploying Cinder as a Stand-Alone Service Using Containers

- John Griffith
- Kendall Nelson  
- [https://www.openstack.org/summit/boston-2017/summit-schedule/events/18850/deploying-cinder-as-a-stand-alone-service-using-containers](https://www.openstack.org/summit/boston-2017/summit-schedule/events/18850/deploying-cinder-as-a-stand-alone-service-using-containers)

John and Kendall join Julia and Mark on the stage, and talk through adding Cinder (without Nova). They deploy Cinder in Docker compose to have a standalone Cinder service (no Keystone, no Nova). They spoke about [LOCI](https://github.com/openstack/loci) project which builds lightweight [OCI](https://github.com/opencontainers/image-spec) images for individual projects, and that these can be used by simply attaching a database and message queue. Then they demonstrate creating a volume in Cinder, where they ran into some timing issues, and although the could create the volume, they were unable to attach it. I'm sure this could have been sorted out, but they had about a 5 minute slot for this demo.

#### Unified Platform VMs, Containers, Bare Metal

- Jakub Pavlík
- [https://www.openstack.org/summit/boston-2017/summit-schedule/events/18854/unified-platform-vms-containers-bare-metal](https://www.openstack.org/summit/boston-2017/summit-schedule/events/18854/unified-platform-vms-containers-bare-metal)

Jakub introduces his demo, and describes his infrastructure, which is Kubernetes, Nova, and baremetal, all connected via Open Contrail. Atop this infra, [Spinnaker](http://www.spinnaker.io/) orchestrates the clouds, which includes a streaming API poller for Twitter, ingested into [Apache Kafka](https://kafka.apache.org/), processed in [Apache Spark](http://spark.apache.org/), and finally results are stored in [HDFS](https://hortonworks.com/apache/hdfs/). The poller and Kafka run on containers, Spark runs in Nova, and HDFS is on baremetal.

Most things were pre-deployed because the demo time is short, but poller part of the pipeline was deployed live and the demo was successful, rendering and re-rendering a hashtag cloud based on the Twitter streaming API. 

As a side note, I can't recommend [I <3 logs](http://shop.oreilly.com/product/0636920034339.do) by Jay Kreps enough. If you interested in distributed systems and Kafka's design, it is an excellent read. There is a lot of content that is repeated from free papers he has published, and it is very short, but I found it very engaging and useful beyond Kafka, as he discussed CAP theorem, and other important concepts in distributed computing.


#### Accelerating Innovation in a Data-Driven World

- Imad Sousou
- [https://www.openstack.org/videos/boston-2017/accelerating-innovation-in-a-data-driven-world](https://www.openstack.org/videos/boston-2017/accelerating-innovation-in-a-data-driven-world)

Intel's keynote starts with a video. Imad took the stage and thanked the Intel engineers who gave 30+ talks throughout the summit. He then spoke about IoT and sensors, and how this staggering amount of billions of connected devices will generate 1.5GB a day, per person by the year 2020 (I think I may have reached that mark years ago). He also talked about OpenStack as being enterprise ready, and Intel's significant contributions. He encouraged people to participate in the OpenStack working groups (and I will encourage you to do the same), and acknowledged there is more work to be done to further refine OpenStack.

He talked about clear containers, which leverages VTX extensions in the processor, and how it is supported in Docker, Rkt, Kubernetes, and OpenStack. He spoke about Intel's rackscale design, and how it is integrated with OpenStack. He also talked about Optane storage, describing it as "basically crazy fast storage".

He closes with a slide full of Intel's talks and demo titles at the summit, and highlights a few.

#### Fireside Chat with Brian Stevens, CTO Cloud Platforms, Google

- Brian Stevens (Google cloud)
- [https://www.openstack.org/videos/boston-2017/fireside-chat-with-brian-stevens-cto-cloud-platforms-google](https://www.openstack.org/videos/boston-2017/fireside-chat-with-brian-stevens-cto-cloud-platforms-google)

Mark and Brian discussed Brian's involvement in OpenStack, his hand in establishing the OpenStack Foundation, and his migration from RedHat to Google. 

On open source at Google, Brian remarked:

> You're going to see far less whitepapers from Google; you're going to just see code.

Brian emphasized the importance of the community converging on ways to define services, so that interfaces can be consistent, regardless of platform.


#### Kubernetes and CockroachDB

- Alex Polvi
- Spencer Kimball
- [https://www.openstack.org/videos/boston-2017/kubernetes-and-cockroachdb](https://www.openstack.org/videos/boston-2017/kubernetes-and-cockroachdb)

This demo showcased 3 Kubernetes pods running cockroachDB, and cockroachDB's multi-master architecture. Alex emphasized that Kubernetes _can_ run stateful applications, contrary to popular belief. They scaled the cockroachDB nodes up to 5 pods, and then killed a node. All pods also had loads being generated locally, and the demo included an overview of queries per second, replicas per node, and latency during the expansion and pod being killed. The whole demo was pretty smooth and simple.

#### Interop Challenge

- Brad Topol
- [https://www.openstack.org/videos/boston-2017/interop-challenge](https://www.openstack.org/videos/boston-2017/interop-challenge)

Next up was the "Interop Challenge". Fifteen vendors took the stage, running Ansible to deploy Kubernetes on OpenStack with NFV workloads as well as cockroachDB. There were three people running Newton release, while Canonical ran Ocata. Presumably the remainder was Mitaka (or earlier). In this demo, people deployed the configuration described previously, and then joined the cockroachDB nodes to the existing cluster from the previous demo. All the participants that succeeded in joining the cocroachDB cluster got a trophy. Somewhere in there, there was a [Bill O'Reilly "Do it live!"](https://www.youtube.com/watch?v=O_HyZ5aW76c&t=61) joke and Mark emphasized that the trophies actually meant something, because they were not just handed out blindly to everyone for participation.

#### Deutsche Telekom Sponsor Keynote: Make OpenStack Successful in Public Clouds

- Clemens Hardewig
- [https://www.openstack.org/videos/boston-2017/deutsche-telekom-sponsor-keynote-make-openstack-successful-in-public-clouds](https://www.openstack.org/videos/boston-2017/deutsche-telekom-sponsor-keynote-make-openstack-successful-in-public-clouds)

My notes for this start with "blah blah blah, 'Hybrid Cloud'". This apparently started off a little buzzword heavy for my taste. Things improved and Clemens spoke of the importance of pushing for standardization, citing integration and interoperability as key reasons. He also advocated staying close to upstream, to avoid fragmentation of the code base. Overall, he made some very valid points, but he used a lot of jargon to get there.

#### Mark Collier Q&A with Edward Snowden

- [Mark Collier](https://www.linkedin.com/in/markcollier/)
- Edward Snowden
- [https://www.openstack.org/videos/boston-2017/mark-collier-q-and-a-with-edward-snowden](https://www.openstack.org/videos/boston-2017/mark-collier-q-and-a-with-edward-snowden)

This was, for me, a highlight of the keynotes. I will paraphrase roughly. I took a lot of notes, but I could not write fast enough to be 100% verbatim.

![Snowden](/content/images/2017/05/Photo-May-09--10-33-22-AM.jpg)

Mark started off by asking Snowden's take on cloud computing. 

> Users see it as apps. Then there's you guys. The IaaS layer. For most people, the internet is magic. We can't let people do this mindlessly, especially when you're building the foundations. When you pay for EC2 or Google Cloud, you're giving them more than your money, you're giving them your data and investing in a thing you don't own and can't influence. You give them control, which is kindof a silent vulnerability.

Mark then asked what is in Snowden's open source toolbox.

> Of course there is the stuff I used at the NSA, which is running mostly windows, as many of you know thanks to the recent ShadowBrokers leaks. A lot of the things I ran for anonymity, like ToR guards are open source and ran on Debian.

He then talked about the [Freedom of the Press Foundation](https://freedom.press/), and how they are expanding their use of opensource. He cited the example of [securedrop](https://securedrop.org/), which is a tool for securely giving data to journalists. he then circled back to privacy, asking the question:

> If you run on someone else's infrastructure, how do you know that your data is not being stolen?

He spoke more about open hardware, and how if you put your phone in airplane mode, you don't really know the antenna is off, since it is all happening in software, and hidden from you. He stressed the importance of open hardware designs, like phones that allow you to see where the electrons are actually flowing, so that you can really know your antennas are actually off, and really know what your hardware is actually doing. He talked about the double edged sword of privacy and intelligence, which lead to this quote:

> It's hard to say it, but there are good people at the FBI

He continued

> We should be working for technology itself, not for an organization or an employer. When you think of your ethical obligation, you should think "How do I empower and protect the user?"

There's a [Tron joke in there somewhere](https://www.youtube.com/watch?v=DkTb7Pe2MtY&t=2m17s)

Mark asked how the exploit market impacts people securing the infrastructure.

> Mitigations work. Code securely, and follow best practices. Open source helps, but is not a panacea.

Snowden used shellshock as an example of open source exploits, and said that while open source doesn't guarantee a lack of exploits, the whole community responds, and that response is both empowering and educational, and you don't get that in a closed source patch.

> We want a better world and we're here to build it.

Snowden then compares today's computer science with the nuclear physics of the past century, and talks about bad actors and how governments encouraging use of SS7 was irresponsible. This was an example of how technology needs to be built with foresight, so that anything we build would be less dangerous if it falls into the hands of bad actors.

> You can't unsplit the atom; you can't put the genie back in the bottle. We need to build not for today, but for tomorrow.

There was one more question, but it was very political, and that stuff doesn't interest me.

