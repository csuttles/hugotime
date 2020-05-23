+++
author = "Chris Suttles"
categories = ["OpenStack", "Python", "automation", "summit"]
date = 2017-05-10T11:32:37Z
description = ""
draft = false
cover = "/images/2017/09/openstack-summit-boston-6.jpeg"
slug = "openstack-summit-boston-early-bird-at-the-upstream-institute"
tags = ["OpenStack", "Python", "automation", "summit"]
title = "OpenStack Summit Boston - Upstream Institute"

+++


##### Upstream Institute
Sunday, I attended this session, and met some great people. I sat with two Swift core contributors, two Cinder contributors, and a Manilla contributor who also previously contributed to Horizon and Trove. The class was  an intro to contributing to OpenStack, so I was lucky to end up at a table full of seasoned folks. I made connections with almost everyone at the table, and I really enjoyed speaking with them and learning from them.

#### Highlights:

* https://docs.openstack.org/upstream-training/
* Launchpad getting replaced with storyboard, which is more of a Kanban feel
* grtty is a useful tool for doing gerrit via ncurses (mouse free!)
* Vim users: Vgq will format your git commits nicely!
* I saw a demo of Swift running in Kubernetes (one of the guys at my table)

We talked about all the stuff in the docs I linked, developer setup/workflow, and some additional detail on:
* Zuul/Gataekeeper
* How CI is configured, and how many nodes back it (800-1200)
* How and when to recheck a diff
* Elastic rechecks (automatic rechecks that occur to workaround known bugs or CI system failures unrelated to code)
* Stacking changes from other repos with `depends-on: <commit-id>`
* http://stackalytics.com - Using it for metrics and to find core contributors so you can contact them for help landing diffs
* IRC/mailing lists, and getting help

