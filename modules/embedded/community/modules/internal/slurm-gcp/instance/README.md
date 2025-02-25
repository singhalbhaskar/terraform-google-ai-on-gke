# terraform-google-ai-on-gke

# Module: Slurm Instance

<!-- mdformat-toc start --slug=github --no-anchors --maxlevel=6 --minlevel=1 -->

- [Module: Slurm Instance](#module-slurm-instance)
  - [Overview](#overview)
  - [Module API](#module-api)

<!-- mdformat-toc end -->

## Overview

This module creates a [compute instance](../../../../docs/glossary.md#vm) from
[instance template](../../../../docs/glossary.md#instance-template) for a
[Slurm cluster](../slurm_cluster/README.md).

> **NOTE:** This module is only intended to be used by Slurm modules. For
> general usage, please consider using:
>
> - [terraform-google-modules/vm/google//modules/compute_instance](https://registry.terraform.io/modules/terraform-google-modules/vm/google/latest/submodules/compute_instance).
> **WARNING:** The source image is not modified. Make sure to use a compatible
> source image.

## Module API

For the terraform module API reference, please see
[README_TF.md](./README_TF.md).

<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| access\_config | Access configurations, i.e. IPs via which the VM instance can be accessed via the Internet. | <pre>list(object({<br>    nat_ip       = string<br>    network_tier = string<br>  }))</pre> | `[]` | no |
| additional\_networks | Additional network interface details for GCE, if any. | <pre>list(object({<br>    access_config = optional(list(object({<br>      nat_ip       = string<br>      network_tier = string<br>    })), [])<br>    alias_ip_range = optional(list(object({<br>      ip_cidr_range         = string<br>      subnetwork_range_name = string<br>    })), [])<br>    ipv6_access_config = optional(list(object({<br>      network_tier = string<br>    })), [])<br>    network            = optional(string)<br>    network_ip         = optional(string, "")<br>    nic_type           = optional(string)<br>    queue_count        = optional(number)<br>    stack_type         = optional(string)<br>    subnetwork         = optional(string)<br>    subnetwork_project = optional(string)<br>  }))</pre> | `[]` | no |
| hostname | Hostname of instances | `string` | n/a | yes |
| instance\_template | Instance template self\_link used to create compute instances | `string` | n/a | yes |
| network | Network to deploy to. Only one of network or subnetwork should be specified. | `string` | `""` | no |
| num\_instances | Number of instances to create. This value is ignored if static\_ips is provided. | `number` | `1` | no |
| project\_id | The GCP project ID | `string` | `null` | no |
| region | Region where the instances should be created. | `string` | `null` | no |
| replace\_trigger | Trigger value to replace the instances. | `string` | `""` | no |
| static\_ips | List of static IPs for VM instances | `list(string)` | `[]` | no |
| subnetwork | Subnet to deploy to. Only one of network or subnetwork should be specified. | `string` | `""` | no |
| subnetwork\_project | The project that subnetwork belongs to | `string` | `null` | no |
| zone | Zone where the instances should be created. If not specified, instances will be spread across available zones in the region. | `string` | `null` | no |

## Outputs

| Name | Description |
|------|-------------|
| available\_zones | List of available zones in region |
| instances\_details | List of all details for compute instances |
| instances\_self\_links | List of self-links for compute instances |
| names | List of available zones in region |
| slurm\_instances | List of all resource objects for compute instances |

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
