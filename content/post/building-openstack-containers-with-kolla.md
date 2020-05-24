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

{{< highlight markdown >}}

horizon_port: "8080"
database_port: 3307
enable_haproxy: "no"

{{< / highlight >}}
