## Description

> [!NOTE]
> Slurm-gcp-v5-login module is deprecated. See
> [this update](../../../../examples/README.md#completed-migration-to-slurm-gcp-v6)
> for specific recommendations and timelines.

This module creates a login node for a Slurm cluster based on the
[SchedMD/slurm-gcp] [slurm\_instance\_template] and [slurm\_login\_instance]
terraform modules. The login node is used in conjunction with the
[Slurm controller](../schedmd-slurm-gcp-v5-controller/README.md).

[SchedMD/slurm-gcp]: https://github.com/GoogleCloudPlatform/slurm-gcp/tree/5.12.0
[slurm\_login\_instance]: https://github.com/GoogleCloudPlatform/slurm-gcp/tree/5.12.0/terraform/slurm_cluster/modules/slurm_login_instance
[slurm\_instance\_template]: https://github.com/GoogleCloudPlatform/slurm-gcp/tree/5.12.0/terraform/slurm_cluster/modules/slurm_instance_template

### Example

```yaml
- id: slurm_login
  source: community/modules/scheduler/schedmd-slurm-gcp-v5-login
  use:
  - network1
  - slurm_controller
  settings:
    machine_type: n2-standard-4
```

This creates a Slurm login node which is:

* connected to the primary subnet of network1 via `use`
* associated with the `slurm_controller` module as the slurm controller via
  `use`
* of VM machine type `n2-standard-4`

## Custom Images

For more information on creating valid custom images for the login node VM
instances or for custom instance templates, see our [vm-images.md] documentation
page.

[vm-images.md]: ../../../../docs/vm-images.md#slurm-on-gcp-custom-images

## GPU Support

More information on GPU support in Slurm on GCP and other Cluster Toolkit modules
can be found at [docs/gpu-support.md](../../../../docs/gpu-support.md)

## Support
The Cluster Toolkit team maintains the wrapper around the [slurm-on-gcp] terraform
modules. For support with the underlying modules, see the instructions in the
[slurm-gcp README][slurm-gcp-readme].

[slurm-on-gcp]: https://github.com/GoogleCloudPlatform/slurm-gcp/tree/5.12.0
[slurm-gcp-readme]: https://github.com/GoogleCloudPlatform/slurm-gcp/tree/5.12.0#slurm-on-google-cloud-platform

## License
<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| access\_config | Access configurations, i.e. IPs via which the VM instance can be accessed via the Internet. | <pre>list(object({<br>    nat_ip       = string<br>    network_tier = string<br>  }))</pre> | `[]` | no |
| additional\_disks | List of maps of disks. | <pre>list(object({<br>    disk_name    = string<br>    device_name  = string<br>    disk_type    = string<br>    disk_size_gb = number<br>    disk_labels  = map(string)<br>    auto_delete  = bool<br>    boot         = bool<br>  }))</pre> | `[]` | no |
| allow\_automatic\_updates | If false, disables automatic system package updates on the created instances.  This feature is<br>only available on supported images (or images derived from them).  For more details, see<br>https://cloud.google.com/compute/docs/instances/create-hpc-vm#disable_automatic_updates | `bool` | `true` | no |
| can\_ip\_forward | Enable IP forwarding, for NAT instances for example. | `bool` | `false` | no |
| controller\_instance\_id | The server-assigned unique identifier of the controller instance. This value<br>must be supplied as an output of the controller module, typically via `use`. | `string` | n/a | yes |
| deployment\_name | Name of the deployment. | `string` | n/a | yes |
| disable\_login\_public\_ips | If set to false. The login will have a random public IP assigned to it. Ignored if access\_config is set. | `bool` | `true` | no |
| disable\_smt | Disables Simultaneous Multi-Threading (SMT) on instance. | `bool` | `true` | no |
| disk\_auto\_delete | Whether or not the boot disk should be auto-deleted. | `bool` | `true` | no |
| disk\_labels | Labels specific to the boot disk. These will be merged with var.labels. | `map(string)` | `{}` | no |
| disk\_size\_gb | Boot disk size in GB. | `number` | `50` | no |
| disk\_type | Boot disk type. | `string` | `"pd-standard"` | no |
| enable\_confidential\_vm | Enable the Confidential VM configuration. Note: the instance image must support option. | `bool` | `false` | no |
| enable\_oslogin | Enables Google Cloud os-login for user login and authentication for VMs.<br>See https://cloud.google.com/compute/docs/oslogin | `bool` | `true` | no |
| enable\_reconfigure | Enables automatic Slurm reconfigure on when Slurm configuration changes (e.g.<br>slurm.conf.tpl, partition details).<br><br>NOTE: Requires Google Pub/Sub API. | `bool` | `false` | no |
| enable\_shielded\_vm | Enable the Shielded VM configuration. Note: the instance image must support option. | `bool` | `false` | no |
| gpu | DEPRECATED: use var.guest\_accelerator | <pre>object({<br>    type  = string<br>    count = number<br>  })</pre> | `null` | no |
| guest\_accelerator | List of the type and count of accelerator cards attached to the instance. | <pre>list(object({<br>    type  = string,<br>    count = number<br>  }))</pre> | `[]` | no |
| instance\_image | Defines the image that will be used in the Slurm login node VM instances.<br><br>Expected Fields:<br>name: The name of the image. Mutually exclusive with family.<br>family: The image family to use. Mutually exclusive with name.<br>project: The project where the image is hosted.<br><br>For more information on creating custom images that comply with Slurm on GCP<br>see the "Slurm on GCP Custom Images" section in docs/vm-images.md. | `map(string)` | <pre>{<br>  "family": "slurm-gcp-5-12-hpc-centos-7",<br>  "project": "schedmd-slurm-public"<br>}</pre> | no |
| instance\_image\_custom | A flag that designates that the user is aware that they are requesting<br>to use a custom and potentially incompatible image for this Slurm on<br>GCP module.<br><br>If the field is set to false, only the compatible families and project<br>names will be accepted.  The deployment will fail with any other image<br>family or name.  If set to true, no checks will be done.<br><br>See: https://goo.gle/hpc-slurm-images | `bool` | `false` | no |
| instance\_template | Self link to a custom instance template. If set, other VM definition<br>variables such as machine\_type and instance\_image will be ignored in favor<br>of the provided instance template.<br><br>For more information on creating custom images for the instance template<br>that comply with Slurm on GCP see the "Slurm on GCP Custom Images" section<br>in docs/vm-images.md. | `string` | `null` | no |
| labels | Labels, provided as a map. | `map(string)` | `{}` | no |
| machine\_type | Machine type to create. | `string` | `"n2-standard-2"` | no |
| metadata | Metadata, provided as a map. | `map(string)` | `{}` | no |
| min\_cpu\_platform | Specifies a minimum CPU platform. Applicable values are the friendly names of<br>CPU platforms, such as Intel Haswell or Intel Skylake. See the complete list:<br>https://cloud.google.com/compute/docs/instances/specify-min-cpu-platform | `string` | `null` | no |
| network\_ip | DEPRECATED: Use `static_ips` variable to assign an internal static ip address. | `string` | `null` | no |
| network\_self\_link | Network to deploy to. Either network\_self\_link or subnetwork\_self\_link must be specified. | `string` | `null` | no |
| num\_instances | Number of instances to create. This value is ignored if static\_ips is provided. | `number` | `1` | no |
| on\_host\_maintenance | Instance availability Policy. | `string` | `"MIGRATE"` | no |
| preemptible | Allow the instance to be preempted. | `bool` | `false` | no |
| project\_id | Project ID to create resources in. | `string` | n/a | yes |
| pubsub\_topic | The cluster pubsub topic created by the controller when enable\_reconfigure=true. | `string` | `null` | no |
| region | Region where the instances should be created.<br>Note: region will be ignored if it can be extracted from subnetwork. | `string` | `null` | no |
| service\_account | Service account to attach to the login instance. If not set, the<br>default compute service account for the given project will be used with the<br>"https://www.googleapis.com/auth/cloud-platform" scope. | <pre>object({<br>    email  = string<br>    scopes = set(string)<br>  })</pre> | `null` | no |
| shielded\_instance\_config | Shielded VM configuration for the instance. Note: not used unless<br>enable\_shielded\_vm is 'true'.<br>- enable\_integrity\_monitoring : Compare the most recent boot measurements to the<br>  integrity policy baseline and return a pair of pass/fail results depending on<br>  whether they match or not.<br>- enable\_secure\_boot : Verify the digital signature of all boot components, and<br>  halt the boot process if signature verification fails.<br>- enable\_vtpm : Use a virtualized trusted platform module, which is a<br>  specialized computer chip you can use to encrypt objects like keys and<br>  certificates. | <pre>object({<br>    enable_integrity_monitoring = bool<br>    enable_secure_boot          = bool<br>    enable_vtpm                 = bool<br>  })</pre> | <pre>{<br>  "enable_integrity_monitoring": true,<br>  "enable_secure_boot": true,<br>  "enable_vtpm": true<br>}</pre> | no |
| slurm\_cluster\_name | Cluster name, used for resource naming and slurm accounting. If not provided it will default to the first 8 characters of the deployment name (removing any invalid characters). | `string` | `null` | no |
| source\_image | DEPRECATED: Use `instance_image` instead. | `string` | `null` | no |
| source\_image\_family | DEPRECATED: Use `instance_image` instead. | `string` | `null` | no |
| source\_image\_project | DEPRECATED: Use `instance_image` instead. | `string` | `null` | no |
| startup\_script | Startup script that will be used by the login node VM. | `string` | `""` | no |
| static\_ips | List of static IPs for VM instances. | `list(string)` | `[]` | no |
| subnetwork\_project | The project that subnetwork belongs to. | `string` | `null` | no |
| subnetwork\_self\_link | Subnet to deploy to. Either network\_self\_link or subnetwork\_self\_link must be specified. | `string` | `null` | no |
| tags | Network tag list. | `list(string)` | `[]` | no |
| zone | Zone where the instances should be created. If not specified, instances will be<br>spread across available zones in the region. | `string` | `null` | no |

## Outputs

No outputs.

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
