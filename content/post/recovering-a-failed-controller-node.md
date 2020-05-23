+++
author = "Chris Suttles"
categories = ["OpenStack"]
date = 2017-08-10T15:30:30Z
description = ""
draft = false
cover = "/images/2017/08/operation-3.jpg"
slug = "recovering-a-failed-controller-node"
tags = ["OpenStack"]
title = "Recovering a Failed Controller Node"

+++


###### What happened?

In an OpenStack instance with a single controller, using Galera and RabbitMQ for clustered database and message queue, the controller failed. I still had the message queue and database on the other nodes, but thanks to a looming timeline, pressure from management, and my own mistakes, I ended up with a SD card backing some key logical volumes. The SD card died, and thus, I am rebuilding the controller from nothing.

It's important to note the the first step was to fix the automation that built things touching the SD card. This work was completed by some of my colleagues about a month ago (thanks guys!). I was under pressure to get something done in a hurry, and instead of pushing back, I made the wrong choice and succumbed to the pressure. Now, I must pay. This activity is the cost of tech debt and rushing to a deadline instead of doing it the right way the first time and letting the timeline slip.

###### Steps to Recovery: Mise en place

First, we need to reinstall the OS with PV/VG/LV configured correctly. Since the deployer profile for this host was fixed after it was built, this part was easy. Next, I ran our Ansible automation for installing OpenStack. Then, I stopped all the OpenStack services, including RabbitMQ and MySQL. 

###### Steps to Recovery: Galera

I configured Galera on the host [as described in a previous post](http://blog.highspeedlogic.org/clustering-galera-on-single-stack-ipv6/), and started the service, but recovery did not start. I checked the grastate.dat files on the other hosts in the cluster, and while they were both in sync and at the same change ID/UUID, both had `safe_to_bootstrap: 0`. I set `safe_to_bootstrap: 1` in the grastate.dat file on one of the hosts; normally you'd pick the one with the highest change number, but I could do either node since they were the same. Next, I started recovery again on the host being rebuilt. The newly rebuilt host joined the cluster, which I was able to monitor locally in logs by setting `wsrep_debug=ON` in the hosts config, and remotely on the boostrap and other cluster member with standard `show state like 'wsrep%';` commands in mysql client.

example grastate.dat:

```
# GALERA saved state
version: 2.1
uuid:    08e5cdf5-32b1-11e7-8a55-33a0042f0e96
seqno:   -1
safe_to_bootstrap: 0
```

Just to make myself feel a little less nervous, I ran some simple sanity checks on all the Galera nodes and made sure I got the same result:

```
mysql nova -e 'select count(*) from instances;'
```

What a relief!

###### Steps to Recovery: RabbitMQ

Again, I will refer to [a previous post on configuration for reference](http://blog.highspeedlogic.org/clustering-rabbitmq-on-ipv6-only-with-openstack-ocata/). Fortunately, I did the right things and folded the changes mentioned there into our Ansible automation for RabbitMQ, appending  this to `/etc/rabbit/rabbitmq-env.conf`:

```
SERVER_ERL_ARGS="-proto_dist inet6_tcp +P 1048576"
CTL_ERL_ARGS=${SERVER_ERL_ARGS}
```

That sets up Erlang under RabbitBQ correctly to cluster on IPv6.

Since the node that failed was the master of the cluster, and anything lingering in the queue is not worthwhile, I simply stopped rabbitmq everywhere, blew away the mnesia dir and rebuilt the cluster from scratch.

```
systemctl stop rabbitmq-server
rm -rf /var/lib/rabbitmq/mnesia/*
systemctl start rabbitmq-server
```

Then I copied the erlang cookie to the repaired node, and ran through the cluster join procedure in [the OpenStack HA Guide](https://docs.openstack.org/ha-guide/shared-messaging.html#rabbitmq-configure).

Because I destroyed the queues and config, I also needed to set up permissions in RabbitMQ again once the cluster configuration was complete, [as described in OpenStack install guide](https://docs.openstack.org/ocata/install-guide-ubuntu/environment-messaging.html).

###### Steps to Recovery: OpenStack

Next, I needed to run through the configs and make sure that when I start OpenStack services again, I don't explode the world. For the most part, this was a noop, but it's better to check than to wish I checked after the damage is done. The one difference I found was in recently merged changes that referred to a separate instance of RabbitMQ for notifications. This was present on the new node, but not on the old nodes. The other difference was in the specification of transport_url, which referred only to a single host instead of the whole RabbitMQ cluster. Once I checked and fixed configs, I ran through restarting services in roughly this order:

```
glance-api
glance-registry
nova-api
nova-conductor
nova-consoleauth
nova-spiceproxy
nova-scheduler
neutron-server
neutron-linuxbridge-agent
neutron-dhcp-agent
neutron-metadata-agent
cinder-scheduler
ceilometer-agent-notification
ceilometer-collector
heat-engine
heat-api
heat-api-cfn
apache2
haproxy
```
I logged in to horizon and things were looking good.

###### Verification is the hard part

Logging in to horizon and giving a thumbs up is a pretty lazy verification; here's my list of things to follow up and sanity check the recovery.

* Launch an instance from image in each AZ (this covers a lot of Nova, Neutron, Glance, and mirrors a basic use case)
* Create, Attach, Detach, Remove Volumes (basic vol CRUD)
* Glance CRUD
* Launch a Stack
* Check Ceilometer metrics are working
* Look at queues in RabbitMQ, verify no stuck messages
* Check Galera, are nodes still in sync after previous actions?

Sounds great. Step 1, launching an instance was a failure. I ended up with errors creating volumes. I followed cinder-scheduler.log and found that no backends were available:

```
INFO cinder.scheduler.base_filter ... Filtering removed all hosts for the request with volume ID 'xxx'. Filter results: AvailabilityZoneFilter: (start: 24, end: 0), CapacityFilter: (start: 0, end: 0), CapabilitiesFilter: (start: 0, end: 0)
```
...
```
cinder.scheduler.flows.create_volume.ScheduleCreateVolumeTask;volume:create: No valid backend was found. No weighed backends available
```

I checked `openstack volume service list` and found duplicate backends for every storage node and backend. One up, one down. I generated a list of the bad ones with `openstack volume service list -f value -c Binary -c Host -c State | awk '/down/ {print $1 " " $2}'`, and then used `cinder-manage service remove BINARY HOST`, [as suggested in this post on OpenStack ask](https://ask.openstack.org/en/question/35046/how-to-delete-a-cinder-service/).

I tried to deploy an instance again, but no joy.

So let's dial it back and just try creating some volumes. I was able to reproduce the same error. I narrowed it down by specifying the AZ I wanted, and I picked one that is sparse. Next, I bounced nova, cinder, and neutron services on the compute/storage nodes in that AZ. I was finally able to create a volume.

The lesson here, is that I probably would have saved myself some grief by restarting nova, cinder, and neutron agents on the compute/storage nodes after resurrecting my dead controller.

I tried creating a VM again, and got a similar error. I scratched my head a bit on this one. I started walking through the process, and specifically what I was asking Cinder to do. I wanted to create a volume from an image. I double checked configs for Glance, and found that there's an autogenerated uuid to specify the swift container to use as a backend. I was able to dig the previous container out of the glance database with a quick query: `MariaDB [glance]> select * from image_locations LIMIT 5;`

Once I corrected the swift config, I thought that might resolve the issue, but I was skeptical, since it seems possible that the image_locations table is used on each query, not just the container in the config; this would make sense if there are multiple glance backends in play. I tried again, and again, was unsuccessful.

I took another step back. I had been attempting to create VMs in horizon this whole time. I tried first creating volumes in each AZ via the cli, which succeeded. Next, I tried creating VMs via the cli, which also succeeded. So that's great, but why? When you create a VM from an image in the cli, it doesn't create a persistent volume on the default backend (at least with our config), but creates a lvm ephemeral volume instead.

So I can create volumes on the default backend (NetApp over FC) from the cli, and I can create VMs using LVM storage via cli, but I still could not create a VM using the default backend, and even worse, I got different errors depending on placement. In one AZ, I got a 'not enough hosts available' error, and in the others, I got 'Block device mapping invalid' errors. I found stuck cinder-scheduler-fanout queues in rabbitmq, so I restarted every node in the rabbitmq cluster and restarted nova and cinder scheduler services on the controller. I also restarted nova, cinder, and neutron services on a compute node I was watching during testing. This was somewhat successful, and cleared the queues, also getting rid of my 'not enough hosts available' errors, but I was still unable to provision VMs from horizon.

I spent a little time looking for differences in the things I was doing. When I created a volume in the cli, I was specifying the type. I checked the cinder config, and found I had missed the `default_volume_type=xxx` option when scrubbing configs. I added the value, scrubbed configs again just in case, and restarted services on the controller.

Finally, my test instances in each AZ were able to launch successfully from both the cli and horizon.

###### Verification, Continued

Now, I finally begin working through the remainder of my verification list.

Volume CRUD works fine in all availability zones. We use `cross_az_attach=false` in `/etc/nova/nova.conf` to prevent block traffic traversing between AZs, so it was important to validate Nova/Cinder within each.

The basic flow for testing this was:

```
openstack volume create --type $type --size 100 --availability-zone $az $volname
openstack server add volume $server_id $vol_id
openstack server remove volume $server_id $vol_id
openstack volume delete $vol_id
```
During the course of testing this, it was late, and I was getting tired, so I made a mistake. I accidentally validated the `cross_az_attach=false` parameter, and got this helpful message:

```
openstack server add volume 868fbde4-FFFF-FFFF-baae-b132ebe844ed 086f3021-4ea8-FFFF-FFFF-6aa27ef4a069

Invalid volume: Instance 186 and volume 086f3021-4ea8-FFFF-FFFF-6aa27ef4a069 are not in the same availability_zone. Instance is in AZ1. Volume is in AZ3 (HTTP 400) (Request-ID: req-736732d7-FFFF-FFFF-839c-92b23cbca6a8)
```

I double checked RabbitMQ and Galera, and all appears well in both:

```
# rabbitmqctl list_queues | grep -v 0$
Listing queues ...
```

No stuck messages in queues.

```
rabbitmqctl cluster_status
```

All nodes report the same cluster_status / membership.

```
MariaDB [(none)]> show status like 'wsrep%';
```
This all looks good, according to [the official Galera docs on monitoring the cluster](http://galeracluster.com/documentation-webpages/monitoringthecluster.html).

The last thing to do was add a stack. I fired off a simple single server test stack to validate heat was working. I checked the output of `openstack server list`, `openstack stack list` and `openstack stack show $mystackid`.

Lastly, I installed and checked our custom daemon to sync with IPAM, and was able to validate that test nodes I created were getting pushed to IPAM and therefore DNS.

It is with great joy that I can finally publish this post. I wrote this as I worked through the recovery. When it got a little rough I was glad to have notes. Now, I am glad to be done.

