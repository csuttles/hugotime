+++
author = "Chris Suttles"
categories = ["Getting Started", "OpenStack"]
date = 2017-02-27T02:35:16Z
description = ""
draft = false
cover = "/images/2017/09/OpenStack_logo.png"
slug = "getting-started-with-openstack"
tags = ["Getting Started", "OpenStack"]
title = "Getting started with OpenStack"

+++


<a href="http://openstack.org/">OpenStack</a> is exciting!

I started using OpenStack at work; in fact, my desire to work on it and learn more about it was a major factor in accepting my current position. The learning curve is steep, but don't let that intimidate you. The rewards for persistence are great.
<h6>Crawl, Walk, Run</h6>
OpenStack is a really large project, with a lot of moving parts. My approach, after some trial and error, and the approach I advise to people new to OpenStack is to crawl, walk, and run. For more details on this approach, check out this talk from the 2016 OpenStack Summit in Barcelona:

<a href="https://www.youtube.com/watch?v=fFAoX2V6gUU">From How-To to POC to Production- Learning By Building</a>

The basic premise is this. Start small. Build from <a href="https://docs.openstack.org/developer/devstack/">Devstack</a> and play around where there are no consequences. Once you start getting more comfortable, and do some reading, try building an 'all in one' (AIO) install manually. There are lots of great tools to help you deploy OpenStack, but you will learn more through a manual install the first time (maybe the first few times). Focus on core components; it's easy to get overwhelmed with all the parts of OpenStack. Install just enough to deploy a VM with <em>simple</em> networking.

Once you have AIO figured out, start playing around with deployment tools and find something you like. Once you gain a better understanding of OpenStack, a manual install is not a good use of your time. When you find a deployment tool you like, start customizing your AIO environment.

Now you are ready to start working on a more production like architecture. The following guides will make more sense and be helpful at this point:

<a href="https://docs.openstack.org/arch-design/">Architecture Design Guide</a>

<a href="https://docs.openstack.org/ha-guide/">HA Guide</a>

<a href="https://docs.openstack.org/admin-guide/">Admin Guide</a>

Where you go from here will largely depend on your environment and requirements. The possibilities are nearly endless. I have really enjoyed learning about OpenStack and I've found the community very helpful. I hope you will too!

