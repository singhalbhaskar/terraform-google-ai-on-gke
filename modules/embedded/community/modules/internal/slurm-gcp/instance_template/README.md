# terraform-google-ai-on-gke

<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| access\_config | Access configurations, i.e. IPs via which the VM instance can be accessed via the Internet. | <pre>list(object({<br>    nat_ip       = string<br>    network_tier = string<br>  }))</pre> | `[]` | no |
| additional\_disks | List of maps of disks. | <pre>list(object({<br>    disk_name    = string<br>    device_name  = string<br>    disk_type    = string<br>    disk_size_gb = number<br>    disk_labels  = map(string)<br>    auto_delete  = bool<br>    boot         = bool<br>  }))</pre> | `[]` | no |
| additional\_networks | Additional network interface details for GCE, if any. | <pre>list(object({<br>    network            = string<br>    subnetwork         = string<br>    subnetwork_project = string<br>    network_ip         = string<br>    nic_type           = string<br>    access_config = list(object({<br>      nat_ip       = string<br>      network_tier = string<br>    }))<br>    ipv6_access_config = list(object({<br>      network_tier = string<br>    }))<br>  }))</pre> | `[]` | no |
| bandwidth\_tier | Tier 1 bandwidth increases the maximum egress bandwidth for VMs.<br>Using the `virtio_enabled` setting will only enable VirtioNet and will not enable TIER\_1.<br>Using the `tier_1_enabled` setting will enable both gVNIC and TIER\_1 higher bandwidth networking.<br>Using the `gvnic_enabled` setting will only enable gVNIC and will not enable TIER\_1.<br>Note that TIER\_1 only works with specific machine families & shapes and must be using an image that supports gVNIC. See [official docs](https://cloud.google.com/compute/docs/networking/configure-vm-with-high-bandwidth-configuration) for more details. | `string` | `"platform_default"` | no |
| can\_ip\_forward | Enable IP forwarding, for NAT instances for example. | `bool` | `false` | no |
| disable\_smt | Disables Simultaneous Multi-Threading (SMT) on instance. | `bool` | `false` | no |
| disk\_auto\_delete | Whether or not the boot disk should be auto-deleted. | `bool` | `true` | no |
| disk\_labels | Labels to be assigned to boot disk, provided as a map. | `map(string)` | `{}` | no |
| disk\_size\_gb | Boot disk size in GB. | `number` | `100` | no |
| disk\_type | Boot disk type, can be either pd-ssd, local-ssd, or pd-standard. | `string` | `"pd-standard"` | no |
| enable\_confidential\_vm | Enable the Confidential VM configuration. Note: the instance image must support option. | `bool` | `false` | no |
| enable\_oslogin | Enables Google Cloud os-login for user login and authentication for VMs.<br>See https://cloud.google.com/compute/docs/oslogin | `bool` | `true` | no |
| enable\_shielded\_vm | Enable the Shielded VM configuration. Note: the instance image must support option. | `bool` | `false` | no |
| gpu | GPU information. Type and count of GPU to attach to the instance template. See<br>https://cloud.google.com/compute/docs/gpus more details.<br>- type : the GPU type<br>- count : number of GPUs | <pre>object({<br>    type  = string<br>    count = number<br>  })</pre> | `null` | no |
| labels | Labels, provided as a map | `map(string)` | `{}` | no |
| machine\_type | Machine type to create. | `string` | `"n1-standard-1"` | no |
| metadata | Metadata, provided as a map. | `map(string)` | `{}` | no |
| min\_cpu\_platform | Specifies a minimum CPU platform. Applicable values are the friendly names of<br>CPU platforms, such as Intel Haswell or Intel Skylake. See the complete list:<br>https://cloud.google.com/compute/docs/instances/specify-min-cpu-platform | `string` | `null` | no |
| name\_prefix | Prefix for template resource. | `string` | `"default"` | no |
| network | The name or self\_link of the network to attach this interface to. Use network<br>attribute for Legacy or Auto subnetted networks and subnetwork for custom<br>subnetted networks. | `string` | `null` | no |
| network\_ip | Private IP address to assign to the instance if desired. | `string` | `""` | no |
| on\_host\_maintenance | Instance availability Policy | `string` | `"MIGRATE"` | no |
| preemptible | Allow the instance to be preempted. | `bool` | `false` | no |
| project\_id | Project ID to create resources in. | `string` | n/a | yes |
| region | Region where the instance template should be created. | `string` | `null` | no |
| service\_account | Service account to attach to the instances. See<br>'main.tf:local.service\_account' for the default. | <pre>object({<br>    email  = string<br>    scopes = set(string)<br>  })</pre> | `null` | no |
| shielded\_instance\_config | Shielded VM configuration for the instance. Note: not used unless<br>enable\_shielded\_vm is 'true'.<br>- enable\_integrity\_monitoring : Compare the most recent boot measurements to the<br>  integrity policy baseline and return a pair of pass/fail results depending on<br>  whether they match or not.<br>- enable\_secure\_boot : Verify the digital signature of all boot components, and<br>  halt the boot process if signature verification fails.<br>- enable\_vtpm : Use a virtualized trusted platform module, which is a<br>  specialized computer chip you can use to encrypt objects like keys and<br>  certificates. | <pre>object({<br>    enable_integrity_monitoring = bool<br>    enable_secure_boot          = bool<br>    enable_vtpm                 = bool<br>  })</pre> | <pre>{<br>  "enable_integrity_monitoring": true,<br>  "enable_secure_boot": true,<br>  "enable_vtpm": true<br>}</pre> | no |
| slurm\_bucket\_path | GCS Bucket URI of Slurm cluster file storage. | `string` | n/a | yes |
| slurm\_cluster\_name | Cluster name, used for resource naming. | `string` | n/a | yes |
| slurm\_instance\_role | Slurm instance type. Must be one of: controller; login; compute; or null. | `string` | n/a | yes |
| source\_image | Source disk image. | `string` | `""` | no |
| source\_image\_family | Source image family. | `string` | `""` | no |
| source\_image\_project | Project where the source image comes from. If it is not provided, the provider project is used. | `string` | `""` | no |
| spot | Provision as a SPOT preemptible instance.<br>See https://cloud.google.com/compute/docs/instances/spot for more details. | `bool` | `false` | no |
| subnetwork | The name of the subnetwork to attach this interface to. The subnetwork must<br>exist in the same region this instance will be created in. Either network or<br>subnetwork must be provided. | `string` | `null` | no |
| subnetwork\_project | The ID of the project in which the subnetwork belongs. If it is not provided, the provider project is used. | `string` | `null` | no |
| tags | Network tag list. | `list(string)` | `[]` | no |
| termination\_action | Which action to take when Compute Engine preempts the VM. Value can be: 'STOP', 'DELETE'. The default value is 'STOP'.<br>See https://cloud.google.com/compute/docs/instances/spot for more details. | `string` | `"STOP"` | no |

## Outputs

| Name | Description |
|------|-------------|
| instance\_template | Instance template details |
| name | Name of instance template |
| self\_link | Self\_link of instance template |
| service\_account | Service account object, includes email and scopes. |
| tags | Tags that will be associated with instance(s) |

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
