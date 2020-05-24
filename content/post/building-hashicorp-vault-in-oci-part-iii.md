+++
author = "Chris Suttles"
categories = ["OCI", "terraform", "consul", "vault", "cloud", "automation"]
date = 2018-11-17T13:40:17Z
description = ""
draft = false
cover = "/images/2018/11/Screen-Shot-2018-10-09-at-1.17.31-PM-1.png"
slug = "building-hashicorp-vault-in-oci-part-iii"
tags = ["OCI", "terraform", "consul", "vault", "cloud", "automation"]
title = "Building Hashicorp Vault in OCI - Part III"

+++


This post is the last in a series on deploying the [Hashicorp recommended architecture for a single DC](https://www.vaultproject.io/guides/operations/reference-architecture.html#one-dc) deployment of Vault on [Oracle Cloud Infrastructure (OCI)](https://cloud.oracle.com/cloud-infrastructure). Here are some related links:

* [https://github.com/csuttles/oci-vault/](https://github.com/csuttles/oci-vault/) (the code for all of this)
* [http://blog.csuttles.io/getting-started-with-terraform-on-oracle-cloud-infrastructure-oci/](http://blog.csuttles.io/getting-started-with-terraform-on-oracle-cloud-infrastructure-oci/) (intro)
* [http://blog.csuttles.io/building-hashicorp-vault-in-oci-part-i/](http://blog.csuttles.io/building-hashicorp-vault-in-oci-part-i/) (part i)
* [http://blog.csuttles.io/building-hashicorp-vault-in-oci-part-ii/](http://blog.csuttles.io/building-hashicorp-vault-in-oci-part-ii/) (part ii)
* [http://blog.csuttles.io/building-hashicorp-vault-in-oci-part-iii/](http://blog.csuttles.io/building-hashicorp-vault-in-oci-part-iii/) (this article)

## Create Vault Nodes

In the previous articles in the series, we built out prerequisite resources, including compartments, a VCN, subnets, seclists, and finally consul servers. After doing all of that, adding Hashicorp Vault is pretty simple.Let's take a look at our `vault.tf` file:

{{< highlight markdown >}}

csuttles@cs-mbp15:[~/src/oci-vault/iad/vault]:(master)
[Exit: 0] 23:13: cat vault.tf
//vault nodes
// consul nodes
resource "oci_core_instance" "vault" {
  count               = 3
  availability_domain = "${lookup(data.oci_identity_availability_domains.ADs.availability_domains[count.index],"name")}"
  compartment_id      = "${data.terraform_remote_state.common.vault_compartment}"
  display_name        = "consul-${count.index}"
  shape               = "${var.instance_shape}"

  create_vnic_details {
    subnet_id        = "${data.terraform_remote_state.network.vault_subnets[count.index]}"
    display_name     = "primaryvnic"
    assign_public_ip = true
    hostname_label   = "vault-${count.index}"
  }

  source_details {
    source_type = "image"
    source_id   = "${var.instance_image_ocid}"

    # Apply this to set the size of the boot volume that's created for this instance.
    # Otherwise, the default boot volume size of the image is used.
    # This should only be specified when source_type is set to "image".
    #boot_volume_size_in_gbs = "60"
  }

  # Apply the following flag only if you wish to preserve the attached boot volume upon destroying this instance
  # Setting this and destroying the instance will result in a boot volume that should be managed outside of this config.
  # When changing this value, make sure to run 'terraform apply' so that it takes effect before the resource is destroyed.
  #preserve_boot_volume = true

  metadata {
    ssh_authorized_keys = "${var.ssh_public_key}"
    user_data           = "${base64encode(file("${path.module}/user-data/vault.txt"))}"
  }
  freeform_tags = "${map("vault-server", "freeformvalue${count.index}")}"
  timeouts {
    create = "60m"
  }
}

output "vault_instances" {
 value = ["${oci_core_instance.vault.*.id}"]
}

output "vault_instance_public_ips" {
 value = ["${oci_core_instance.vault.*.public_ip}"]
}

{{< / highlight >}}

It's ridiculously similar to the consul file:

{{< highlight markdown >}}

csuttles@cs-mbp15:[~/src/oci-vault/iad/vault]:(master)
[Exit: 2] 23:16: diff -U 0 consul.tf vault.tf
--- consul.tf	2018-11-05 20:21:48.000000000 -0600
+++ vault.tf	2018-11-16 23:13:51.000000000 -0600
@@ -0,0 +1 @@
+//vault nodes
@@ -2,4 +3,3 @@
-
-resource "oci_core_instance" "consul" {
-  count               = 5
-  availability_domain = "${lookup(data.oci_identity_availability_domains.ADs.availability_domains[lookup(var.consul_node_to_ad_map, count.index, 1) - 1],"name")}"
+resource "oci_core_instance" "vault" {
+  count               = 3
+  availability_domain = "${lookup(data.oci_identity_availability_domains.ADs.availability_domains[count.index],"name")}"
@@ -11 +11 @@
-    subnet_id        = "${data.terraform_remote_state.network.vault_subnets[lookup(var.consul_node_to_ad_map, count.index, 1) - 1]}"
+    subnet_id        = "${data.terraform_remote_state.network.vault_subnets[count.index]}"
@@ -14 +14 @@
-    hostname_label   = "consul-${count.index}"
+    hostname_label   = "vault-${count.index}"
@@ -34 +34 @@
-    user_data           = "${base64encode(file("${path.module}/user-data/consul.txt"))}"
+    user_data           = "${base64encode(file("${path.module}/user-data/vault.txt"))}"
@@ -36 +36 @@
-  freeform_tags = "${map("consul-server", "freeformvalue${count.index}")}"
+  freeform_tags = "${map("vault-server", "freeformvalue${count.index}")}"
@@ -42,2 +42,2 @@
-output "consul_instances" {
- value = ["${oci_core_instance.consul.*.id}"]
+output "vault_instances" {
+ value = ["${oci_core_instance.vault.*.id}"]
@@ -46,2 +46,2 @@
-output "consul_instance_public_ips" {
- value = ["${oci_core_instance.consul.*.public_ip}"]
+output "vault_instance_public_ips" {
+ value = ["${oci_core_instance.vault.*.public_ip}"]

{{< / highlight >}}

So we can basically `s/consul/vault/g` and call it done? Almost

## Cloud Init

We're going to leverage more of what we have already done, which will speed things up quite a bit, but we do need to add some things. Based on the [recommended architecture](https://learn.hashicorp.com/vault/operations/ops-reference-architecture#deployment-topology-within-one-datacenter)[,] we need to build 3 Vault boxes in 3 Availability Domains, but they also need Consul running in client mode locally. We'll take our previous `cloud-config-data` and refactor it a little to get the job done. It ends up looking like this:

{{< highlight markdown >}}

[Exit: 1] 23:21: cat user-data/vault.txt
#cloud-config
# vim: syntax=yaml:ts=4:sw=4:expandtab
#
groups:
  - consul
  - vault
users:
  - default
  - name: vault
    gecos: vault
    primary_group: vault
    groups: vault
    system: true
    homedir: /etc/vault
  - name: consul
    gecos: consul
    primary_group: consul
    groups: consul
    system: true
    homedir: /etc/consul.d
write_files:
-   encoding: b64
    content: |
        W1VuaXRdCkRlc2NyaXB0aW9uPXZhdWx0IHNlcnZlcgpSZXF1aXJlcz1uZXR3b3JrLW9ubGluZS
        50YXJnZXQKQWZ0ZXI9bmV0d29yay1vbmxpbmUudGFyZ2V0CgpbU2VydmljZV0KUmVzdGFydD1v
        bi1mYWlsdXJlClVzZXI9dmF1bHQKR3JvdXA9dmF1bHQKUGVybWlzc2lvbnNTdGFydE9ubHk9dH
        J1ZQpFeGVjU3RhcnRQcmU9L3NiaW4vc2V0Y2FwICdjYXBfaXBjX2xvY2s9K2VwJyAvdXNyL2xv
        Y2FsL2Jpbi92YXVsdApFeGVjU3RhcnQ9L3Vzci9sb2NhbC9iaW4vdmF1bHQgc2VydmVyIC1jb2
        5maWcgL2V0Yy92YXVsdC92YXVsdC5oY2wKRXhlY1JlbG9hZD0vYmluL2tpbGwgLUhVUCAkTUFJ
        TlBJRApLaWxsU2lnbmFsPVNJR0lOVAoKW0luc3RhbGxdCldhbnRlZEJ5PW11bHRpLXVzZXIudG
        FyZ2V0Cg==
    owner: root:root
    path: /etc/systemd/system/vault.service
    permissions: '0644'
-   encoding: b64
    content: |
        c3RvcmFnZSAiY29uc3VsIiB7CiAgYWRkcmVzcyA9ICIxMjcuMC4wLjE6ODUwMCIKICBwYXRoIC
        AgID0gInZhdWx0LyIKfQoKbGlzdGVuZXIgInRjcCIgewogYWRkcmVzcyAgICAgPSAiMC4wLjAu
        MDo4MjAwIgogdGxzX2Rpc2FibGUgPSAxCn0K
    owner: root:root
    path: /etc/vault/vault.hcl
    permissions: '0640'
-   encoding: b64
    content: |
        c2VydmVyID0gdHJ1ZQpib290c3RyYXBfZXhwZWN0ID0gNQp1aSA9IHRydWUK
    owner: root:root
    path: /etc/vault.d/server.hcl
    permissions: '0640'
-   encoding: b64
    content: |
        W1VuaXRdCkRlc2NyaXB0aW9uPSJIYXNoaUNvcnAgQ29uc3VsIC0gQSBzZXJ2aWNlIG1lc2ggc2
        9sdXRpb24iCkRvY3VtZW50YXRpb249aHR0cHM6Ly93d3cuY29uc3VsLmlvLwpSZXF1aXJlcz1u
        ZXR3b3JrLW9ubGluZS50YXJnZXQKQWZ0ZXI9bmV0d29yay1vbmxpbmUudGFyZ2V0CkNvbmRpdG
        lvbkZpbGVOb3RFbXB0eT0vZXRjL2NvbnN1bC5kL2NvbnN1bC5oY2wKCltTZXJ2aWNlXQpVc2Vy
        PWNvbnN1bApHcm91cD1jb25zdWwKRXhlY1N0YXJ0PS91c3IvbG9jYWwvYmluL2NvbnN1bCBhZ2
        VudCAtY29uZmlnLWRpcj0vZXRjL2NvbnN1bC5kLwpFeGVjUmVsb2FkPS91c3IvbG9jYWwvYmlu
        L2NvbnN1bCByZWxvYWQKS2lsbE1vZGU9cHJvY2VzcwpSZXN0YXJ0PW9uLWZhaWx1cmUKTGltaX
        ROT0ZJTEU9NjU1MzYKCltJbnN0YWxsXQpXYW50ZWRCeT1tdWx0aS11c2VyLnRhcmdldAo=
    owner: root:root
    path: /etc/systemd/system/consul.service
    permissions: '0644'
-   encoding: b64
    content: |
        ZGF0YWNlbnRlciA9ICJkYzEiCmRhdGFfZGlyID0gIi9vcHQvY29uc3VsIgplbmNyeXB0ID0gIk
        x1ajJGWld3bHQ4NDc1d0QxV3R3VVE9PSIKcmV0cnlfam9pbiA9IFsiY29uc3VsLTAudmF1bHQx
        Lm9jaXZhdWx0Lm9yYWNsZXZjbi5jb20iLCAiY29uc3VsLTEudmF1bHQxLm9jaXZhdWx0Lm9yYW
        NsZXZjbi5jb20iLCAiY29uc3VsLTIudmF1bHQyLm9jaXZhdWx0Lm9yYWNsZXZjbi5jb20iLCAi
        Y29uc3VsLTMudmF1bHQzLm9jaXZhdWx0Lm9yYWNsZXZjbi5jb20iLCAiY29uc3VsLTQudmF1bH
        QzLm9jaXZhdWx0Lm9yYWNsZXZjbi5jb20iXQo=
    owner: root:root
    path: /etc/consul.d/consul.hcl
    permissions: '0640'
-   encoding: b64
    content: |
        c2VydmVyID0gZmFsc2UK
    owner: root:root
    path: /etc/consul.d/client.hcl
    permissions: '0640'
runcmd:
 - /sbin/usermod -s /bin/false vault
 - wget https://releases.hashicorp.com/vault/0.11.5/vault_0.11.5_linux_amd64.zip
 - unzip vault_0.11.5_linux_amd64.zip
 - chown root:root vault
 - mv vault /usr/local/bin/
 - vault --version
 - vault -autocomplete-install
 - mkdir --parents /opt/vault
 - chown --recursive vault:vault /opt/vault
 - mkdir --parents /etc/vault
 - chown --recursive vault:vault /etc/vault
 - firewall-offline-cmd --add-port=8200-8201/tcp
 - firewall-offline-cmd --add-port=8200-8201/udp
 - /sbin/usermod -s /bin/false consul
 - wget https://releases.hashicorp.com/consul/1.3.0/consul_1.3.0_linux_amd64.zip
 - unzip consul_1.3.0_linux_amd64.zip
 - chown root:root consul
 - mv consul /usr/local/bin/
 - consul --version
 - consul -autocomplete-install
 - complete -C /usr/local/bin/consul consul
 - mkdir --parents /opt/consul
 - chown --recursive consul:consul /opt/consul
 - mkdir --parents /etc/consul.d
 - chown --recursive consul:consul /etc/consul.d
 - firewall-offline-cmd --add-port=8301-8302/tcp
 - firewall-offline-cmd --add-port=8301-8302/udp
 - systemctl restart firewalld
 - systemctl daemon-reload
 - systemctl enable consul
 - systemctl start consul
 - systemctl status consul
 - systemctl enable vault
 - systemctl start vault
 - systemctl status vault

{{< / highlight >}}

Again, we've embedded config files in base64, but we can re-use several constructs, like users, group, and runcmd to make things a little easier.Here's what the plaintext versions of the relevant files look like:vault.hcl

{{< highlight markdown >}}

csuttles@cs-mbp15:[~/src/oci-vault/iad/vault]:(master)
[Exit: 0] 23:22: cat user-data/vault.hcl
storage "consul" {
  address = "127.0.0.1:8500"
  path    = "vault/"
}

listener "tcp" {
 address     = "0.0.0.0:8200"
 tls_disable = 1
}

{{< / highlight >}}

consul.hcl (stored in git as consul-client.hcl)

{{< highlight markdown >}}

csuttles@cs-mbp15:[~/src/oci-vault/iad/vault]:(master)
[Exit: 0] 23:22: cat user-data/consul-client.hcl
datacenter = "dc1"
data_dir = "/opt/consul"
encrypt = "Luj2FZWwlt8475wD1WtwUQ=="
retry_join = ["consul-0.vault1.ocivault.oraclevcn.com", "consul-1.vault1.ocivault.oraclevcn.com", "consul-2.vault2.ocivault.oraclevcn.com", "consul-3.vault3.ocivault.oraclevcn.com", "consul-4.vault3.ocivault.oraclevcn.com"]

{{< / highlight >}}

vault.service

{{< highlight markdown >}}

csuttles@cs-mbp15:[~/src/oci-vault/iad/vault]:(master)
[Exit: 0] 23:23: cat user-data/vault.service
[Unit]
Description=vault server
Requires=network-online.target
After=network-online.target

[Service]
Restart=on-failure
User=vault
Group=vault
PermissionsStartOnly=true
ExecStartPre=/sbin/setcap 'cap_ipc_lock=+ep' /usr/local/bin/vault
ExecStart=/usr/local/bin/vault server -config /etc/vault/vault.hcl
ExecReload=/bin/kill -HUP $MAINPID
KillSignal=SIGINT

[Install]
WantedBy=multi-user.target

{{< / highlight >}}

## Deploying

Again, we run a simple `terraform plan && terraform apply`, and then check the results. After building the nodes with terraform, I connected to the node `vault-0` to check the status. I started by checking consul:

{{< highlight markdown >}}

[root@vault-0 ~]# consul members
Node      Address         Status  Type    Build  Protocol  DC   Segment
consul-0  10.0.1.36:8301  alive   server  1.3.0  2         dc1  <all>
consul-1  10.0.1.35:8301  alive   server  1.3.0  2         dc1  <all>
consul-2  10.0.2.21:8301  alive   server  1.3.0  2         dc1  <all>
consul-3  10.0.3.35:8301  alive   server  1.3.0  2         dc1  <all>
consul-4  10.0.3.37:8301  alive   server  1.3.0  2         dc1  <all>
vault-0   10.0.1.37:8301  alive   client  1.3.0  2         dc1  <default>
vault-1   10.0.2.19:8301  alive   client  1.3.0  2         dc1  <default>
vault-2   10.0.3.36:8301  alive   client  1.3.0  2         dc1  <default>

{{< / highlight >}}

While we deployed the recommended architecture, I left TLS disabled in the config, so we have to tell vault client not to connect with it. Hardening and load balancing are going to be left to the reader.

{{< highlight markdown >}}

[root@vault-0 ~]# export VAULT_ADDR=http://127.0.0.1:8200
[root@vault-0 ~]# vault operator init
Unseal Key 1: YIOO/aCZk1EUGTAUxixpMZ8Q1nGNGnyS8vTmaWcSyyyN
Unseal Key 2: i879I6XXeHDFs5hF6QMfXHTd25MRe4hPkxcd8kCEOpbZ
Unseal Key 3: JjytjBvkb2dZ+N3+7/aT4iFu/EmtpDR9VhaFG4KWTvTI
Unseal Key 4: vtLdwpk15tuA5SaaqWTdERrGqoUXOf/e7ENg/5XO+1pu
Unseal Key 5: v5gtBpem7aBSH8MwSgY81qw0tYDdTfy3jg5oyWItZ4JE

Initial Root Token: Kh33Jaw8XWOCc0PtvcKvDv1X

Vault initialized with 5 key shares and a key threshold of 3. Please securely
distribute the key shares printed above. When the Vault is re-sealed,
restarted, or stopped, you must supply at least 3 of these keys to unseal it
before it can start servicing requests.

Vault does not store the generated master key. Without at least 3 key to
reconstruct the master key, Vault will remain permanently sealed!

It is possible to generate new unseal keys, provided you have a quorum of
existing unseal keys shares. See "vault operator rekey" for more information.
[root@vault-0 ~]# vault status
Key                Value
---                -----
Seal Type          shamir
Initialized        true
Sealed             true
Total Shares       5
Threshold          3
Unseal Progress    0/3
Unseal Nonce       n/a
Version            0.11.5
HA Enabled         true

{{< / highlight >}}

We can see that initializing Vault was successful, but it is still sealed. Next we will unseal it and verify the status.

{{< highlight markdown >}}

[root@vault-0 ~]# vault operator unseal
Unseal Key (will be hidden):
Key                Value
---                -----
Seal Type          shamir
Initialized        true
Sealed             true
Total Shares       5
Threshold          3
Unseal Progress    1/3
Unseal Nonce       e8689e91-a119-41f9-0a90-b86f2f0eb82d
Version            0.11.5
HA Enabled         true
[root@vault-0 ~]# vault operator unseal
Unseal Key (will be hidden):
Key                Value
---                -----
Seal Type          shamir
Initialized        true
Sealed             true
Total Shares       5
Threshold          3
Unseal Progress    2/3
Unseal Nonce       e8689e91-a119-41f9-0a90-b86f2f0eb82d
Version            0.11.5
HA Enabled         true
[root@vault-0 ~]# vault operator unseal
Unseal Key (will be hidden):
Key                    Value
---                    -----
Seal Type              shamir
Initialized            true
Sealed                 false
Total Shares           5
Threshold              3
Version                0.11.5
Cluster Name           vault-cluster-41cb8536
Cluster ID             08e0e128-c0e6-032c-4688-f21b585e7910
HA Enabled             true
HA Cluster             n/a
HA Mode                standby
Active Node Address    <none>
[root@vault-0 ~]# vault status
Key             Value
---             -----
Seal Type       shamir
Initialized     true
Sealed          false
Total Shares    5
Threshold       3
Version         0.11.5
Cluster Name    vault-cluster-41cb8536
Cluster ID      08e0e128-c0e6-032c-4688-f21b585e7910
HA Enabled      true
HA Cluster      https://10.0.1.37:8201
HA Mode         active

{{< / highlight >}}

## First Secrets

Now that we have a Vault cluster up and running, it is of course time for some secrets! What demo would be complete without a `hello world`?

{{< highlight markdown >}}

[root@vault-0 ~]# vault login
Token (will be hidden):
Success! You are now authenticated. The token information displayed below
is already stored in the token helper. You do NOT need to run "vault login"
again. Future Vault requests will automatically use this token.

Key                  Value
---                  -----
token                Kh33Jaw8XWOCc0PtvcKvDv1X
token_accessor       5MHrA58OMOwJmtCtV5kA2U7n
token_duration       ∞
token_renewable      false
token_policies       ["root"]
identity_policies    []
policies             ["root"]
[root@vault-0 ~]# vault kv put secret/hello foo=world excited=yes
Success! Data written to: secret/hello
[root@vault-0 ~]# vault kv get secret/hello
===== Data =====
Key        Value
---        -----
excited    yes
foo        world

{{< / highlight >}}
