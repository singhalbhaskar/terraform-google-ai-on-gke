## Description

This module creates a GPU accelerated virtual machine that can be accessed using
Chrome Remote Desktop.

> **Note**: This is an experimental module. This module has only been tested in
> limited capacity with the Cluster Toolkit. The module interface may have undergo
> breaking changes in the future.

### Example

The following example will create a single GPU accelerated remote desktop.

```yaml
  - id: remote-desktop
    source: community/modules/remote-desktop/chrome-remote-desktop
    use: [network1]
    settings:
      install_nvidia_driver: true
```

### Setting up the Remote Desktop

1. Once the remote desktop has been deployed, navigate to https://remotedesktop.google.com/headless.
1. Click through `Begin`, `Next`, & `Authorize`.
1. Copy the code snippet for `Debian Linux`.
1. SSH into the remote desktop machine. It will be listed under
   [VM Instances](https://console.cloud.google.com/compute/instances) in the
   Google Cloud web console.
1. Run the copied command and follow instructions to set up a PIN.
1. You should now see your machine listed on the
   [Chrome Remote Desktop page](https://remotedesktop.google.com/access) under `Remote devices`.
1. Click on your machine and enter PIN if prompted.

## License

<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| add\_deployment\_name\_before\_prefix | If true, the names of VMs and disks will always be prefixed with `deployment_name` to enable uniqueness across deployments.<br>See `name_prefix` for further details on resource naming behavior. | `bool` | `false` | no |
| auto\_delete\_boot\_disk | Controls if boot disk should be auto-deleted when instance is deleted. | `bool` | `true` | no |
| bandwidth\_tier | Tier 1 bandwidth increases the maximum egress bandwidth for VMs.<br>  Using the `tier_1_enabled` setting will enable both gVNIC and TIER\_1 higher bandwidth networking.<br>  Using the `gvnic_enabled` setting will only enable gVNIC and will not enable TIER\_1.<br>  Note that TIER\_1 only works with specific machine families & shapes and must be using an image th<br>at supports gVNIC. See [official docs](https://cloud.google.com/compute/docs/networking/configure-v<br>m-with-high-bandwidth-configuration) for more details. | `string` | `"not_enabled"` | no |
| deployment\_name | Cluster Toolkit deployment name. Cloud resource names will include this value. | `string` | n/a | yes |
| disk\_size\_gb | Size of disk for instances. | `number` | `200` | no |
| disk\_type | Disk type for instances. | `string` | `"pd-balanced"` | no |
| enable\_oslogin | Enable or Disable OS Login with "ENABLE" or "DISABLE". Set to "INHERIT" to inherit project OS Login setting. | `string` | `"ENABLE"` | no |
| enable\_public\_ips | If set to true, instances will have public IPs on the internet. | `bool` | `true` | no |
| guest\_accelerator | List of the type and count of accelerator cards attached to the instance. Requires virtual workstation accelerator if Nvidia Grid Drivers are required | <pre>list(object({<br>    type  = string,<br>    count = number<br>  }))</pre> | <pre>[<br>  {<br>    "count": 1,<br>    "type": "nvidia-tesla-t4-vws"<br>  }<br>]</pre> | no |
| install\_nvidia\_driver | Installs the nvidia driver (true/false). For details, see https://cloud.google.com/compute/docs/gpus/install-drivers-gpu | `bool` | n/a | yes |
| instance\_count | Number of instances | `number` | `1` | no |
| instance\_image | Image used to build chrome remote desktop node. The default image is<br>name="debian-12-bookworm-v20240815" and project="debian-cloud".<br>NOTE: uses fixed version of image to avoid NVIDIA driver compatibility issues.<br><br>An alternative image is from name="ubuntu-2204-jammy-v20240126" and project="ubuntu-os-cloud".<br><br>Expected Fields:<br>name: The name of the image. Mutually exclusive with family.<br>family: The image family to use. Mutually exclusive with name.<br>project: The project where the image is hosted. | `map(string)` | <pre>{<br>  "name": "debian-12-bookworm-v20240815",<br>  "project": "debian-cloud"<br>}</pre> | no |
| labels | Labels to add to the instances. Key-value pairs. | `map(string)` | `{}` | no |
| machine\_type | Machine type to use for the instance creation. Must be N1 family if GPU is used. | `string` | `"n1-standard-8"` | no |
| metadata | Metadata, provided as a map | `map(string)` | `{}` | no |
| name\_prefix | An optional name for all VM and disk resources.<br>If not supplied, `deployment_name` will be used.<br>When `name_prefix` is supplied, and `add_deployment_name_before_prefix` is set,<br>then resources are named by "<`deployment_name`>-<`name_prefix`>-<#>". | `string` | `null` | no |
| network\_interfaces | A list of network interfaces. The options match that of the terraform<br>network\_interface block of google\_compute\_instance. For descriptions of the<br>subfields or more information see the documentation:<br>https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/compute_instance#nested_network_interface<br>**\_NOTE:\_** If `network_interfaces` are set, `network_self_link` and<br>`subnetwork_self_link` will be ignored, even if they are provided through<br>the `use` field. `bandwidth_tier` and `enable_public_ips` also do not apply<br>to network interfaces defined in this variable.<br>Subfields:<br>network            (string, required if subnetwork is not supplied)<br>subnetwork         (string, required if network is not supplied)<br>subnetwork\_project (string, optional)<br>network\_ip         (string, optional)<br>nic\_type           (string, optional, choose from ["GVNIC", "VIRTIO\_NET", "RDMA", "IRDMA", "MRDMA"])<br>stack\_type         (string, optional, choose from ["IPV4\_ONLY", "IPV4\_IPV6"])<br>queue\_count        (number, optional)<br>access\_config      (object, optional)<br>ipv6\_access\_config (object, optional)<br>alias\_ip\_range     (list(object), optional) | <pre>list(object({<br>    network            = string,<br>    subnetwork         = string,<br>    subnetwork_project = string,<br>    network_ip         = string,<br>    nic_type           = string,<br>    stack_type         = string,<br>    queue_count        = number,<br>    access_config = list(object({<br>      nat_ip                 = string,<br>      public_ptr_domain_name = string,<br>      network_tier           = string<br>    })),<br>    ipv6_access_config = list(object({<br>      public_ptr_domain_name = string,<br>      network_tier           = string<br>    })),<br>    alias_ip_range = list(object({<br>      ip_cidr_range         = string,<br>      subnetwork_range_name = string<br>    }))<br>  }))</pre> | `[]` | no |
| network\_self\_link | The self link of the network to attach the VM. | `string` | `"default"` | no |
| network\_storage | An array of network attached storage mounts to be configured. | <pre>list(object({<br>    server_ip             = string,<br>    remote_mount          = string,<br>    local_mount           = string,<br>    fs_type               = string,<br>    mount_options         = string,<br>    client_install_runner = map(string)<br>    mount_runner          = map(string)<br>  }))</pre> | `[]` | no |
| on\_host\_maintenance | Describes maintenance behavior for the instance. If left blank this will default to `MIGRATE` except for when `placement_policy`, spot provisioning, or GPUs require it to be `TERMINATE` | `string` | `"TERMINATE"` | no |
| project\_id | Project in which Google Cloud resources will be created | `string` | n/a | yes |
| region | Default region for creating resources | `string` | n/a | yes |
| service\_account | Service account to attach to the instance. See https://www.terraform.io/docs/providers/google/r/compute_instance_template.html#service_account. | <pre>object({<br>    email  = string,<br>    scopes = set(string)<br>  })</pre> | <pre>{<br>  "email": null,<br>  "scopes": [<br>    "https://www.googleapis.com/auth/cloud-platform"<br>  ]<br>}</pre> | no |
| spot | Provision VMs using discounted Spot pricing, allowing for preemption | `bool` | `false` | no |
| startup\_script | Startup script used on the instance | `string` | `null` | no |
| subnetwork\_self\_link | The self link of the subnetwork to attach the VM. | `string` | `null` | no |
| tags | Network tags, provided as a list | `list(string)` | `[]` | no |
| threads\_per\_core | Sets the number of threads per physical core | `number` | `2` | no |
| zone | Default zone for creating resources | `string` | n/a | yes |

## Outputs

| Name | Description |
|------|-------------|
| instance\_name | Name of the first instance created, if any. |
| startup\_script | script to load and run all runners, as a string value. |

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
