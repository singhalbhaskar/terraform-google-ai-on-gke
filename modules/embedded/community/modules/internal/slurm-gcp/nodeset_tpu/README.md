# terraform-google-ai-on-gke

# Module: Slurm Nodeset (TPU)

[FAQ](../../../../docs/faq.md) |
[Troubleshooting](../../../../docs/troubleshooting.md) |
[Glossary](../../../../docs/glossary.md)

<!-- mdformat-toc start --slug=github --no-anchors --maxlevel=6 --minlevel=1 -->

- [Module: Slurm Nodeset (TPU)](#module-slurm-nodeset-tpu)
  - [Overview](#overview)
  - [Module API](#module-api)

<!-- mdformat-toc end -->

## Overview

This is a submodule of [slurm_cluster](../../../slurm_cluster/README.md). It
creates a Slurm TPU nodeset for [slurm_partition](../slurm_partition/README.md).

## Module API

For the terraform module API reference, please see
[README_TF.md](./README_TF.md).

<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| accelerator\_config | Nodeset accelerator config, see https://cloud.google.com/tpu/docs/supported-tpu-configurations for details. | <pre>object({<br>    topology = string<br>    version  = string<br>  })</pre> | <pre>{<br>  "topology": "",<br>  "version": ""<br>}</pre> | no |
| data\_disks | The data disks to include in the TPU node | `list(string)` | `[]` | no |
| docker\_image | The gcp container registry id docker image to use in the TPU vms, it defaults to gcr.io/schedmd-slurm-public/tpu:slurm-gcp-6-8-tf-<var.tf\_version> | `string` | `""` | no |
| enable\_public\_ip | Enables IP address to access the Internet. | `bool` | `false` | no |
| network\_storage | An array of network attached storage mounts to be configured on nodes. | <pre>list(object({<br>    server_ip     = string,<br>    remote_mount  = string,<br>    local_mount   = string,<br>    fs_type       = string,<br>    mount_options = string,<br>  }))</pre> | `[]` | no |
| node\_count\_dynamic\_max | Maximum number of nodes allowed in this partition to be created dynamically. | `number` | `0` | no |
| node\_count\_static | Number of nodes to be statically created. | `number` | `0` | no |
| node\_type | Specify a node type to base the vm configuration upon it. Not needed if you use accelerator\_config | `string` | `null` | no |
| nodeset\_name | Name of Slurm nodeset. | `string` | n/a | yes |
| preemptible | Specify whether TPU-vms in this nodeset are preemtible, see https://cloud.google.com/tpu/docs/preemptible for details. | `bool` | `false` | no |
| preserve\_tpu | Specify whether TPU-vms will get preserve on suspend, if set to true, on suspend vm is stopped, on false it gets deleted | `bool` | `true` | no |
| project\_id | Project ID to create resources in. | `string` | n/a | yes |
| reserved | Specify whether TPU-vms in this nodeset are created under a reservation. | `bool` | `false` | no |
| service\_account | Service account to attach to the TPU-vm.<br>If none is given, the default service account and scopes will be used. | <pre>object({<br>    email  = string<br>    scopes = set(string)<br>  })</pre> | `null` | no |
| subnetwork | The name of the subnetwork to attach the TPU-vm of this nodeset to. | `string` | n/a | yes |
| tf\_version | Nodeset Tensorflow version, see https://cloud.google.com/tpu/docs/supported-tpu-configurations#tpu_vm for details. | `string` | n/a | yes |
| zone | Nodes will only be created in this zone. Check https://cloud.google.com/tpu/docs/regions-zones to get zones with TPU-vm in it. | `string` | n/a | yes |

## Outputs

| Name | Description |
|------|-------------|
| nodeset | Nodeset details. |
| nodeset\_name | Nodeset name. |
| service\_account | Service account object, includes email and scopes. |

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
