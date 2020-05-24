+++
author = "Chris Suttles"
categories = ["vault", "chef", "secrets", "automation", "Python", "consul"]
date = 2018-01-08T05:27:56Z
description = ""
draft = false
cover = "/images/2018/01/Screen-Shot-2018-01-07-at-1.14.27-PM.png"
slug = "managing-dynamic-database-credentials-with-hashicorp-vault-and-chef"
tags = ["vault", "chef", "secrets", "automation", "Python", "consul"]
title = "Managing Dynamic Database Credentials With Hashicorp Vault and Chef"

+++


This post takes a look at using [Hashicorp's Vault](https://www.vaultproject.io/) to manage dynamic database credentials, using [Chef](https://www.chef.io/).

For this post (and the [my previous post on Vault](http://blog.highspeedlogic.org/managing-ssh-secrets-with-vault/)), I started working with Vault pretty quickly via this `docker-compose` [setup I found via GitHub](https://github.com/tolitius/cault). It's a very quick way to get a Vault instance with a [Consul](https://www.consul.io/) backend. You'd never do this for production, since they are single instances, but for functional testing, it's enough.

The code that corresponds to this post is available in the following git repository: [https://github.com/csuttles/chef-secrets-in-hashi-vault](https://github.com/csuttles/chef-secrets-in-hashi-vault)

# Requirements

What are we trying to accomplish?

Let's verify the ability to generate dynamic database secrets, and provision an application with access to the secrets. Let's also verify the revocation process for the dynamic secrets.

## Setup, Unseal, and Use Vault

I'm reusing the same setup from last time, so if you set that up too, you can skip this part.

[Refer to this github repo](https://github.com/tolitius/cault) for bootstrapping the containers, and unsealing the vault as well as environment set up, and creating/accessing a credential, and creating a one-time token, and using it for access to a key (secret) stored in Vault.

## Chef, Consul Templates, and Hashicorp Vault

### Overview

This post and the associated tasks are heavily based on [this Hashicorp blog post](https://www.hashicorp.com/blog/using-hashicorps-vault-with-chef) by [@sethvargo](https://twitter.com/sethvargo). I had to make a few changes, but the premise for this post is essentially "approach 1" in the Seth Vargo post I mentioned. I wanted to go into more detail and make it go, not just grok the ideas, so here we go.

### Configuring Vault

We need to enable the [database backend](https://www.vaultproject.io/docs/secrets/databases/index.html), and follow the quickstart steps there. I also used MySQL, but many other RDBMS are supported. You'll need to tweak the MySQL connection string (and database engine) as applicable when doing the quickstart.

Once that is done, you'll also need to create a policy so you can issue tokens used to unlock the secrets in your database role.

I created a policy file like this:

{{< highlight markdown >}}
 bash
path "database/roles/readonly" {
  capabilities = ["read"]
}

{{< / highlight >}}

Then created the policy with:

{{< highlight markdown >}}
 bash
vault policy-write database-readonly database-readonly.hcl

{{< / highlight >}}

Next I generated a renewable token for reading those credentials with:

{{< highlight markdown >}}
 bash
vault token-create -policy=database-readonly

{{< / highlight >}}

We can see the details of the token using `vault token-lookup`:

{{< highlight markdown >}}
 bash
vault token-lookup acf245df-93aa-a853-e89a-01084d7c7af6
Key                     Value
---                     -----
accessor                10746d74-ec94-f932-5f16-fcfea199d65f
creation_time           1515292361
creation_ttl            2764800
display_name            token
entity_id
expire_time             2018-02-08T02:32:41.463680434Z
explicit_max_ttl        0
id                      acf245df-93aa-a853-e89a-01084d7c7af6
issue_time              2018-01-07T02:32:41.463657262Z
last_renewal            2018-01-07T04:08:15.21354412Z
last_renewal_time       1515298095
meta                    <nil>
num_uses                0
orphan                  false
path                    auth/token/create
policies                [database-readonly default]
renewable               true
ttl                     2758960

{{< / highlight >}}

This token is how the Consul Template authenticates to Vault, where it reads the database credentials, which get stored in a config file on disk for our application to read. While plaintext credentials on disk are not secure, you can do some things to mitigate this risk. Vault is already rotating these credentials for you, so they are short lived. You could also use a ramdisk, which makes the most sense if you wanted to leverage this in a cloud native environment with containers. In that case, you would run the Consul Template services as a sidecar container, use a tmpfs (ramdisk) type volume for both containers, and then mount the volume in both. In this case, the Consul Template container would write the config, and the other container running your application would read it from the shared, ephemeral volume. The current best practice is to specify a ttl that is long enough for one connection to the database only; a revocation or ttl expiry does not disconnect current sessions so the idea is that every time your application connects to the database, it does so with a fresh set of credentials. With the quick ttl expiry, even if a set of credentials were compromised, the exposure window is pretty small, and if you use unique tokens and roles for each application instance, the attack surface becomes quite small too.

#### Manual Verification of Vault Database Credentials

At this point, it's useful to know if what we've done so far is correct.

Running:

{{< highlight markdown >}}
 bash
vault read database/creds/readonly

{{< / highlight >}}

Should produce temporary credentials like so:
{{< highlight markdown >}}
 bash
Key             Value
---             -----
lease_id        database/creds/readonly/a2ea299d-d8d5-75cf-e77a-e7fb19ee2af8
lease_duration  5m0s
lease_renewable true
password        A1a-x6vxvp7v4z23wsyp
username        v-root-readonly-tvrsw560u8t5t309

{{< / highlight >}}
Note that your ttl may differ and depends on what you specified when creating the database role.

Now, we simply access the database using the credentials we got from Vault, before the ttl expires. In this example, it would be something like this:

{{< highlight markdown >}}
 bash
mysql -u v-root-readonly-tvrsw560u8t5t309 -p -h mysql.example.com

{{< / highlight >}}

You should be able to connect and select on any database, based on the role we defined earlier. Try a simple query to verify:

{{< highlight markdown >}}
 bash
mysql> SELECT user FROM mysql.user;
+----------------------------------+
| user                             |
+----------------------------------+
| root                             |
| v-root-readonly-tvrsw560u8t5t309 |
| v-token-readonly-54r6tr9z419wu7p |
| v-token-readonly-y37222w48xu391w |
| mysql.session                    |
| mysql.sys                        |
| root                             |
+----------------------------------+
7 rows in set (0.00 sec)

{{< / highlight >}}

In this example I am looking at users in mysql. Note the users created by Vault are all formatted similarly, and it is easy to see that I have one temporary readonly user created via root access to Vault, and two readonly users that were created using the token we created earlier.

*Don't worry if your output doesn't show token users yet; I ran this query after completing all the steps in this post, when writing things up at the end.*


### Configuring the Application via Chef

I started with the examples in [this gist](https://gist.github.com/sethvargo/6f1a315094fbd1a18c6d). Those didn't work for me out of the box, but they also say not to expect that. I found that gist, and the post that lead me there incredibly helpful, and the gist was a great starting point.

#### Consul Template Sidecar

Here's where [that repo I mentioned earlier](https://github.com/csuttles/chef-secrets-in-hashi-vault) is going to be useful.

I don't want to paste the whole recipe here, but the parts that I'm referring to as a sidecar are basically:

[part 1](https://github.com/csuttles/chef-secrets-in-hashi-vault/blob/master/cookbooks/echoserver/recipes/default.rb#L8-L39)
* get the consul-template binary
* unzip it
* install it
* make required directories for configs

[part 2](https://github.com/csuttles/chef-secrets-in-hashi-vault/blob/master/cookbooks/echoserver/recipes/default.rb#L72-L106)
* create a systemd unit for the service
* register, start, and enable the service

[part 3](https://github.com/csuttles/chef-secrets-in-hashi-vault/blob/master/cookbooks/echoserver/recipes/default.rb#L108-L138)
* write config file for communication with Vault
* write config file to specify template to render into app config
* write template used to render app config

All the other parts of the Chef recipe are for the actual application. In practice, I would normally put all the Consul Template stuff in another cookbook, and use it like a wrapper cookbook or library so that you could simply include it with your application cookbook and set a couple attributes to render the right templates for your app.

#### Our Application

Our application is very simple and very small. It takes less of the chef recipe than the Consul Template stuff because it doesn't have many parts.

It's based on an example from an excellent series in USENIX ;login: by [@dabeaz](https://twitter.com/dabeaz) (David Beazley), called ["A Tale of Two Concurrencies"](https://www.usenix.org/publications/login/june15/beazley). I basically took the first example from that article I had lying around in an old repository and [added a couple simple things to make it more applicable to this exercise](https://github.com/csuttles/chef-secrets-in-hashi-vault/blob/master/cookbooks/echoserver/files/default/echoserver#L3-L17).

This is a super simple, single threaded application that opens a socket on port 25000. It prints a message locally indicating each new client connection and echoes anything they send to the server right back to them.

The parts I added were [reading a config file to get our database connection info](https://github.com/csuttles/chef-secrets-in-hashi-vault/blob/master/cookbooks/echoserver/files/default/echoserver#L8-L10), [connecting to the database instance and retrieving a list of databases](https://github.com/csuttles/chef-secrets-in-hashi-vault/blob/master/cookbooks/echoserver/files/default/echoserver#L12-L17), and printing the [database list](https://github.com/csuttles/chef-secrets-in-hashi-vault/blob/master/cookbooks/echoserver/files/default/echoserver#L42) and [welcome message to the client upon connection](https://github.com/csuttles/chef-secrets-in-hashi-vault/blob/master/cookbooks/echoserver/files/default/echoserver#L30-L31). None of this is very fancy, it's just enough to demonstrate how we tie Vault credentials to an application deployed via Chef. It could be better in a lot of ways, like reading the host and port from the config, error handling, unit tests, etc.

### Testing the Application

I checked things in test kitchen a LOT (using only CentOS7). Once I got test kitchen to converge successfully, I also tested on a local linux box in my home lab. Running `chef-client -o recipe[echoserver]` got me all the necessary things installed on my test box. Once that was done, I verified the service was running via a simple telnet command:

{{< highlight markdown >}}
 bash
csuttles@devnull:[~]: telnet localhost 25000
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
Welcome to echoserver. Here is a list of databases. Anything you type will be echoed back to you. Have fun!
information_schema
mysql
performance_schema
sys^]
telnet> q
Connection closed.

{{< / highlight >}}

This step took a lot longer than the paragraph above might lead you to believe. Here's a list of things to check if it doesn't go super silky smooth as described.

* Check status of consul-template and echoserver services
* Walk consul-template configs
    * Can the service communicate with Vault?
    * Is the config being rendered, in the right location, as desired?
* If everything with consul-template is OK, check echoserver
    * All dependencies installed?
    * Can you connect to the MySQL instance from your host manually?
    * Stop the service and run manually to see STDOUT/STDERR for clues

All of those things took me time to sort out. There's definitely a learning curve, but once you grok the process and template language(s), it's pretty easy.

### Revoking Credentials

The token we created is used by consul-template to communicate with vault and get database credentials, so we can revoke access by simply revoking that token. Here's a lookup of the token we created for reference:

{{< highlight markdown >}}
 bash
~ # vault token-lookup acf245df-93aa-a853-e89a-01084d7c7af6
Key                     Value
---                     -----
accessor                10746d74-ec94-f932-5f16-fcfea199d65f
creation_time           1515292361
creation_ttl            2764800
display_name            token
entity_id
expire_time             2018-02-08T02:32:41.463679735Z
explicit_max_ttl        0
id                      acf245df-93aa-a853-e89a-01084d7c7af6
issue_time              2018-01-07T02:32:41.463657262Z
last_renewal            2018-01-07T20:56:09.987322489Z
last_renewal_time       1515358569
meta                    <nil>
num_uses                0
orphan                  false
path                    auth/token/create
policies                [database-readonly default]
renewable               true
ttl                     2698554

{{< / highlight >}}

Here's a [call to the Vault API, looking for leases](https://www.vaultproject.io/api/system/leases.html) related to our database credentials:

{{< highlight markdown >}}
 bash
csuttles@devnull:[~/src/cault]: curl --header "X-Vault-Token:$VAULT_TOKEN" --request LIST "$VAULT_ADDR/v1/sys/leases/lookup/database/creds/readonly" | jq
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   285  100   285    0     0    285      0  0:00:01 --:--:--  0:00:01 35625
{
  "request_id": "8a8d089b-3a15-3763-1db9-04f33b154229",
  "lease_id": "",
  "renewable": false,
  "lease_duration": 0,
  "data": {
    "keys": [
      "46d418c7-1f1b-fdd6-62dc-f4832404fd15",
      "46dbaa57-7fd1-5d44-bc91-0e760236e03f",
      "4b26863b-f099-c46b-6915-c06673463f15"
    ]
  },
  "wrap_info": null,
  "warnings": null,
  "auth": null
}
[Exit: 0 0] 12:56

{{< / highlight >}}

Now we can revoke the token from which our leases our derived. Since these leases are what creates the temporary users in MySQL, revoking the token, and therefore the leases, will also revoke the MySQL credentials. It is also possible to revoke individual leases, but if you only revoke the individual lease, the token is still valid, and Consul Template simply uses the same token to get another lease. This means you must plan carefully when mapping your tokens and leases, and think about how a revocation will impact you. If you set everything up under a single token, you will only get a "STOP EVERYTHING" button. That's better than no revocation, but not much. A better approach would be more granular division of resources each assigned a token so that revoking a token revokes access for a logical grouping of instances, like a rack, a datacenter, production or dev, etc.

With all that said, let's revoke this token:

{{< highlight markdown >}}
 bash
~ # vault token-revoke acf245df-93aa-a853-e89a-01084d7c7af6
Success! Token revoked if it existed.

{{< / highlight >}}

We can issue the same API (`leases/lookup`) request to verify that the leases derived from that token are also revoked:

{{< highlight markdown >}}
 bash
csuttles@devnull:[~/src/cault]: curl --header "X-Vault-Token:$VAULT_TOKEN" --request LIST "$VAULT_ADDR/v1/sys/leases/lookup/database/creds/readonly" | jq
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    14  100    14    0     0     14      0  0:00:01 --:--:--  0:00:01  1750
{
  "errors": []
}

{{< / highlight >}}

Finally, we can check our application and verify it cannot connect to the database. When restarting the application to force a new MySQL connection, I get the following error, which is oddly satisfying in this case:

{{< highlight markdown >}}
 bash
csuttles@devnull:[~]: /usr/local/bin/echoserver
Traceback (most recent call last):
  File "/usr/lib64/python3.4/site-packages/sqlalchemy/pool.py", line 1122, in _do_get
    return self._pool.get(wait, self._timeout)
  File "/usr/lib64/python3.4/site-packages/sqlalchemy/util/queue.py", line 145, in get
    raise Empty
sqlalchemy.util.queue.Empty

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "/usr/lib64/python3.4/site-packages/sqlalchemy/engine/base.py", line 2141, in _wrap_pool_connect
    return fn()
  File "/usr/lib64/python3.4/site-packages/sqlalchemy/pool.py", line 328, in unique_connection
    return _ConnectionFairy._checkout(self)
  File "/usr/lib64/python3.4/site-packages/sqlalchemy/pool.py", line 766, in _checkout
    fairy = _ConnectionRecord.checkout(pool)
  File "/usr/lib64/python3.4/site-packages/sqlalchemy/pool.py", line 516, in checkout
    rec = pool._do_get()
  File "/usr/lib64/python3.4/site-packages/sqlalchemy/pool.py", line 1138, in _do_get
    self._dec_overflow()
...
sqlalchemy.exc.OperationalError: (pymysql.err.OperationalError) (1045, "Access denied for user 'v-token-readonly-50yq76zvp4vz3u0'@'172.17.0.1' (using password: YES)")

{{< / highlight >}}

This means that we tried to connect using the credentials derived from Vault, but were denied. We can also confirm the users are gone in MySQL:

{{< highlight markdown >}}
 bash
mysql> SELECT user FROM mysql.user;
+---------------+
| user          |
+---------------+
| root          |
| mysql.session |
| mysql.sys     |
| root          |
+---------------+
4 rows in set (0.01 sec)

{{< / highlight >}}

If we wanted to restore access at this point, we could simply generate a new token like we did previously:

{{< highlight markdown >}}
 bash
vault token-create -policy=database-readonly

{{< / highlight >}}
