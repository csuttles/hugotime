+++
author = "Chris Suttles"
date = 2018-09-07T08:47:08Z
description = ""
draft = false
cover = "/images/2018/09/homelab.jpg"
slug = "why-i-sold-off-my-home-lab"
title = "Why I sold off my home lab"

+++


It's been a while since my last post. In that time I moved from the suburb I could not afford that was _near_ Silicon Valley back to Texas. The move has been terrific for my entire family. During that move, I worried a lot about all my home lab gear. It traveled with us cross country along with other things that were too important to trust movers to handle.Now, nearly all of that gear has been sold. When we finally arrived in our home, and movers brought all of our stuff, we spent a lot of time unpacking. We spent a lot of time getting rid of things before the move, and once we arrived, that train only picked up steam. We sold or donated a lot of things post move. Parting with my home lab was one of the things that went, and that was months ago now. It was a good decision.

I looked at my mountain of gear. I had quite a lot of it. I had

* (2) 24-port Gigabit Edge Switches with POE fiber uplinks,
* 8-port Edge Switch with fiber uplinks
* 8-port Gigabit Edge Router Pro
* (5) Ubiquiti 802.11ac PRO Access Points
* QNAP for centralized, redundant storage
* (5) powerful desktops, each with high end i7 processors, at least 32GB of RAM, multiple Gigabit NICS, a SSD or NVMe of 256-512GB for boot and also a 1TB or greater spindle for large storage
* (5) 1500VA UPSs
* (2) Raspberry Pis

Clearly, I invested a lot in this home lab. I had dual stack support for everything, a zone based firewall, vlans for segregating traffic, excellent WiFi signal everywhere for all my devices, a local DNS server to speed up my browsing, and managed my traffic with QoS. I could (and did) run any kind of software I wanted or needed to for studying. I was able to spin up OpenStack with the recommended bare metal architecture and use one NIC for management, one for OpenStack, and one for guest/Overlay networking. I ran Kafka, and could spin up ELK on a whim. I had a Kubernetes instance going too.It was around the time I looked at this mountain of gear that I started thinking about where I ran into limitations with Kubernetes. A lot of the cool things you can do with Kubernetes are really better with a cloud where you can make an API call and spin up a load balancer. While you can use a private cloud for that, and I even had one, the problem is someone has to maintain your private cloud, and in this case, that someone was me.

I got a lot out of the lab; it was an investment in myself and my career. It also took a lot of my personal time, and ballooned my power bill to terrific heights.Unless you are a huge company like Facebook, most of the world has moved to the cloud or is going there. That's where my focus is going too. There was a time when the bare minimum to be a warm body with a badge was some core sysadmin skills. That time is gone. The bar has been raised and to be tall enough to ride the ride, you need to know sysadmin skills, be able to write code in at least one language, and be able to use at least one cloud. I've never been shooting for the bare minimum. I'm aiming for excellence everywhere. Building my skills in multiple clouds and multiple languages is my next step, and getting rid of my home lab means all my study here _has_ to be done in a cloud or on a laptop.I got rid of my home lab, also as an investment in my future.

