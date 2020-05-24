+++
author = "Chris Suttles"
categories = ["cloud", "OCI", "Getting Started", "terraform"]
date = 2018-10-10T01:18:51Z
description = ""
draft = false
cover = "/images/2018/10/Screen-Shot-2018-10-09-at-1.17.31-PM.png"
slug = "getting-started-with-terraform-on-oracle-cloud-infrastructure-oci"
tags = ["cloud", "OCI", "Getting Started", "terraform"]
title = "Getting Started with Terraform on Oracle Cloud Infrastructure (OCI)"

+++


Getting started with the [OCI terraform provider](https://www.terraform.io/docs/providers/oci/index.html) is easy, particularly since it became an official terraform provider. The [github docs](https://github.com/terraform-providers/terraform-provider-oci/blob/master/docs/Table of Contents.md) [] are also a good resource, with a lot of examples in the repo.

## Layout the Repository

I like to re-use a pattern that I learned from some of my very talented colleagues. It looks like this:

```
csuttles@cs-mbp15:[~/src/oci-vault]:(master)
[Exit: 0 0] 12:46: find . -type d | grep -vP '.git|.terr'
.
./phx
./phx/network
./phx/vault
./common
./iad
./iad/network
./iad/vault
./modules
```

To summarize, the base of the repo is the tenancy. The common directory is for resources that are global (IAM). The modules dir is of course for modules, and the phx and iad dirs are regions. Beneath each region, there is a subdirectory per compartment. In this example, there are only two: network, and vault.It's also possible to slightly modify this and use the tenancy and modules as first order directories to support multi-tenancy in a single repo. That looks more like this:

```
csuttles@cs-mbp15:[~/src/oci-vault-multitenant]:(master)
[Exit: 0 0] 12:53: find . -type d | grep -vP '.git|.terr'
.
./tenancy1
./tenancy1/phx
./tenancy1/phx/network
./tenancy1/phx/vault
./tenancy1/common
./tenancy1/iad
./tenancy1/iad/network
./tenancy1/iad/vault
./tenancy3
./tenancy3/phx
./tenancy3/phx/network
./tenancy3/phx/vault
./tenancy3/common
./tenancy3/iad
./tenancy3/iad/network
./tenancy3/iad/vault
./tenancy2
./tenancy2/phx
./tenancy2/phx/network
./tenancy2/phx/vault
./tenancy2/common
./tenancy2/iad
./tenancy2/iad/network
./tenancy2/iad/vault
./modules
```

## Define the Provider and Storage Backend

Within each directory with configs, you must define the provider and (optionally) a storage backend. My provider configuration looks like this, and is part of my `variables.tf` in the common directory:

```
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

I also store terraform state in OCI, using the Object Storage S3 compatible API, as described [here](https://medium.com/oracledevs/storing-terraform-remote-state-to-oracle-cloud-infrastructure-object-storage-b32fe7402781). Here's an example of two ways to do it:

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

All of the variables that this depends on are defined as environment vars in a separate file that I source before running terraform. That looks like this:

```
csuttles@cs-mbp15:[~/src/oci-vault/common]:(master)
[Exit: 0] 13:09: cat ~/.oci/oci-vault.sh
#!/bin/bash

export TF_VAR_tenancy="TENANCY_NAME"
export TF_VAR_tenancy_ocid="ocid1.tenancy.oc1..long-string-of-ocid"
export TF_VAR_compartment_ocid="ocid1.tenancy.oc1..long-string-of-ocid"
export TF_VAR_user_ocid="ocid1.user.oc1..long-string-of-ocid"
export TF_VAR_fingerprint="de:ad:be:ef:ca:fe:d0:0d:00:ca:b0:b0:b0:de:ad:be:ef"
export TF_VAR_private_key_path="${HOME}/.oci/your_actual_oci_api_key.pem"
```

## Define Resources

Let's dig into the resource definitions in the common directory, since IAM is the foundation of everything, and those resources are available globally.

```
csuttles@cs-mbp15:[~/src/oci-vault/common]:(master)
[Exit: 0] 12:55: cat compartments.tf
resource "oci_identity_compartment" "network_compartment" {
    #Required
    description = "network compartment"
    name = "network_compartment"

    #Optional
    //defined_tags = {"Operations.CostCenter"= "42"}
    freeform_tags = {"app"="vault"}
}

resource "oci_identity_compartment" "vault_compartment" {
    #Required
    description = "vault compartment"
    name = "vault_compartment"

    #Optional
    //defined_tags = {"Operations.CostCenter"= "42"}
    freeform_tags = {"app"="vault"}
}
data "oci_identity_compartments" "compartments" {
  compartment_id = "${oci_identity_compartment.network_compartment.compartment_id}"
/*
  filter {
    name   = "name"
    values = ["tf-example-compartment"]
  }
*/
}

data "oci_identity_compartments" "network_compartment" {
    #Required
    compartment_id = "${oci_identity_compartment.network_compartment.compartment_id}"

    filter {
        name   = "name"
        values  = [ "network_compartment" ]
    }
}

data "oci_identity_compartments" "vault_compartment" {
    #Required
    compartment_id = "${oci_identity_compartment.vault_compartment.compartment_id}"
    filter {
        name   = "name"
        values  = [ "vault_compartment" ]
    }
}
output "network_compartment" {
  value = "${oci_identity_compartment.network_compartment.id}"
}
output "vault_compartment" {
  value = "${oci_identity_compartment.vault_compartment.id}"
}
```

In this file I am simply defining two compartments, which we will use for organization of resources and policy later.

## Summary

This is just the beginning, and should get you as far as a `terraform init && terraform plan`. I plan to continue this post as a series, where we will explore building out the [Hashicorp Vault reference architecture for a single DC](https://www.vaultproject.io/guides/operations/reference-architecture.html#one-dc).

