+++
author = "Chris Suttles"
categories = ["OpenStack"]
date = 2017-03-07T13:56:21Z
description = ""
draft = false
cover = "/images/2017/10/COMPUTER-REPAIR.jpg"
slug = "troubleshooting-openstack"
tags = ["OpenStack"]
title = "Troubleshooting OpenStack"

+++


#### Everything begins with Keystone

This is nothing new, but it's important to point out. Nothing works if Keystone is not set up properly. The easiest way to check if Keystone and your client are set up correctly is to run:

{{< highlight markdown >}}

openstack token issue

{{< / highlight >}}

An often overlooked part of Keystone is the service catalog. Keystone is responsible for authentication and authorization in OpenStack, but it's also responsible for the service catalog. If you run the following, you'll see the contents of the service catalog:

{{< highlight markdown >}}

openstack endpoint list

{{< / highlight >}}
