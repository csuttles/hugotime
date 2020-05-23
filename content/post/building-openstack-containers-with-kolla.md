+++
author = "Chris Suttles"
categories = ["OpenStack", "containers"]
date = 2017-08-03T11:04:28Z
description = ""
draft = false
cover = "/images/2017/09/openstack-kolla.jpeg"
slug = "building-openstack-containers-with-kolla"
tags = ["OpenStack", "containers"]
title = "Building OpenStack Containers with Kolla"

+++


###### Docs, Pitfalls, and First Steps

The Kolla OpenStack project is focused on deploying OpenStack in containers. There's a good [quickstart guide](https://docs.openstack.org/kolla-ansible/latest/quickstart.html), but like most things OpenStack, you'll need to dig a little deeper if you want to be successful.

You can deploy an all-in-one, or multinode instance of OpenStack with Kolla; the latter requires a local Docker registry, so even for an all-in-one deployment, I chose to create a local registry, so that I might be better prepared when the time came to move to multinode.

I ran into several problems which were not documented in the official Kolla docs at the time of writing. Some of them are related to the Docker registry and daemon, some were related to Kolla itself.

One of the issues I ran into was with running the registry. By default, Docker wants to talk to a registry over HTTPS and with authentication. That's great, but it created some hurdles for me when testing with Kolla. Running the registry container with selinux enabled meant I needed to add `:Z` to my volume mount command for the registry. The full command I ended up using to run the registry was `docker run -d -p 4000:5000 --restart=always -v /data/registry:/var/lib/registry:Z  --name regstry registry:2`. Next, I needed to add the `--insecure-registry` flag to the docker daemon. Next, I needed to add `--enable-secrets=false` to the docker daemon config due to a race condition with `/run/secrets` that is touched on in [this kolla bug](https://bugs.launchpad.net/kolla/+bug/1668059). I am running `docker-1.12.6-32.git88a4867.el7.centos.x86_64` for the record. There are some additional details on the Kolla `/run/secrets` problem in [this RedHat bug](https://bugzilla.redhat.com/show_bug.cgi?id=1410118). All of this was a trial and error process and needed to be done before I could really start getting traction following the quickstart guide. The `--insecure-registry` option is discussed there, but without the other changes, it was a pretty frustrating experience.

I also had some port conflicts, so I ended up customizing some of the build config. Most of my options are defaults, but some of the ones I needed to add to `/etc/kolla/globals.yml` included:

```
horizon_port: "8080"
database_port: 3307
enable_haproxy: "no"
```

Once those changes were in place, and I got my kolla-build done, I was on my way. I ended up passing a few flags not in the quickstart guide to be more explicit and speed things up: `kolla-build --tag 5.0.0.0b3 --cache -t binary`. [This page](https://releases.openstack.org/teams/kolla.html) was indispensible in mapping the Kolla tags to the actual OpenStack release I wanted to build.

###### Finally, Deploy

Once all the Docker and Kolla changes were in place, I was able to run the all-in-one install pretty easily with just:

* `kolla-ansible prechecks -i /path/to/all-in-one`
* `kolla-ansible deploy -i /path/to/all-in-one`
* `. /etc/kolla/admin-openrc.sh`
* `cd /usr/share/kolla-ansible && ./init-runonce`

I checked `docker ps` and was able to use the openstack cli to create an instance: `openstack server create --image cirros --flavor m1.tiny --key-name mykey --nic net-id=7a6ed25b-2034-46c7-a844-e5451e361c99 demo1` (your net-id will of course be different). Everything was looking great. I ran the following command to open up ports in firewalld:

`for port in $(openstack endpoint list -f value -c URL | perl -pe 's/.*://;s/\D+$/\n/;s/\/v.*$//' | sort -u ) ; do firewall-cmd --add-port=${port}/tcp; done`

I tried to use the dashboard, but it didn't load. I ended up digging around in the configs and found that my port change for horizon in `/etc/kolla/globals.yml` did not take effect. I found the horizon config in `/etc/kolla/horizon/horizon.conf` and could see the default port of `80` being used. I attached to the container and saw the same. I updated the config, and restarted the container, and now I got a login page, but when I logged in I got an error page on redirect to the dashboard. I found [this RedHat bug](https://access.redhat.com/solutions/2038223) for the issue, and made similar changes. The fix was to add this Alias to the vhost config in `/etc/kolla/horizon/horizon.conf` and restart the container:

`Alias /dashboard/static /usr/share/openstack-dashboard/static`

After making that change, I was able to get a proper horizon dashboard and truthfully call my all-in-one deployment a success.

