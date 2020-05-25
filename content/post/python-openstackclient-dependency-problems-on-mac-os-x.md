+++
author = "Chris Suttles"
categories = ["OpenStack", "Python"]
date = 2017-08-18T11:27:33Z
description = ""
draft = false
cover = "https://imgs.xkcd.com/comics/dependencies.png"
slug = "python-openstackclient-dependency-problems-on-mac-os-x"
tags = ["OpenStack", "Python"]
title = "Python-OpenStackclient Dependency Problems on MacOS"
aliases = ["/python-openstackclient-dependency-problems-on-mac-os-x/"]

+++


##### Pip won't do it

I recently [installed OpenStack using Kolla into a single node](http://blog.highspeedlogic.org/building-openstack-containers-with-kolla/) at home for testing the latest beta of the Pike release. When I decided to install `python-openstackclient` on my Mac, I ran into some problems.

I ran `sudo pip install python-openstackclient` and things looked like they were working normally, until a wild traceback appears:

{{< highlight markdown >}}
Installing collected packages: pbr, monotonic, iso8601, netifaces, netaddr, pyparsing, Babel, oslo.i18n, wrapt, debtcollector, oslo.utils, PrettyTable, unicodecsv, pyperclip, cmd2, PyYAML, stevedore, cliff, msgpack-python, oslo.serialization, simplejson, positional, keystoneauth1, python-novaclient, python-cinderclient, ipaddress, asn1crypto, enum34, pycparser, cffi, cryptography, pyOpenSSL, functools32, jsonschema, jsonpointer, jsonpatch, warlock, python-glanceclient, rfc3986, oslo.config, python-keystoneclient, requestsexceptions, appdirs, os-client-config, osc-lib, deprecation, openstacksdk, python-openstackclient
  Running setup.py install for netifaces ... done
  Found existing installation: pyparsing 2.0.1
    DEPRECATION: Uninstalling a distutils installed project (pyparsing) has been deprecated and will be removed in a future version. This is due to the fact that uninstalling a distutils project will only partially uninstall the project.
    Uninstalling pyparsing-2.0.1:
Exception:
Traceback (most recent call last):
  File "/Library/Python/2.7/site-packages/pip/basecommand.py", line 215, in main
    status = self.run(options, args)
  File "/Library/Python/2.7/site-packages/pip/commands/install.py", line 342, in run
    prefix=options.prefix_path,
  File "/Library/Python/2.7/site-packages/pip/req/req_set.py", line 778, in install
    requirement.uninstall(auto_confirm=True)
  File "/Library/Python/2.7/site-packages/pip/req/req_install.py", line 754, in uninstall
    paths_to_remove.remove(auto_confirm)
  File "/Library/Python/2.7/site-packages/pip/req/req_uninstall.py", line 115, in remove
    renames(path, new_path)
  File "/Library/Python/2.7/site-packages/pip/utils/__init__.py", line 267, in renames
    shutil.move(old, new)
  File "/System/Library/Frameworks/Python.framework/Versions/2.7/lib/python2.7/shutil.py", line 302, in move
    copy2(src, real_dst)
  File "/System/Library/Frameworks/Python.framework/Versions/2.7/lib/python2.7/shutil.py", line 131, in copy2
    copystat(src, dst)
  File "/System/Library/Frameworks/Python.framework/Versions/2.7/lib/python2.7/shutil.py", line 103, in copystat
    os.chflags(dst, st.st_flags)
OSError: [Errno 1] Operation not permitted: '/tmp/pip-wMRwXf-uninstall/System/Library/Frameworks/Python.framework/Versions/2.7/Extras/lib/python/pyparsing-2.0.1-py2.7.egg-info'
{{< / highlight >}}

Honestly, I decided to live without `python-openstackclient` on my Mac, and chose to just manage OpenStack from the Linux node where it was running. The issue still irked me, but there are only so many hours in the day, and it's important to prioritize and choose your battles.

Fast forward about three weeks, and I found myself getting a similar traceback when trying to install `boto3` for some AWS action. Now I had to get to the bottom of this.

I found [this pip bug](https://github.com/pypa/pip/issues/3165) with some very helpful comments. The most important part is this sequence of comments:

* Problem described in detail:

> This is because OS X El Capitan ships with six 1.4.1 installed already and when it attempts to uninstall it (because awscli depends on botocore, botocore depends on python-dateutil, and python-dateutil depends on six >= 1.5) it doesn't have permission to do so because System Integrity Protection doesn't allow even root to modify those directories. 

> Ideally, pip should just skip uninstalling those items since they aren't installed to site-packages they are installed to a special Apple directory. However, even if pip skips uninstalling those items and installs six into site-packages we'll hit another bug where Apple puts their pre-installed stuff earlier in the sys.path than site-packages. I've talked to Apple about this and I'm not sure if they're going to do anything about it or not.

* Follow up question:

> This affects many other packages which rely on Six, pardon my obliviousness, but is there any way to tell pip to either 1. install an updated version to /Library's site-packages and be pointed to prefer that one (instead of saying 'requirement satisfied! and exiting) or 2. ignore the version found in /System on 10.11+? 

* Solution:

> `pip install --ignore-installed six`

All of this may be very obvious to someone more familiar with MacOS or pip, but this combination of circumstances was new to me. My recent experiences with MacOS have been steadily more and more disappointing, and pip has always 'just worked' for me. [I have been through dependency hell](https://xkcd.com/754/) with rpm, up2date, and yum, as well as other Linux package managers, but this was a first for me on MacOS. I am on the latest (as of this writing) release, 10.12.6, and to Apple's credit, this is the first problem of this type I have had after many years of MacOS use. To Apple's discredit, I have had more issues with MacOS in the last year than I had in all the years prior.

##### Pip will do it

I followed the suggestion after skimming the thread and a couple others, and was able to install both packages that had failed for me in the past by simply appending `--ignore-installed six`:

* `pip install boto3 --ignore-installed six`
* `pip install python-openstackclient --ignore-installed six`

##### Victory

Finally!

{{< highlight markdown >}}
openstack token issue
+------------+----------------------------------+
| Field      | Value                            |
+------------+----------------------------------+
| expires    | 2017-08-18T05:24:10+0000         |
| id         | 258e80ee702a4351a6a5388470b5c8b5 |
| project_id | 2058de16cfc9425f91394b52d9af26d7 |
| user_id    | 65aee56966b44dc29ad57ac9050a14bf |
+------------+----------------------------------+
{{< / highlight >}}

