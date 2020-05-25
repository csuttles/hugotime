+++
author = "Chris Suttles"
categories = ["azure", "Python", "cloud", "batch", "exif", "gps", "automation"]
date = 2020-05-23T00:59:21Z
description = ""
draft = false
cover = "/images/2020/05/overview-of-the-azure-batch-workflow.png"
slug = "extending-the-azure-batch-python-quickstart"
tags = ["azure", "Python", "cloud", "batch", "exif", "gps", "automation"]
title = "Extending the Azure Batch Python Quickstart"
aliases = ["/extending-the-azure-batch-python-quickstart/"]

+++


### ****What is this?****

This is a fun little project I did to try to take some a simple Azure Quickstart post and enhance it to do something a little more (?:interesting|useful|realistic).

### ****Challenge yourself****

Can you accomplish the end goal of this lab without looking at the solution? If you looked at the solution and ran it, can you change it to make it do something else, like transcode videos or one of the other common uses of batch?  What about removing the GPS Position from the photos to protect the privacy of the user?

### ****What does it do?****

Upload all jpgs in a dir, extract GPS position from those photos, and turn them into a Google maps link. If you want to try to do this without any spoilers, please stop reading now, [head to the MSFT article, and fork the original repo](https://docs.microsoft.com/azure/batch/quick-run-python) to start making changes!

### ****How does it do that?****

* adds a little randomization to allow for faster prototyping (don't reuse the container, pool, or batch names since cleanup takes a long time)
* adds minor batch node customization (install exiftool)
* adds minor image processing (parse GPS position from images and turn into Google maps link)

### **Where can I get the code and all the details?**

Here's external and internal links to the repo and readme with details:

[https://github.com/csuttles/batch-python-quickstart](https://github.com/csuttles/batch-python-quickstart)

You can find the Azure quickstart this is based on here: [https://docs.microsoft.com/azure/batch/quick-run-python](https://docs.microsoft.com/azure/batch/quick-run-python)

## Show me the money

Here's the end result of this little example, where we essentially wrote two lines: [one line to customize the worker](https://github.com/csuttles/batch-python-quickstart/blob/master/src/python_quickstart_client.py#L167) and [one line to execute the jobs](https://github.com/csuttles/batch-python-quickstart/blob/master/src/python_quickstart_client.py#L219).

Two lines of input can do a lot! What else can you do with two lines?

{{< highlight markdown >}}
csuttles@jabroni:[~/src/csuttles-batch-python-quickstart/src]:(master *%)
[Exit: 0] 11:05: python python_quickstart_client.py
Sample start: 2020-05-22 11:05:23

Uploading file ../img/england-london-bridge.jpg to container [input1]...
Uploading file ../img/turkey-bodrum.jpg to container [input1]...
Uploading file ../img/england-brighton-pier.jpg to container [input1]...
Uploading file ../img/germany-allgaeu-fliegenpilz.jpg to container [input1]...
Uploading file ../img/irland-dingle.jpg to container [input1]...
Uploading file ../img/germany-english-garden.jpg to container [input1]...
Uploading file ../img/germany-garching-heide.jpg to container [input1]...
Uploading file ../img/taiwan-jiufen.jpg to container [input1]...
Uploading file ../img/italy-garda-lake-sailing-club.jpg to container [input1]...
Uploading file ../img/japan-katsura-river.jpg to container [input1]...
Creating pool [PythonQuickstartPool10]...
Creating job [PythonQuickstartJob4]...
Adding 10 tasks to job [PythonQuickstartJob4]...
command: /bin/bash -c "echo \"https://www.google.com/maps/search/?api=1&query=$(exiftool -gpsposition -s3 -n england-london-bridge.jpg| sed -e 's/ /+/')\" && echo england-london-bridge.jpg"
command: /bin/bash -c "echo \"https://www.google.com/maps/search/?api=1&query=$(exiftool -gpsposition -s3 -n turkey-bodrum.jpg| sed -e 's/ /+/')\" && echo turkey-bodrum.jpg"
command: /bin/bash -c "echo \"https://www.google.com/maps/search/?api=1&query=$(exiftool -gpsposition -s3 -n england-brighton-pier.jpg| sed -e 's/ /+/')\" && echo england-brighton-pier.jpg"
command: /bin/bash -c "echo \"https://www.google.com/maps/search/?api=1&query=$(exiftool -gpsposition -s3 -n germany-allgaeu-fliegenpilz.jpg| sed -e 's/ /+/')\" && echo germany-allgaeu-fliegenpilz.jpg"
command: /bin/bash -c "echo \"https://www.google.com/maps/search/?api=1&query=$(exiftool -gpsposition -s3 -n irland-dingle.jpg| sed -e 's/ /+/')\" && echo irland-dingle.jpg"
command: /bin/bash -c "echo \"https://www.google.com/maps/search/?api=1&query=$(exiftool -gpsposition -s3 -n germany-english-garden.jpg| sed -e 's/ /+/')\" && echo germany-english-garden.jpg"
command: /bin/bash -c "echo \"https://www.google.com/maps/search/?api=1&query=$(exiftool -gpsposition -s3 -n germany-garching-heide.jpg| sed -e 's/ /+/')\" && echo germany-garching-heide.jpg"
command: /bin/bash -c "echo \"https://www.google.com/maps/search/?api=1&query=$(exiftool -gpsposition -s3 -n taiwan-jiufen.jpg| sed -e 's/ /+/')\" && echo taiwan-jiufen.jpg"
command: /bin/bash -c "echo \"https://www.google.com/maps/search/?api=1&query=$(exiftool -gpsposition -s3 -n italy-garda-lake-sailing-club.jpg| sed -e 's/ /+/')\" && echo italy-garda-lake-sailing-club.jpg"
command: /bin/bash -c "echo \"https://www.google.com/maps/search/?api=1&query=$(exiftool -gpsposition -s3 -n japan-katsura-river.jpg| sed -e 's/ /+/')\" && echo japan-katsura-river.jpg"
Monitoring all tasks for 'Completed' state, timeout in 0:30:00..................................................................................................................................................................................................
  Success! All tasks reached the 'Completed' state within the specified timeout period.
Printing task output...
Task: Task0
Node: tvmps_b4adedcce39813ba58c10d6fedc0f04a4b7561354452b15a1776d415cc5eaa39_d
Standard output:
https://www.google.com/maps/search/?api=1&query=51.5041055555556+-0.074575
england-london-bridge.jpg

Task: Task1
Node: tvmps_bb6dfc1d2c40d1ed4211ed84a427acc8e5c6190941e1157f2ab065467d768d75_d
Standard output:
https://www.google.com/maps/search/?api=1&query=37.0299527777778+27.4132638888889
turkey-bodrum.jpg

Task: Task2
Node: tvmps_bb6dfc1d2c40d1ed4211ed84a427acc8e5c6190941e1157f2ab065467d768d75_d
Standard output:
https://www.google.com/maps/search/?api=1&query=50.8190533333333+-0.136791666666667
england-brighton-pier.jpg

Task: Task3
Node: tvmps_b4adedcce39813ba58c10d6fedc0f04a4b7561354452b15a1776d415cc5eaa39_d
Standard output:
https://www.google.com/maps/search/?api=1&query=47.4669055555556+10.20375
germany-allgaeu-fliegenpilz.jpg

Task: Task4
Node: tvmps_bb6dfc1d2c40d1ed4211ed84a427acc8e5c6190941e1157f2ab065467d768d75_d
Standard output:
https://www.google.com/maps/search/?api=1&query=52.1392766572306+-10.2745947971778
irland-dingle.jpg

Task: Task5
Node: tvmps_bb6dfc1d2c40d1ed4211ed84a427acc8e5c6190941e1157f2ab065467d768d75_d
Standard output:
https://www.google.com/maps/search/?api=1&query=48.1457277777778+11.5876888888889
germany-english-garden.jpg

Task: Task6
Node: tvmps_bb6dfc1d2c40d1ed4211ed84a427acc8e5c6190941e1157f2ab065467d768d75_d
Standard output:
https://www.google.com/maps/search/?api=1&query=48.268275+11.6033611111111
germany-garching-heide.jpg

Task: Task7
Node: tvmps_bb6dfc1d2c40d1ed4211ed84a427acc8e5c6190941e1157f2ab065467d768d75_d
Standard output:
https://www.google.com/maps/search/?api=1&query=25.1082038611111+121.843948361111
taiwan-jiufen.jpg

Task: Task8
Node: tvmps_bb6dfc1d2c40d1ed4211ed84a427acc8e5c6190941e1157f2ab065467d768d75_d
Standard output:
https://www.google.com/maps/search/?api=1&query=45.8776305555556+10.8571611111111
italy-garda-lake-sailing-club.jpg

Task: Task9
Node: tvmps_bb6dfc1d2c40d1ed4211ed84a427acc8e5c6190941e1157f2ab065467d768d75_d
Standard output:
https://www.google.com/maps/search/?api=1&query=35.014377+135.669015
japan-katsura-river.jpg


Sample end: 2020-05-22 11:09:25
Elapsed time: 0:04:02

Delete job? [Y/n]
Delete pool? [Y/n]
Deleting container [input1]...

Press ENTER to exit...
{{< / highlight >}}



