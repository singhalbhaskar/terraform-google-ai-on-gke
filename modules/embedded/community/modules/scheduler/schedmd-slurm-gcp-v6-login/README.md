# terraform-google-ai-on-gke

## Description

This module creates a login node for a Slurm cluster based on the
[slurm-gcp] [slurm\_instance\_template] and [slurm\_login\_instance]
terraform modules. The login node is used in conjunction with the
[Slurm controller](../schedmd-slurm-gcp-v6-controller/README.md).

[slurm-gcp]: https://github.com/GoogleCloudPlatform/slurm-gcp/tree/6.8.6
[slurm\_login\_instance]: https://github.com/GoogleCloudPlatform/slurm-gcp/tree/6.8.6/terraform/slurm_cluster/modules/slurm_login_instance
[slurm\_instance\_template]: https://github.com/GoogleCloudPlatform/slurm-gcp/tree/6.8.6/terraform/slurm_cluster/modules/slurm_instance_template

### Example

```yaml
- id: slurm_login
  source: community/modules/scheduler/schedmd-slurm-gcp-v6-login
  use:
  - network
  settings:
    group_name: login
    machine_type: n2-standard-4

- id: slurm_controller
    source: community/modules/scheduler/schedmd-slurm-gcp-v6-controller
    use:
    - network
    - slurm_login
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

[slurm-on-gcp]: https://github.com/GoogleCloudPlatform/slurm-gcp/tree/7
[slurm-gcp-readme]: https://github.com/GoogleCloudPlatform/slurm-gcp/tree/6.8.6#slurm-on-google-cloud-platform

<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| additional\_disks | List of maps of disks. | <pre>list(object({<br>    disk_name    = string<br>    device_name  = string<br>    disk_type    = string<br>    disk_size_gb = number<br>    disk_labels  = map(string)<br>    auto_delete  = bool<br>    boot         = bool<br>  }))</pre> | `[]` | no |
| additional\_networks | Additional network interface details for GCE, if any. | <pre>list(object({<br>    access_config = optional(list(object({<br>      nat_ip       = string<br>      network_tier = string<br>    })), [])<br>    alias_ip_range = optional(list(object({<br>      ip_cidr_range         = string<br>      subnetwork_range_name = string<br>    })), [])<br>    ipv6_access_config = optional(list(object({<br>      network_tier = string<br>    })), [])<br>    network            = optional(string)<br>    network_ip         = optional(string, "")<br>    nic_type           = optional(string)<br>    queue_count        = optional(number)<br>    stack_type         = optional(string)<br>    subnetwork         = optional(string)<br>    subnetwork_project = optional(string)<br>  }))</pre> | `[]` | no |
| allow\_automatic\_updates | If false, disables automatic system package updates on the created instances.  This feature is<br>only available on supported images (or images derived from them).  For more details, see<br>https://cloud.google.com/compute/docs/instances/create-hpc-vm#disable_automatic_updates | `bool` | `true` | no |
| bandwidth\_tier | Configures the network interface card and the maximum egress bandwidth for VMs.<br>  - Setting `platform_default` respects the Google Cloud Platform API default values for networking.<br>  - Setting `virtio_enabled` explicitly selects the VirtioNet network adapter.<br>  - Setting `gvnic_enabled` selects the gVNIC network adapter (without Tier 1 high bandwidth).<br>  - Setting `tier_1_enabled` selects both the gVNIC adapter and Tier 1 high bandwidth networking.<br>  - Note: both gVNIC and Tier 1 networking require a VM image with gVNIC support as well as specific VM families and shapes.<br>  - See [official docs](https://cloud.google.com/compute/docs/networking/configure-vm-with-high-bandwidth-configuration) for more details. | `string` | `"platform_default"` | no |
| can\_ip\_forward | Enable IP forwarding, for NAT instances for example. | `bool` | `false` | no |
| disable\_login\_public\_ips | DEPRECATED: Use `enable_login_public_ips` instead. | `bool` | `null` | no |
| disable\_smt | DEPRECATED: Use `enable_smt` instead. | `bool` | `null` | no |
| disk\_auto\_delete | Whether or not the boot disk should be auto-deleted. | `bool` | `true` | no |
| disk\_labels | Labels specific to the boot disk. These will be merged with var.labels. | `map(string)` | `{}` | no |
| disk\_size\_gb | Boot disk size in GB. | `number` | `50` | no |
| disk\_type | Boot disk type, can be either hyperdisk-balanced, pd-ssd, pd-standard, pd-balanced, or pd-extreme. | `string` | `"pd-ssd"` | no |
| enable\_confidential\_vm | Enable the Confidential VM configuration. Note: the instance image must support option. | `bool` | `false` | no |
| enable\_login\_public\_ips | If set to true. The login node will have a random public IP assigned to it. | `bool` | `false` | no |
| enable\_oslogin | Enables Google Cloud os-login for user login and authentication for VMs.<br>See https://cloud.google.com/compute/docs/oslogin | `bool` | `true` | no |
| enable\_shielded\_vm | Enable the Shielded VM configuration. Note: the instance image must support option. | `bool` | `false` | no |
| enable\_smt | Enables Simultaneous Multi-Threading (SMT) on instance. | `bool` | `false` | no |
| guest\_accelerator | List of the type and count of accelerator cards attached to the instance. | <pre>list(object({<br>    type  = string,<br>    count = number<br>  }))</pre> | `[]` | no |
| instance\_image | Defines the image that will be used in the Slurm controller VM instance.<br><br>Expected Fields:<br>name: The name of the image. Mutually exclusive with family.<br>family: The image family to use. Mutually exclusive with name.<br>project: The project where the image is hosted.<br><br>For more information on creating custom images that comply with Slurm on GCP<br>see the "Slurm on GCP Custom Images" section in docs/vm-images.md. | `map(string)` | <pre>{<br>  "family": "slurm-gcp-6-8-hpc-rocky-linux-8",<br>  "project": "schedmd-slurm-public"<br>}</pre> | no |
| instance\_image\_custom | A flag that designates that the user is aware that they are requesting<br>to use a custom and potentially incompatible image for this Slurm on<br>GCP module.<br><br>If the field is set to false, only the compatible families and project<br>names will be accepted.  The deployment will fail with any other image<br>family or name.  If set to true, no checks will be done.<br><br>See: https://goo.gle/hpc-slurm-images | `bool` | `false` | no |
| instance\_template | DEPRECATED: Instance template can not be specified for login nodes. | `string` | `null` | no |
| labels | Labels, provided as a map. | `map(string)` | `{}` | no |
| machine\_type | Machine type to create. | `string` | `"c2-standard-4"` | no |
| metadata | Metadata, provided as a map. | `map(string)` | `{}` | no |
| min\_cpu\_platform | Specifies a minimum CPU platform. Applicable values are the friendly names of<br>CPU platforms, such as Intel Haswell or Intel Skylake. See the complete list:<br>https://cloud.google.com/compute/docs/instances/specify-min-cpu-platform | `string` | `null` | no |
| name\_prefix | Unique name prefix for login nodes. Automatically populated by the module id if not set.<br>If setting manually, ensure a unique value across all login groups. | `string` | n/a | yes |
| num\_instances | Number of instances to create. This value is ignored if static\_ips is provided. | `number` | `1` | no |
| on\_host\_maintenance | Instance availability Policy. | `string` | `"MIGRATE"` | no |
| preemptible | Allow the instance to be preempted. | `bool` | `false` | no |
| project\_id | Project ID to create resources in. | `string` | n/a | yes |
| region | Region where the instances should be created. | `string` | `null` | no |
| service\_account | DEPRECATED: Use `service_account_email` and `service_account_scopes` instead. | <pre>object({<br>    email  = string<br>    scopes = set(string)<br>  })</pre> | `null` | no |
| service\_account\_email | Service account e-mail address to attach to the login instances. | `string` | `null` | no |
| service\_account\_scopes | Scopes to attach to the login instances. | `set(string)` | <pre>[<br>  "https://www.googleapis.com/auth/cloud-platform"<br>]</pre> | no |
| shielded\_instance\_config | Shielded VM configuration for the instance. Note: not used unless<br>enable\_shielded\_vm is 'true'.<br>  enable\_integrity\_monitoring : Compare the most recent boot measurements to the<br>  integrity policy baseline and return a pair of pass/fail results depending on<br>  whether they match or not.<br>  enable\_secure\_boot : Verify the digital signature of all boot components, and<br>  halt the boot process if signature verification fails.<br>  enable\_vtpm : Use a virtualized trusted platform module, which is a<br>  specialized computer chip you can use to encrypt objects like keys and<br>  certificates. | <pre>object({<br>    enable_integrity_monitoring = bool<br>    enable_secure_boot          = bool<br>    enable_vtpm                 = bool<br>  })</pre> | <pre>{<br>  "enable_integrity_monitoring": true,<br>  "enable_secure_boot": true,<br>  "enable_vtpm": true<br>}</pre> | no |
| static\_ips | List of static IPs for VM instances. | `list(string)` | `[]` | no |
| subnetwork\_self\_link | Subnet to deploy to. | `string` | n/a | yes |
| tags | Network tag list. | `list(string)` | `[]` | no |
| zone | Zone where the instances should be created. If not specified, instances will be<br>spread across available zones in the region. | `string` | `null` | no |

## Outputs

| Name | Description |
|------|-------------|
| login\_nodes | Slurm login instance definition. |

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
