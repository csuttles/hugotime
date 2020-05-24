+++
author = "Chris Suttles"
categories = ["OpenStack", "automation"]
date = 2017-09-06T13:50:23Z
description = ""
draft = false
cover = "/images/2017/09/openstack-rally.jpg"
slug = "getting-acquainted-with-openstack-rally"
tags = ["OpenStack", "automation"]
title = "Getting Acquainted with OpenStack Rally"

+++


## Automate Everything

It's important to automate all the things you can to make your workflow efficient and error free. It's easy to miss a step in a runbook or let a typo ruin your testing.

## Enter Rally

I think best way to test changes is via CI/CD, and [Rally](https://wiki.openstack.org/wiki/Rally) is the tool that's used for gating changes to OpenStack in [OpenStack's Zuul](http://status.openstack.org/zuul/). Even if you are not commiting code to OpenStack, Rally can help you validate changes to your environment. Leveraging Rally, you can automate testing after a release, config change, or package upgrade, and have confidence that you didn't break anything (or at least that you didn't break anything that was tested).

## Getting Rally

Check out the [Rally docs](http://docs.xrally.xyz/projects/openstack/en/latest/), or jump straight into the [quick start guide](http://docs.xrally.xyz/projects/openstack/en/latest/quick_start/tutorial.html).

## Using Rally

This is all covered well in the quick start guide, so I am going to be brief. You start by adding a deployment, and then you can test just about anything you like. The task  definitions are very felxible and customizable, and the sample configurations are very helpful as a starting point for both understanding the framework, and customizing it to your needs. A basic test might be something like this:

##### Install
```
curl https://raw.githubusercontent.com/openstack/rally/master/install_rally.sh | bash
```

_disclaimer: I always break this into steps. Download, read, then execute. This is a copypasta from the docs._

##### Configure

```
source openrc
rally deployment create --fromenv --name=braaap
...
+--------------------------------------+---------------------+--------+------------------+--------+
| uuid                                 | created_at          | name   | status           | active |
+--------------------------------------+---------------------+--------+------------------+--------+
| 1f102242-37db-419e-accf-db485cf3be25 | 2017-09-06T06:39:30 | braaap | deploy->finished |        |
+--------------------------------------+---------------------+--------+------------------+--------+
```

##### Sanity Check

```
rally deployment check
--------------------------------------------------------------------------------
Platform openstack:
--------------------------------------------------------------------------------

Available services:
+-------------+----------------+-----------+
| Service     | Service Type   | Status    |
+-------------+----------------+-----------+
| __unknown__ | compute_legacy | Available |
| cloud       | cloudformation | Available |
| glance      | image          | Available |
| heat        | orchestration  | Available |
| keystone    | identity       | Available |
| neutron     | network        | Available |
| nova        | compute        | Available |
+-------------+----------------+-----------+
```

##### Run a Task

Have a look at your image and flavor names. Then edit the sample task to make sure they match:

`vim samples/tasks/scenarios/nova/boot-and-delete.json`

In my case, I had to edit the image name to match my avaiable cirros test image.

Run the Task:

```
rally task start samples/tasks/scenarios/nova/boot-and-delete.json
--------------------------------------------------------------------------------
Preparing input task
--------------------------------------------------------------------------------

Task is:
{
    "NovaServers.boot_and_delete_server": [
        {
            "args": {
                "flavor": {
                    "name": "m1.tiny"
                },
                "image": {
                    "name": "^cirros.*$"
                },
                "force_delete": false
            },
            "runner": {
                "type": "constant",
                "times": 10,
                "concurrency": 2
            },
            "context": {
                "users": {
                    "tenants": 3,
                    "users_per_tenant": 2
                }
            },
            "sla": {
                "failure_rate": {
                    "max": 0
                }
            }
        },
        ...
  ```
        
##### Check the Results!

```
--------------------------------------------------------------------------------
Task ed76acf4-5685-4b7c-864d-6bffe0c9d7bb has 0 error(s)
--------------------------------------------------------------------------------

+-----------------------------------------------------------------------------------------------------------------------+
|                                                 Response Times (sec)                                                  |
+--------------------+-----------+--------------+--------------+--------------+-----------+-----------+---------+-------+
| Action             | Min (sec) | Median (sec) | 90%ile (sec) | 95%ile (sec) | Max (sec) | Avg (sec) | Success | Count |
+--------------------+-----------+--------------+--------------+--------------+-----------+-----------+---------+-------+
| nova.boot_server   | 10.072    | 12.471       | 15.027       | 15.082       | 15.137    | 12.773    | 100.0%  | 10    |
| nova.delete_server | 2.283     | 2.408        | 2.594        | 2.612        | 2.629     | 2.438     | 100.0%  | 10    |
| total              | 11.476    | 13.945       | 16.489       | 16.603       | 16.718    | 14.211    | 100.0%  | 10    |
+--------------------+-----------+--------------+--------------+--------------+-----------+-----------+---------+-------+

Load duration: 76.55527
Full duration: 110.138024

HINTS:
* To plot HTML graphics with this data, run:
        rally task report ed76acf4-5685-4b7c-864d-6bffe0c9d7bb --out output.html

* To generate a JUnit report, run:
        rally task export ed76acf4-5685-4b7c-864d-6bffe0c9d7bb --type junit --to output.xml

* To get raw JSON output of task results, run:
        rally task results ed76acf4-5685-4b7c-864d-6bffe0c9d7bb
```

This is really only scratching the surface of what you can do with Rally, but I think it is easy to see the power of this tool. Enjoy!

