+++
author = "Chris Suttles"
categories = ["OpenStack", "Storage"]
date = 2017-04-23T00:46:43Z
description = ""
draft = false
cover = "/images/2017/09/Cinder_netapp_Setup-300x255.png"
slug = "netapp-over-fc-and-cinder-on-openstack-ocata"
tags = ["OpenStack", "Storage"]
title = "NetApp over FC and Cinder on OpenStack Ocata"

+++


In Mitaka release, I worked with our storage team and deployed a NetApp Cinder backend, using Fibre Channel connectivity from the nodes running `cinder-volume`.

When I upgraded to Newton, we started seeing errors in the `cinder-volume` logs that seemed to match this bug:

[NetApp: Failed to get info for aggregate](https://bugs.launchpad.net/cinder/+bug/1660870)

Unfortunately, at that time, there was not a fix available, we lost my POC for this project on the storage team, and the leaders in the org wanted to explore going a different way, so we dropped the cinder backend, instead of submitting a patch or finding a resolution.

Now, we have upgraded to Ocata, and we don't have a replacement storage backend to replace NetApp ready. We are still testing ScaleIO, but initial tests went poorly. While it may be a viable option, we need something better than LVM, so we decided to explore NetApp again.

Fortunately, there was not too much to do to get things working in Ocata when we finally chose to investigate this again. We double checked the zoning and FC related bits, and then looked at the account. NetApp recommends creating a role for the OpenStack user, but when we created a role equivalent to their suggestions, we ran into problems (in Mitaka, Newton, and Ocata), so we made the OpenStack user a member of the vserver admin role for the SVM that was configured to serve OpenStack. We also only use HTTPS and need to verify certificates, so we had to set a few extra options in `cinder.conf`.

The end result was that we were able to get things working again by adding a couple of optional configuration options, and replacing the following `cinder-volume` files that came in our package install (from Ubuntu upstream repos package 2:10.0.0-0) with the ones from [this commit](https://git.openstack.org/cgit/openstack/cinder/commit/?id=109177a366b2440f31d5994df0318bdc0124a88b):

* `	cinder/volume/drivers/netapp/dataontap/client/client_cmode.py`
* `	cinder/volume/drivers/netapp/dataontap/block_cmode.py`

After patching those files on nodes running `cinder-volume`, and restarting the service, we finally saw `cinder service-list` report the NetApp backend as "up".

I created a volume type mapped to the backend with something like:

`openstack volume type create --property volume_backend_name=netapp netapp`

Once we were able to verify volume CRUD operations and attach/detach of volumes from VMs, I made the volume type the default with `default_volume_type=netapp` in `nova.conf` (on the nova-api host).

We used the documentation from NetApp for reference, and choosing configuration options. It's [available on GitHub](https://netapp.github.io/openstack-deploy-ops-guide/ocata/content/ch_executive-summary.html).

