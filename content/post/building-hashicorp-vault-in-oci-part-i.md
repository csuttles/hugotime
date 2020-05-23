+++
author = "Chris Suttles"
categories = ["OCI", "terraform", "cloud", "vault", "automation"]
date = 2018-10-29T03:08:00Z
description = ""
draft = false
cover = "/images/2018/10/Screen-Shot-2018-10-09-at-1.17.31-PM-1.png"
slug = "building-hashicorp-vault-in-oci-part-i"
tags = ["OCI", "terraform", "cloud", "vault", "automation"]
title = "Building Hashicorp Vault in OCI - Part I"

+++


This post will [continue a previous post](http://blog.csuttles.io/getting-started-with-terraform-on-oracle-cloud-infrastructure-oci/) on using Hashicorp's Terraform with OCI (Oracle Cloud Infrastructure).

## Building the Network Resources

Let's walk through a single region where we will build out the network resources where our Vault installation will reside. Here's the variables where we define the storage backend and Terraform provider. It's the same basic setup [as defined in my previous post](http://blog.csuttles.io/getting-started-with-terraform-on-oracle-cloud-infrastructure-oci/#define-the-provider-and-storage-backend).

```
csuttles@cs-mbp15:[~/src/oci-vault/iad/network]:(master)
[Exit: 0] 11:19: cat variables.tf
variable "tenancy" {}
variable "tenancy_ocid" {}
variable "user_ocid" {}
variable "fingerprint" {}
variable "private_key_path" {}
variable "region" {
default = "us-ashburn-1"
}
variable "compartment_ocid" {}

provider "oci" {
tenancy_ocid = "${var.tenancy_ocid}"
user_ocid = "${var.user_ocid}"
fingerprint = "${var.fingerprint}"
private_key_path = "${var.private_key_path}"
region = "${var.region}"
version = "~> 3.1.0"
}

```

...

```
# use s3 compatible API
terraform {
backend "s3" {
bucket   = "oci-vault"
key      = "common/terraform.tfstate"
region   = "us-ashburn-1"
endpoint = "https://YOUR_TENANCY_NAME.compat.objectstorage.REGION.oraclecloud.com"
skip_region_validation      = true
skip_credentials_validation = true
skip_requesting_account_id  = true
skip_get_ec2_platforms      = true
skip_metadata_api_check     = true
force_path_style            = true
shared_credentials_file     = "/Users/csuttles/.aws/oci-creds"
}
}
/*
# use a PAR for backend storage
terraform {
backend "http" {
update_method = "PUT"
address       = "https://objectstorage.us-ashburn-1.oraclecloud.com/p/big-par-baseurl-goes-here/n/NAMESPACE/b/BUCKET/o/OBJECTPATH/terraform.tfstate"
}
}
*/
```

## Setting up a VCN

In OCI a VCN is very similar to a VPC in AWS. It's easy to think of as a bag in which all your other related network resources reside. Here's the terraform config for creating a VCN.

```
csuttles@cs-mbp15:[~/src/oci-vault/iad/network]:(master)
[Exit: 0] 12:19: cat vcn.tf
resource "oci_core_virtual_network" "oci-vault-vcn1" {
  cidr_block     = "10.0.0.0/16"
  dns_label      = "ocivault"
  compartment_id = "${data.terraform_remote_state.common.network_compartment}"
  display_name   = "oci-vault-vcn1"
}

resource "oci_core_internet_gateway" "oci-vault-ig1" {
  compartment_id = "${var.compartment_ocid}"
  display_name   = "oci-vault-ig1"
  vcn_id         = "${oci_core_virtual_network.oci-vault-vcn1.id}"
}

resource "oci_core_default_route_table" "oci-vault-default-route-table" {
  manage_default_resource_id = "${oci_core_virtual_network.oci-vault-vcn1.default_route_table_id}"
  display_name               = "oci-vault-default-route-table"

  route_rules {
    destination       = "0.0.0.0/0"
    destination_type  = "CIDR_BLOCK"
    network_entity_id = "${oci_core_internet_gateway.oci-vault-ig1.id}"
  }
}

resource "oci_core_route_table" "oci-vault-route-table1" {
  compartment_id = "${var.compartment_ocid}"
  vcn_id         = "${oci_core_virtual_network.oci-vault-vcn1.id}"
  display_name   = "oci-vault-route-table1"

  route_rules {
    destination       = "0.0.0.0/0"
    destination_type  = "CIDR_BLOCK"
    network_entity_id = "${oci_core_internet_gateway.oci-vault-ig1.id}"
  }
}

resource "oci_core_default_dhcp_options" "oci-vault-default-dhcp-options" {
  manage_default_resource_id = "${oci_core_virtual_network.oci-vault-vcn1.default_dhcp_options_id}"
  display_name               = "oci-vault-default-dhcp-options"

  // required
  options {
    type        = "DomainNameServer"
    server_type = "VcnLocalPlusInternet"
  }

  // optional
  options {
    type                = "SearchDomain"
    search_domain_names = ["csuttles.io"]
  }
}

resource "oci_core_dhcp_options" "oci-vault-dhcp-options1" {
  compartment_id = "${var.compartment_ocid}"
  vcn_id         = "${oci_core_virtual_network.oci-vault-vcn1.id}"
  display_name   = "oci-vault-dhcp-options1"

  // required
  options {
    type        = "DomainNameServer"
    server_type = "VcnLocalPlusInternet"
  }

  // optional
  options {
    type                = "SearchDomain"
    search_domain_names = ["csuttles.io"]
  }
}
```

## Create Subnets and Seclists in the VCN

In OCI, Virtual NICs (VNICs) attach to subnets, which live inside a VCN. The seclist construct is also very similar to AWS.

Here's how that looks in Terraform:

```
csuttles@cs-mbp15:[~/src/oci-vault/iad/network]:(master)
[Exit: 0] 13:27: cat subnets.tf
variable "availability_domain" {
  default = 3
}
/*
Because you can specify multiple security lists/subnet the security_list_ids value must be specified as a list in []'s.
 See https://www.terraform.io/docs/configuration/syntax.html

Generally you wouldn't specify a subnet without first specifying a VCN. Once the VCN has been created you would get the vcn_id, route_table_id, and security_list_id(s) from that resource and use Terraform attributes below to populate those values.
 See https://www.terraform.io/docs/configuration/interpolation.html*/
resource "oci_core_subnet" "oci_vault" {
  count = 3
  //availability_domain = "${lookup(data.oci_identity_availability_domains.ADs.availability_domains[var.availability_domain - 1],"name")}"
  availability_domain = "${lookup(data.oci_identity_availability_domains.ADs.availability_domains[count.index],"name")}"
  cidr_block          = "${format("10.0.%d.0/24", count.index + 1)}"
  display_name        = "${format("oci_vault_subnet_%d", count.index + 1)}"
  compartment_id = "${data.terraform_remote_state.common.network_compartment}"
  vcn_id              = "${oci_core_virtual_network.oci-vault-vcn1.id}"
  security_list_ids   = ["${oci_core_virtual_network.oci-vault-vcn1.default_security_list_id}"]
  route_table_id      = "${oci_core_virtual_network.oci-vault-vcn1.default_route_table_id}"
  dhcp_options_id     = "${oci_core_virtual_network.oci-vault-vcn1.default_dhcp_options_id}"
}

data "oci_identity_availability_domains" "ADs" {
  compartment_id = "${var.tenancy_ocid}"
}

data "oci_core_subnets" "oci_vault_subnets" {
    #Required
    compartment_id = "${data.terraform_remote_state.common.network_compartment}"
    vcn_id              = "${oci_core_virtual_network.oci-vault-vcn1.id}"

    #Optional
/*
    display_name = "${var.subnet_display_name}"
    state = "${var.subnet_state}"
*/
}

output "subnets" {
    value = "${data.oci_core_subnets.oci_vault_subnets.subnets}"
}
```

```
csuttles@cs-mbp15:[~/src/oci-vault/iad/network]:(master)
[Exit: 0] 14:30: cat seclists.tf
resource "oci_core_default_security_list" "oci-vault-default-security-list" {
  manage_default_resource_id = "${oci_core_virtual_network.oci-vault-vcn1.default_security_list_id}"
  display_name               = "oci-vault-default-security-list"

  // allow outbound tcp traffic on all ports
  egress_security_rules {
    destination = "0.0.0.0/0"
    protocol    = "6"
  }

  // allow outbound udp traffic on a port range
  egress_security_rules {
    destination = "0.0.0.0/0"
    protocol    = "17"        // udp
    stateless   = true

    udp_options {
      "min" = 319
      "max" = 320
    }
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
```

## Apply Infrastructure Changes via Terraform

Finally, from the same directory where these resources are defined, we run `terraform apply` and create the resources in the IAD region. The pipe to Perl is just to obfuscate my OCIDs because the internet is a scary place to just leave your data lying around, and is totally unnecessary in the 'real world'.

```
csuttles@cs-mbp15:[~/src/oci-vault/iad/network]:(master)
[Exit: 0] 15:00: tf apply 2>&1 | perl -pe 's/ocid.([\w\.]+)\.(?:[\w]+)([\W\s]*?)$/ocid.$1.07734$2/'
data.terraform_remote_state.common: Refreshing state...
data.oci_identity_availability_domains.ADs: Refreshing state...
data.oci_identity_availability_domains.ads: Refreshing state...

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create
 <= read (data resources)

Terraform will perform the following actions:

 <= data.oci_core_subnets.oci_vault_subnets
      id:                                                                 <computed>
      compartment_id:                                                     "ocid..compartment.oc1..07734"
      subnets.#:                                                          <computed>
      vcn_id:                                                             "${oci_core_virtual_network.oci-vault-vcn1.id}"

  + oci_core_default_dhcp_options.oci-vault-default-dhcp-options
      id:                                                                 <computed>
      display_name:                                                       "oci-vault-default-dhcp-options"
      freeform_tags.%:                                                    <computed>
      manage_default_resource_id:                                         "${oci_core_virtual_network.oci-vault-vcn1.default_dhcp_options_id}"
      options.#:                                                          "2"
      options.0.server_type:                                              "VcnLocalPlusInternet"
      options.0.type:                                                     "DomainNameServer"
      options.1.search_domain_names.#:                                    "1"
      options.1.search_domain_names.0:                                    "csuttles.io"
      options.1.type:                                                     "SearchDomain"
      state:                                                              <computed>
      time_created:                                                       <computed>

  + oci_core_default_route_table.oci-vault-default-route-table
      id:                                                                 <computed>
      display_name:                                                       "oci-vault-default-route-table"
      freeform_tags.%:                                                    <computed>
      manage_default_resource_id:                                         "${oci_core_virtual_network.oci-vault-vcn1.default_route_table_id}"
      route_rules.#:                                                      "1"
      route_rules.~1768267378.cidr_block:                                 <computed>
      route_rules.~1768267378.destination:                                "0.0.0.0/0"
      route_rules.~1768267378.destination_type:                           "CIDR_BLOCK"
      route_rules.~1768267378.network_entity_id:                          "${oci_core_internet_gateway.oci-vault-ig1.id}"
      state:                                                              <computed>
      time_created:                                                       <computed>
      time_modified:                                                      <computed>

  + oci_core_default_security_list.oci-vault-default-security-list
      id:                                                                 <computed>
      display_name:                                                       "oci-vault-default-security-list"
      egress_security_rules.#:                                            "2"
      egress_security_rules.1420396200.destination:                       "0.0.0.0/0"
      egress_security_rules.1420396200.destination_type:                  <computed>
      egress_security_rules.1420396200.icmp_options.#:                    "0"
      egress_security_rules.1420396200.protocol:                          "6"
      egress_security_rules.1420396200.stateless:                         <computed>
      egress_security_rules.1420396200.tcp_options.#:                     "0"
      egress_security_rules.1420396200.udp_options.#:                     "0"
      egress_security_rules.1565661692.destination:                       "0.0.0.0/0"
      egress_security_rules.1565661692.destination_type:                  <computed>
      egress_security_rules.1565661692.icmp_options.#:                    "0"
      egress_security_rules.1565661692.protocol:                          "17"
      egress_security_rules.1565661692.stateless:                         "true"
      egress_security_rules.1565661692.tcp_options.#:                     "0"
      egress_security_rules.1565661692.udp_options.#:                     "1"
      egress_security_rules.1565661692.udp_options.0.max:                 "320"
      egress_security_rules.1565661692.udp_options.0.min:                 "319"
      egress_security_rules.1565661692.udp_options.0.source_port_range.#: "0"
      freeform_tags.%:                                                    <computed>
      ingress_security_rules.#:                                           "2"
      ingress_security_rules.1709695539.icmp_options.#:                   "1"
      ingress_security_rules.1709695539.icmp_options.0.code:              "4"
      ingress_security_rules.1709695539.icmp_options.0.type:              "3"
      ingress_security_rules.1709695539.protocol:                         "1"
      ingress_security_rules.1709695539.source:                           "0.0.0.0/0"
      ingress_security_rules.1709695539.source_type:                      <computed>
      ingress_security_rules.1709695539.stateless:                        "true"
      ingress_security_rules.1709695539.tcp_options.#:                    "0"
      ingress_security_rules.1709695539.udp_options.#:                    "0"
      ingress_security_rules.47193274.icmp_options.#:                     "0"
      ingress_security_rules.47193274.protocol:                           "6"
      ingress_security_rules.47193274.source:                             "0.0.0.0/0"
      ingress_security_rules.47193274.source_type:                        <computed>
      ingress_security_rules.47193274.stateless:                          "false"
      ingress_security_rules.47193274.tcp_options.#:                      "1"
      ingress_security_rules.47193274.tcp_options.0.max:                  "22"
      ingress_security_rules.47193274.tcp_options.0.min:                  "22"
      ingress_security_rules.47193274.tcp_options.0.source_port_range.#:  "0"
      ingress_security_rules.47193274.udp_options.#:                      "0"
      manage_default_resource_id:                                         "${oci_core_virtual_network.oci-vault-vcn1.default_security_list_id}"
      state:                                                              <computed>
      time_created:                                                       <computed>

  + oci_core_dhcp_options.oci-vault-dhcp-options1
      id:                                                                 <computed>
      compartment_id:                                                     "ocid..compartment.oc1..07734"
      display_name:                                                       "oci-vault-dhcp-options1"
      freeform_tags.%:                                                    <computed>
      options.#:                                                          "2"
      options.0.server_type:                                              "VcnLocalPlusInternet"
      options.0.type:                                                     "DomainNameServer"
      options.1.search_domain_names.#:                                    "1"
      options.1.search_domain_names.0:                                    "csuttles.io"
      options.1.type:                                                     "SearchDomain"
      state:                                                              <computed>
      time_created:                                                       <computed>
      vcn_id:                                                             "${oci_core_virtual_network.oci-vault-vcn1.id}"

  + oci_core_internet_gateway.oci-vault-ig1
      id:                                                                 <computed>
      compartment_id:                                                     "ocid..compartment.oc1..07734"
      display_name:                                                       "oci-vault-ig1"
      enabled:                                                            "true"
      freeform_tags.%:                                                    <computed>
      state:                                                              <computed>
      time_created:                                                       <computed>
      time_modified:                                                      <computed>
      vcn_id:                                                             "${oci_core_virtual_network.oci-vault-vcn1.id}"

  + oci_core_route_table.oci-vault-route-table1
      id:                                                                 <computed>
      compartment_id:                                                     "ocid..compartment.oc1..07734"
      display_name:                                                       "oci-vault-route-table1"
      freeform_tags.%:                                                    <computed>
      route_rules.#:                                                      "1"
      route_rules.~1768267378.cidr_block:                                 <computed>
      route_rules.~1768267378.destination:                                "0.0.0.0/0"
      route_rules.~1768267378.destination_type:                           "CIDR_BLOCK"
      route_rules.~1768267378.network_entity_id:                          "${oci_core_internet_gateway.oci-vault-ig1.id}"
      state:                                                              <computed>
      time_created:                                                       <computed>
      time_modified:                                                      <computed>
      vcn_id:                                                             "${oci_core_virtual_network.oci-vault-vcn1.id}"

  + oci_core_subnet.oci_vault[0]
      id:                                                                 <computed>
      availability_domain:                                                "bwOa:US-ASHBURN-AD-1"
      cidr_block:                                                         "10.0.1.0/24"
      compartment_id:                                                     "ocid..compartment.oc1..07734"
      dhcp_options_id:                                                    "${oci_core_virtual_network.oci-vault-vcn1.default_dhcp_options_id}"
      display_name:                                                       "oci_vault_subnet_1"
      freeform_tags.%:                                                    <computed>
      prohibit_public_ip_on_vnic:                                         <computed>
      route_table_id:                                                     "${oci_core_virtual_network.oci-vault-vcn1.default_route_table_id}"
      security_list_ids.#:                                                <computed>
      state:                                                              <computed>
      subnet_domain_name:                                                 <computed>
      time_created:                                                       <computed>
      vcn_id:                                                             "${oci_core_virtual_network.oci-vault-vcn1.id}"
      virtual_router_ip:                                                  <computed>
      virtual_router_mac:                                                 <computed>

  + oci_core_subnet.oci_vault[1]
      id:                                                                 <computed>
      availability_domain:                                                "bwOa:US-ASHBURN-AD-2"
      cidr_block:                                                         "10.0.2.0/24"
      compartment_id:                                                     "ocid..compartment.oc1..07734"
      dhcp_options_id:                                                    "${oci_core_virtual_network.oci-vault-vcn1.default_dhcp_options_id}"
      display_name:                                                       "oci_vault_subnet_2"
      freeform_tags.%:                                                    <computed>
      prohibit_public_ip_on_vnic:                                         <computed>
      route_table_id:                                                     "${oci_core_virtual_network.oci-vault-vcn1.default_route_table_id}"
      security_list_ids.#:                                                <computed>
      state:                                                              <computed>
      subnet_domain_name:                                                 <computed>
      time_created:                                                       <computed>
      vcn_id:                                                             "${oci_core_virtual_network.oci-vault-vcn1.id}"
      virtual_router_ip:                                                  <computed>
      virtual_router_mac:                                                 <computed>

  + oci_core_subnet.oci_vault[2]
      id:                                                                 <computed>
      availability_domain:                                                "bwOa:US-ASHBURN-AD-3"
      cidr_block:                                                         "10.0.3.0/24"
      compartment_id:                                                     "ocid..compartment.oc1..07734"
      dhcp_options_id:                                                    "${oci_core_virtual_network.oci-vault-vcn1.default_dhcp_options_id}"
      display_name:                                                       "oci_vault_subnet_3"
      freeform_tags.%:                                                    <computed>
      prohibit_public_ip_on_vnic:                                         <computed>
      route_table_id:                                                     "${oci_core_virtual_network.oci-vault-vcn1.default_route_table_id}"
      security_list_ids.#:                                                <computed>
      state:                                                              <computed>
      subnet_domain_name:                                                 <computed>
      time_created:                                                       <computed>
      vcn_id:                                                             "${oci_core_virtual_network.oci-vault-vcn1.id}"
      virtual_router_ip:                                                  <computed>
      virtual_router_mac:                                                 <computed>

  + oci_core_virtual_network.oci-vault-vcn1
      id:                                                                 <computed>
      cidr_block:                                                         "10.0.0.0/16"
      compartment_id:                                                     "ocid..compartment.oc1..07734"
      default_dhcp_options_id:                                            <computed>
      default_route_table_id:                                             <computed>
      default_security_list_id:                                           <computed>
      display_name:                                                       "oci-vault-vcn1"
      dns_label:                                                          "ocivault"
      freeform_tags.%:                                                    <computed>
      state:                                                              <computed>
      time_created:                                                       <computed>
      vcn_domain_name:                                                    <computed>


Plan: 10 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

yes
```
...

```
Apply complete! Resources: 10 added, 0 changed, 0 destroyed.

Outputs:

show-ads = [
    {
        compartment_id = ocid..tenancy.oc1..07734,
        name = bwOa:US-ASHBURN-AD-1
    },
    {
        compartment_id = ocid..tenancy.oc1..07734,
        name = bwOa:US-ASHBURN-AD-2
    },
    {
        compartment_id = ocid..tenancy.oc1..07734,
        name = bwOa:US-ASHBURN-AD-3
    }
]
subnets = [
    {
        availability_domain = bwOa:US-ASHBURN-AD-1,
        cidr_block = 10.0.1.0/24,
        compartment_id = ocid..compartment.oc1..07734,
        defined_tags = map[],
        dhcp_options_id = ocid..dhcpoptions.oc1.iad.07734,
        display_name = oci_vault_subnet_1,
        dns_label = ,
        freeform_tags = map[],
        id = ocid..subnet.oc1.iad.07734,
        prohibit_public_ip_on_vnic = 0,
        route_table_id = ocid..routetable.oc1.iad.07734,
        security_list_ids = [ocid..securitylist.oc1.iad.07734],
        state = AVAILABLE,
        subnet_domain_name = ,
        time_created = 2018-10-27 16:28:17.417 +0000 UTC,
        vcn_id = ocid..vcn.oc1.iad.07734,
        virtual_router_ip = 10.0.1.1,
        virtual_router_mac = 00:00:17:CF:08:33
    },
    {
        availability_domain = bwOa:US-ASHBURN-AD-2,
        cidr_block = 10.0.2.0/24,
        compartment_id = ocid..compartment.oc1..07734,
        defined_tags = map[],
        dhcp_options_id = ocid..dhcpoptions.oc1.iad.07734,
        display_name = oci_vault_subnet_2,
        dns_label = ,
        freeform_tags = map[],
        id = ocid..subnet.oc1.iad.07734,
        prohibit_public_ip_on_vnic = 0,
        route_table_id = ocid..routetable.oc1.iad.07734,
        security_list_ids = [ocid..securitylist.oc1.iad.07734],
        state = AVAILABLE,
        subnet_domain_name = ,
        time_created = 2018-10-27 16:28:16.55 +0000 UTC,
        vcn_id = ocid..vcn.oc1.iad.07734,
        virtual_router_ip = 10.0.2.1,
        virtual_router_mac = 00:00:17:CF:08:33
    },
    {
        availability_domain = bwOa:US-ASHBURN-AD-3,
        cidr_block = 10.0.3.0/24,
        compartment_id = ocid..compartment.oc1..07734,
        defined_tags = map[],
        dhcp_options_id = ocid..dhcpoptions.oc1.iad.07734,
        display_name = oci_vault_subnet_3,
        dns_label = ,
        freeform_tags = map[],
        id = ocid..subnet.oc1.iad.07734,
        prohibit_public_ip_on_vnic = 0,
        route_table_id = ocid..routetable.oc1.iad.07734,
        security_list_ids = [ocid..securitylist.oc1.iad.07734],
        state = AVAILABLE,
        subnet_domain_name = ,
        time_created = 2018-10-27 16:28:15.494 +0000 UTC,
        vcn_id = ocid..vcn.oc1.iad.07734,
        virtual_router_ip = 10.0.3.1,
        virtual_router_mac = 00:00:17:CF:08:33
    }
]
```

## Summary

This is one more milestone toward a working, automated, and easily repeatable deployment of Hashicorp Vault in OCI. In the next installment, we will get to provisioning the instances and the application, using the resources we created earlier in this series.

