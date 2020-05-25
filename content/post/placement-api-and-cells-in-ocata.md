+++
author = "Chris Suttles"
categories = ["OpenStack"]
date = 2017-03-11T02:05:42Z
description = ""
draft = false
cover = "/images/2017/09/openstack-ocata-cells.jpg"
slug = "placement-api-and-cells-in-ocata"
tags = ["OpenStack"]
title = "Placement API and Cells in Ocata"
aliases = ["/placement-api-and-cells-in-ocata/"]

+++


## What's up, Doc?

In Ocata release, the placement API and cells (v2) are mandatory. This is not currently documented in the official installation documentation:

https://docs.openstack.org/ocata/install-guide-ubuntu/nova-controller-install.html

*edit: There's now a draft of docs, based on the work on the bug mentioned later in this post here: http://docs-draft.openstack.org/28/438328/12/check/gate-openstack-manuals-tox-doc-publish-checkbuild/846ac33//publish-docs/draft/install-guide-ubuntu/nova-controller-install.html*

At the time of writing this post, there is no mention of placement API of cells v2, which are both mandatory in Ocata release.

Here's a very helpful post, that I used as a starting point:

https://ask.openstack.org/en/question/102256/how-to-configure-placement-service-for-compute-node-on-ocata/

I also found to following references helpful: 

https://docs.openstack.org/developer/nova/placement.html#deployment

https://docs.openstack.org/developer/nova/cells.html#cells-v2

Between these resources, I got really close. I also found it helpful to deploy devstack with placement API. I then connected to mysql and looked at the database configuration, and compared with what I had configured on my larger test installation of the Ocata release.

##### What can you gain from my mistakes? What did I learn?

The OpenStack community is very helpful, and I found the bug where a lot of troubleshooting is happening. It was comforting to see I wasn't the only person that was struggling with this, and that people are actively working to fix it: 

https://bugs.launchpad.net/openstack-manuals/+bug/1663485

While it's not really a new lesson, I found it very useful to do the following:

* Slow down and read the docs and configs very carefully.
* Slow down and read the logs very carefully.
* Verify the things that aren't working that you think should be working; I wasted a lot of time looking for a problem that was a simple typo I overlooked.
* Check those transport_url and database_connections for your cells and make sure they match database GRANT
* Make sure that you do the db sync and cell commands in the right order. If you mix up any of those steps, you are going to regret it (ask me how I know).
* For small environments, it may be helpful to consider using these `[scheduler]` options in nova.conf, even if only temporarily:

  * `driver=caching_scheduler`
> which aggressively caches the system state for better
> individual scheduler performance at the risk of more retries when running
> multiple schedulers 

  * `discover_hosts_in_cells_interval=300` # default is -1, which is disabled
> Small deployments may want this periodic task enabled, as surveying the
> cells for new hosts is likely to be lightweight enough to not cause undue
> burdon to the scheduler. However, larger clouds (and those that are not
> adding hosts regularly) will likely want to disable this automatic
> behavior and instead use the `nova-manage cell_v2 discover_hosts` command
> when hosts have been added to a cell.

* Work through the VM request flow:
  * Does nova-placement get the compute hosts' information? 
  * Does scheduler filter hosts? If so, why are hosts ineligible? 
  * Does the VM actually get scheduled, but fail after scheduling?

