+++
author = "Chris Suttles"
categories = ["OpenStack", "IPv6"]
date = 2017-07-23T13:13:24Z
description = ""
draft = false
cover = "/images/2017/09/mariadb-galera.jpg"
slug = "clustering-galera-on-single-stack-ipv6"
tags = ["OpenStack", "IPv6"]
title = "Clustering Galera on Single Stack IPv6"

+++


While setting up Galera on IPv6 only as part of our move toward multiple controller nodes, I ran into a few challenges. This post explains those challenges and how I dealt with them.

###### Resources I used and config reference

I found some good references on the web. The most useful ones including the following:

* [OpenStack HA Guide](https://docs.openstack.org/ha-guide/shared-database-manage.html)
* [This very helpful blog post](https://blog.widodh.nl/2016/02/mariadb-galera-cluster-on-ipv6/)
* [MariaDB/Galera Downloads](https://downloads.mariadb.org/mariadb-galera/)
* [MariaDB/Galera Docs](https://mariadb.com/kb/en/mariadb/getting-started-with-mariadb-galera-cluster/)

I also looked at:

* [This post for some Galera/Wsrep basics](https://www.globo.tech/learning-center/install-galera-ubuntu-16/)
* [This bug for hints on fixing SST transfers](https://bugs.launchpad.net/galera/+bug/1130595)

###### Getting Started

I got most of what I needed from the [blog post](https://blog.widodh.nl/2016/02/mariadb-galera-cluster-on-ipv6/) I mentioned previously. It's a great reference and starting point. I used the [MariaDB downloads page](https://downloads.mariadb.org/mariadb-galera/) to get the information needed to configure APT (we run Ubuntu 16.04 as our base OS). Once that was done, I installed MariaDB 10.1 with Galera and set up the config as mentioned in the blog post, and started trying to bootstrap the cluster. I got the nodes to join, but replication wasn't working, so I had to dig a little deeper.

For reference sake, here's a diff of my config before and after:

```
diff -u 50-server.cnf /etc/mysql/mariadb.conf.d/50-server.cnf
--- 50-server.cnf       2017-05-06 15:55:26.813413438 -0700
+++ /etc/mysql/mariadb.conf.d/50-server.cnf     2017-05-06 17:53:30.208914326 -0700
@@ -72,7 +72,7 @@
 # note: if you are setting up a replication slave, see README.Debian about
 #       other settings you may need to change.
 #server-id             = 1
-#log_bin                       = /var/log/mysql/mysql-bin.log
+log_bin                        = /var/log/mysql/mysql-bin.log
 expire_logs_days       = 10
 max_binlog_size   = 100M
 #binlog_do_db          = include_database_name
@@ -113,6 +113,23 @@
 #
 # Also available for other users if required.
 # See https://mariadb.com/kb/en/unix_socket-authentication-plugin/
+#mysql settings
+binlog_format=ROW
+default-storage-engine=innodb
+innodb_autoinc_lock_mode=2
+innodb_doublewrite=1
+query_cache_size=0
+query_cache_type=0
+#galera settings
+wsrep_on=ON
+wsrep_provider=/usr/lib/galera/libgalera_smm.so
+wsrep_provider_options = "gmcast.listen_addr=tcp://[::]:4567;ist.recv_addr=[2001:DB8::5]:4568"
+wsrep_cluster_name="RunTheJewels"
+wsrep_cluster_address=gcomm://2001:DB8::6:4567,2001:DB8::5:4567,2001:DB8::4:4567
+wsrep_sst_method=rsync
+wsrep_node_name=os202
+wsrep_node_address = "[2001:DB8::5]:4567"
+wsrep_sst_receive_address = "[2001:DB8::5]:4444"

 # this is only for embedded server
 [embedded]
```

###### IPv6 and SST

After turning up logging and digging a bit, I saw that SST was failing and rsync was throwing an error. I dug into the rsync SST script and found that the line assigning the `RSYNC_PORT` variable assumed IPv4 syntax. I could have fixed the awk statement by changing `$2` to `$NF`, but I wanted to do a simple fix and by the time I patched this I really just wanted to see replication work, so I was a bit heavy handed. I ended up with a patch like this for `/usr/bin/wsrep_sst_rsync`:

```
diff -u wsrep_sst_rsync /usr/bin/wsrep_sst_rsync
--- wsrep_sst_rsync     2017-05-23 15:37:30.596302338 -0700
+++ /usr/bin/wsrep_sst_rsync    2017-05-23 15:37:41.508184365 -0700
@@ -270,7 +270,8 @@
     rm -rf "$RSYNC_PID"

     ADDR=$WSREP_SST_OPT_ADDR
-    RSYNC_PORT=$(echo $ADDR | awk -F ':' '{ print $2 }')
+    #RSYNC_PORT=$(echo $ADDR | awk -F ':' '{ print $2 }')
+    RSYNC_PORT=4444
     if [ -z "$RSYNC_PORT" ]
     then
         RSYNC_PORT=4444
```

###### Victory

Once I patched all the nodes rsync_sst script, had all the configs set up, and did the cluster bootstrap, I was finally able to check replication and state via the `SHOW STATUS LIKE 'wsrep_%'` statement.

