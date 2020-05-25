+++
author = "Chris Suttles"
categories = ["OCI", "terraform", "cloud", "vault", "automation", "consul"]
date = 2018-11-08T02:05:43Z
description = ""
draft = false
cover = "/images/2018/11/Screen-Shot-2018-10-09-at-1.17.31-PM.png"
slug = "building-hashicorp-vault-in-oci-part-ii"
tags = ["OCI", "terraform", "cloud", "vault", "automation", "consul"]
title = "Building Hashicorp Vault in OCI - Part II"

+++


This post is a continuation of a series. The first two posts are [here](http://blog.csuttles.io/getting-started-with-terraform-on-oracle-cloud-infrastructure-oci/) and [here](http://blog.csuttles.io/building-hashicorp-vault-in-oci-part-i/)[.] The source for this series is [available on GitHub](https://github.com/csuttles/oci-vault).

## Building Consul in OCI

Now that we have defined the IAM and network resources that Vault depends on, it's time to start building Consul nodes, which we will use as the backend for Vault.

In order to build Consul, and completely automate the bootstrap, we will take advantage of some OCI and Terraform features. In OCI, we will leverage the internal DNS so that we can render a static configuration file with the hostnames for the cluster pre-populated. In order for this to work, the `dns_label` must be defined at both the VCN and subnet levels, and in addition, you must set the `hostname_label` on the `oci_core_instance` resource. All of this together allows us to leverage the OCI internal DNS service, and resolve the hostnames for the consul nodes as: `$instance_hostname_label.$subnet_dns_label.$vcn_dns_label.oraclevcn.com`

In addition to using these attributes for our resources in our VCN, we need to do some standard configuration and open the necessary ports in both the system's firewall, and in the seclist definitions for our VCN. Finally, we also need to define the Consul configuration to leverage these attributes.

## Seclists

Let's start with opening ports we will need, [as defined in the Consul documentation](https://www.consul.io/docs/agent/options.html#ports).

{{< highlight markdown >}}
csuttles@cs-mbp15:[~/src/oci-vault/iad/network]:(master)
[Exit: 0] 20:27: cat seclists.tf
resource "oci_core_default_security_list" "oci-vault-default-security-list" {
  manage_default_resource_id = "${oci_core_virtual_network.oci-vault-vcn1.default_security_list_id}"
  display_name               = "oci-vault-default-security-list"

  // allow outbound tcp traffic on all ports
  egress_security_rules {
    destination = "0.0.0.0/0"
    protocol    = "6"
  }

  // allow outbound udp traffic
  egress_security_rules {
    destination = "0.0.0.0/0"
    protocol    = "17"        // udp
  }

  // allow inbound ssh traffic
  ingress_security_rules {
    protocol  = "6"         // tcp
    source    = "0.0.0.0/0"
    stateless = false

    tcp_options {
      "min" = 22
      "max" = 22
    }
  }

  // allow inbound consul traffic
  ingress_security_rules {
    protocol  = "6"         // tcp
    source    = "0.0.0.0/0"
    stateless = false

    tcp_options {
      "min" = 8300
      "max" = 8302
    }
  }
  // allow inbound consul traffic
  ingress_security_rules {
    protocol  = "17"         // udp
    source    = "0.0.0.0/0"
    stateless = false

    udp_options {
      "min" = 8300
      "max" = 8302
    }
  }
  // allow inbound consul traffic
  ingress_security_rules {
    protocol  = "6"         // tcp
    source    = "0.0.0.0/0"
    stateless = false

    tcp_options {
      "min" = 8500
      "max" = 8502
    }
  }
  // allow inbound consul traffic
  ingress_security_rules {
    protocol  = "17"         // udp
    source    = "0.0.0.0/0"
    stateless = false

    udp_options {
      "min" = 8500
      "max" = 8502
    }
  }
  // allow inbound consul traffic
  ingress_security_rules {
    protocol  = "6"         // tcp
    source    = "0.0.0.0/0"
    stateless = false

    tcp_options {
      "min" = 8600
      "max" = 8600
    }
  }
  // allow inbound consul traffic
  ingress_security_rules {
    protocol  = "17"         // udp
    source    = "0.0.0.0/0"
    stateless = false

    udp_options {
      "min" = 8600
      "max" = 8600
    }
  }
  // allow inbound icmp traffic of a specific type
  ingress_security_rules {
    protocol  = 1
    source    = "0.0.0.0/0"
    stateless = true

    icmp_options {
      "type" = 3
      "code" = 4
    }
  }
}
{{< / highlight >}}

These include DNS and UI access, but the default behavior is to bind these services to the loopback, so they are not actually accessible by default. They are included in the seclist for convenience.

## Create Nodes

To build consul, we need somewhere to put it. Based on the [recommended architecture](https://learn.hashicorp.com/vault/operations/ops-reference-architecture#deployment-topology-within-one-datacenter)[,] we will use 5 Consul servers spread across 3 subnets as our storage backend for Vault. We'll define a map variable to allow us to spread the servers in the desired placement while doing a simple `count = 5` for the `oci_core_instance` resource.

{{< highlight markdown >}}
variable "consul_node_to_ad_map" {
  type = "map"

  default = {
    "0" = "1"
    "1" = "1"
    "2" = "2"
    "3" = "3"
    "4" = "3"
  }
}
{{< / highlight >}}

Here's the Terraform config for the Consul servers. I leverage the output from the remote state for network and common to get the subnet and compartment, respectively. Then we define some metadata, including defining ssh_authorized_keys (so we can log in) and the user-data, which is how we will configure the Consul cluster. I also defined some freeform tags, which we can use to label and organize our resources within OCI. I also added some outputs for convenience (the OCID of the instances and public IPs).

{{< highlight markdown >}}
csuttles@cs-mbp15:[~/src/oci-vault/iad/vault]:(master)
[Exit: 0] 20:34: cat consul.tf
// consul nodes

resource "oci_core_instance" "consul" {
  count               = 5
  availability_domain = "${lookup(data.oci_identity_availability_domains.ADs.availability_domains[lookup(var.consul_node_to_ad_map, count.index, 1) - 1],"name")}"
  compartment_id      = "${data.terraform_remote_state.common.vault_compartment}"
  display_name        = "consul-${count.index}"
  shape               = "${var.instance_shape}"

  create_vnic_details {
    subnet_id        = "${data.terraform_remote_state.network.vault_subnets[lookup(var.consul_node_to_ad_map, count.index, 1) - 1]}"
    display_name     = "primaryvnic"
    assign_public_ip = true
    hostname_label   = "consul-${count.index}"
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
    user_data           = "${base64encode(file("${path.module}/user-data/consul.txt"))}"
  }
  freeform_tags = "${map("consul-server", "freeformvalue${count.index}")}"
  timeouts {
    create = "60m"
  }
}

output "consul_instances" {
 value = ["${oci_core_instance.consul.*.id}"]
}

output "consul_instance_public_ips" {
 value = ["${oci_core_instance.consul.*.public_ip}"]
}
{{< / highlight >}}

## Cloud Init

We'll use `cloud-config-data` style cloud-init, passed to the instances via user-data for the actual configuration of the instances.

There is a _lot_ going on here. This userdata creates the consul user and group, writes the config files we've embedded in base64, and then runs a series of commands. These commands configure the consul user (some parameters are not available via `cloud-config-data`) install consul, ensure correct permissions, configure the system firewall, and finally register, enable and start the consul service unit via systemd.

{{< highlight markdown >}}
csuttles@cs-mbp15:[~/src/oci-vault/iad/vault]:(master)
[Exit: 0] 20:33: cat user-data/consul.txt
#cloud-config
# vim: syntax=yaml:ts=4:sw=4:expandtab
#
groups:
  - consul
users:
  - default
  - name: consul
    gecos: consul
    primary_group: consul
    groups: consul
    system: true
    homedir: /etc/consul.d
write_files:
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
        QzLm9jaXZhdWx0Lm9yYWNsZXZjbi5jb20iXQpwZXJmb3JtYW5jZSB7CiAgcmFmdF9tdWx0aXBs
        aWVyID0gMQp9Cg==
    owner: root:root
    path: /etc/consul.d/consul.hcl
    permissions: '0640'
-   encoding: b64
    content: |
        c2VydmVyID0gdHJ1ZQpib290c3RyYXBfZXhwZWN0ID0gNQp1aSA9IHRydWUK
    owner: root:root
    path: /etc/consul.d/server.hcl
    permissions: '0640'
runcmd:
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
 - firewall-offline-cmd --add-port=8300-8302/tcp
 - firewall-offline-cmd --add-port=8300-8302/udp
 - firewall-offline-cmd --add-port=8500-8502/tcp
 - firewall-offline-cmd --add-port=8500-8502/udp
 - firewall-offline-cmd --add-port=8600/tcp
 - firewall-offline-cmd --add-port=8600/udp
 - systemctl restart firewalld
 - systemctl daemon-reload
 - systemctl enable consul
 - systemctl start consul
 - systemctl status consul
{{< / highlight >}}

The systemd unit file is pretty simple, and is the same configuration used in the [Consul deployment guide](https://www.consul.io/docs/guides/deployment-guide.html#configure-systemd).

{{< highlight markdown >}}
csuttles@cs-mbp15:[~/src/oci-vault/iad/vault]:(master)
[Exit: 0] 11:48: cat user-data/consul.service
[Unit]
Description="HashiCorp Consul - A service mesh solution"
Documentation=https://www.consul.io/
Requires=network-online.target
After=network-online.target
ConditionFileNotEmpty=/etc/consul.d/consul.hcl

[Service]
User=consul
Group=consul
ExecStart=/usr/local/bin/consul agent -config-dir=/etc/consul.d/
ExecReload=/usr/local/bin/consul reload
KillMode=process
Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
{{< / highlight >}}

The configuration files for Consul are also very simple.

{{< highlight markdown >}}
csuttles@cs-mbp15:[~/src/oci-vault/iad/vault]:(master)
[Exit: 0] 11:49: cat user-data/consul.hcl
datacenter = "dc1"
data_dir = "/opt/consul"
encrypt = "Luj2FZWwlt8475wD1WtwUQ=="
retry_join = ["consul-0.vault1.ocivault.oraclevcn.com", "consul-1.vault1.ocivault.oraclevcn.com", "consul-2.vault2.ocivault.oraclevcn.com", "consul-3.vault3.ocivault.oraclevcn.com", "consul-4.vault3.ocivault.oraclevcn.com"]
performance {
  raft_multiplier = 1
}
csuttles@cs-mbp15:[~/src/oci-vault/iad/vault]:(master)
[Exit: 0] 11:49: cat user-data/server.hcl
server = true
bootstrap_expect = 5
ui = true
{{< / highlight >}}

This is where we are leveraging the internal DNS; the `retry_join` parameter in the config file is using a list of internal DNS names based on the `$instance_hostname_label.$subnet_dns_label.$vcn_dns_label.oraclevcn.com` hostname specification discussed earlier. This, along with `bootstrap_expect = 5` parameter allows the nodes to discover each other and bootstrap the cluster without any user interaction at all.While we have `ui = true` defined, the default behavior is to bind on the `client` service address, which defaults to `127.0.0.1`.  This means they are not accessible without establishing a tunnel to the host (we're going to use this for Vault). For convenience, they can be enabled by appending the following to `/etc/consul.d/consul.hcl` and restarting the service once the nodes are created:

{{< highlight markdown >}}
  "client_addr": "0.0.0.0"
{{< / highlight >}}

This only really would need to be done on one node, if you wanted to check out the UI for debugging or play around with the DNS interface, which is why I left it unexposed by default (again, using this to store secrets).This will be refined in later posts in this series.

## Summary

All of this adds up to another simple `terraform apply`, run from within the vault directory, which deploys the nodes we will use for Consul. The `cloud-config-data` we pass in `user-data` then configures the nodes post-boot, and as the nodes configure themselves and install consul, they ultimately discover each other and form the Consul cluster we will use as our Vault storage backend.

{{< highlight markdown >}}
[root@consul-0 ~]# consul members
Node      Address         Status  Type    Build  Protocol  DC   Segment
consul-0  10.0.1.30:8301  alive   server  1.3.0  2         dc1  <all>
consul-1  10.0.1.31:8301  alive   server  1.3.0  2         dc1  <all>
consul-2  10.0.2.16:8301  alive   server  1.3.0  2         dc1  <all>
consul-3  10.0.3.31:8301  alive   server  1.3.0  2         dc1  <all>
consul-4  10.0.3.30:8301  alive   server  1.3.0  2         dc1  <all>
[root@consul-0 ~]#
{{< / highlight >}}

