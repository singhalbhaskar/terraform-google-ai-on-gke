# terraform-google-ai-on-gke

## Description

This module provisions a PBS Server Host to operate and administer a PBS
Professional cluster. The following requirements must be observed:

- one must have an existing Altair license server with sufficient licenses to
  run PBS Pro
- jobs should be submitted from a network filesystem mounted on all hosts to
  facilitate file transfers for jobs and their logs

### Example

The following example snippet demonstrates use of the server module in concert
with the [pbspro-preinstall] and [filestore] modules.

```yaml
  - id: pbspro_server
    source: community/modules/scheduler/pbspro-server
    use:
    - homefs
    - pbspro_preinstall
    settings:
      pbs_license_server:  ## IP address or resolvable DNS name of license server
```

[pbspro-preinstall]: ../../scripts/pbspro-preinstall/README.md
[filestore]: ../../../../modules/file-system/filestore/README.md

## GPU Support

More information on GPU support in PBS Pro and other Cluster Toolkit modules
can be found at [docs/gpu-support.md](../../../../docs/gpu-support.md)

## Support

PBS Professional is licensed and supported by [Altair][pbspro]. This module is
maintained and supported by the Cluster Toolkit team in collaboration with Altair.

[pbspro]: https://www.altair.com/pbs-professional

## License

<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| auto\_delete\_boot\_disk | Controls if boot disk should be auto-deleted when instance is deleted. | `bool` | `true` | no |
| bandwidth\_tier | Tier 1 bandwidth increases the maximum egress bandwidth for VMs.<br>  Using the `tier_1_enabled` setting will enable both gVNIC and TIER\_1 higher bandwidth networking.<br>  Using the `gvnic_enabled` setting will only enable gVNIC and will not enable TIER\_1.<br>  Note that TIER\_1 only works with specific machine families & shapes and must be using an image th<br>at supports gVNIC. See [official docs](https://cloud.google.com/compute/docs/networking/configure-v<br>m-with-high-bandwidth-configuration) for more details. | `string` | `"not_enabled"` | no |
| client\_host\_count | Number of client hosts to configure | `number` | `0` | no |
| client\_hostname\_prefix | Name prefix for client hosts | `string` | n/a | yes |
| deployment\_name | Cluster Toolkit deployment name. Cloud resource names will include this value. | `string` | n/a | yes |
| disk\_size\_gb | Size of disk for instances. | `number` | `200` | no |
| disk\_type | Disk type for instances. | `string` | `"pd-standard"` | no |
| enable\_oslogin | Enable or Disable OS Login with "ENABLE" or "DISABLE". Set to "INHERIT" to inherit project OS Login setting. | `string` | `"ENABLE"` | no |
| enable\_public\_ips | If set to true, instances will have public IPs on the internet. | `bool` | `true` | no |
| execution\_host\_count | Number of execution hosts to configure | `number` | n/a | yes |
| execution\_hostname\_prefix | Name prefix for execution hosts | `string` | n/a | yes |
| guest\_accelerator | List of the type and count of accelerator cards attached to the instance. | <pre>list(object({<br>    type  = string,<br>    count = number<br>  }))</pre> | `null` | no |
| instance\_count | Number of instances | `number` | `1` | no |
| instance\_image | Instance Image<br><br>Expected Fields:<br>name: The name of the image. Mutually exclusive with family.<br>family: The image family to use. Mutually exclusive with name.<br>project: The project where the image is hosted. | `map(string)` | <pre>{<br>  "name": "hpc-centos-7-v20240712",<br>  "project": "cloud-hpc-image-public"<br>}</pre> | no |
| labels | Labels to add to the instances. Key-value pairs. | `map(string)` | n/a | yes |
| local\_ssd\_count | The number of local SSDs to attach to each VM. See https://cloud.google.com/compute/docs/disks/local-ssd. | `number` | `0` | no |
| local\_ssd\_interface | Interface to be used with local SSDs. Can be either 'NVME' or 'SCSI'. No effect unless `local_ssd_count` is also set. | `string` | `"NVME"` | no |
| machine\_type | Machine type to use for the instance creation | `string` | `"c2-standard-60"` | no |
| metadata | Metadata, provided as a map | `map(string)` | `{}` | no |
| name\_prefix | Name prefix for PBS execution hostnames | `string` | `null` | no |
| network\_interfaces | A list of network interfaces. The options match that of the terraform<br>network\_interface block of google\_compute\_instance. For descriptions of the<br>subfields or more information see the documentation:<br>https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/compute_instance#nested_network_interface<br><br>**\_NOTE:\_** If `network_interfaces` are set, `network_self_link` and<br>`subnetwork_self_link` will be ignored, even if they are provided through<br>the `use` field. `bandwidth_tier` and `enable_public_ips` also do not apply<br>to network interfaces defined in this variable.<br><br>Subfields:<br>network            (string, required if subnetwork is not supplied)<br>subnetwork         (string, required if network is not supplied)<br>subnetwork\_project (string, optional)<br>network\_ip         (string, optional)<br>nic\_type           (string, optional, choose from ["GVNIC", "VIRTIO\_NET"])<br>stack\_type         (string, optional, choose from ["IPV4\_ONLY", "IPV4\_IPV6"])<br>queue\_count        (number, optional)<br>access\_config      (object, optional)<br>ipv6\_access\_config (object, optional)<br>alias\_ip\_range     (list(object), optional) | <pre>list(object({<br>    network            = string,<br>    subnetwork         = string,<br>    subnetwork_project = string,<br>    network_ip         = string,<br>    nic_type           = string,<br>    stack_type         = string,<br>    queue_count        = number,<br>    access_config = list(object({<br>      nat_ip                 = string,<br>      public_ptr_domain_name = string,<br>      network_tier           = string<br>    })),<br>    ipv6_access_config = list(object({<br>      public_ptr_domain_name = string,<br>      network_tier           = string<br>    })),<br>    alias_ip_range = list(object({<br>      ip_cidr_range         = string,<br>      subnetwork_range_name = string<br>    }))<br>  }))</pre> | `[]` | no |
| network\_self\_link | The self link of the network to attach the VM. | `string` | `"default"` | no |
| network\_storage | An array of network attached storage mounts to be configured. | <pre>list(object({<br>    server_ip             = string,<br>    remote_mount          = string,<br>    local_mount           = string,<br>    fs_type               = string,<br>    mount_options         = string,<br>    client_install_runner = map(string)<br>    mount_runner          = map(string)<br>  }))</pre> | `[]` | no |
| on\_host\_maintenance | Describes maintenance behavior for the instance. If left blank this will default to `MIGRATE` except for when `placement_policy`, spot provisioning, or GPUs require it to be `TERMINATE` | `string` | `null` | no |
| pbs\_data\_service\_user | PBS Data Service POSIX user | `string` | `"pbsdata"` | no |
| pbs\_exec | Root path in which to install PBS | `string` | `"/opt/pbs"` | no |
| pbs\_home | PBS working directory | `string` | `"/var/spool/pbs"` | no |
| pbs\_license\_server | IP address or DNS name of PBS license server | `string` | n/a | yes |
| pbs\_license\_server\_port | Networking port of PBS license server | `number` | `6200` | no |
| pbs\_server\_rpm\_url | Path to PBS Pro Server Host RPM file | `string` | n/a | yes |
| placement\_policy | Control where your VM instances are physically located relative to each other within a zone. | <pre>object({<br>    vm_count                  = number,<br>    availability_domain_count = number,<br>    collocation               = string,<br>  })</pre> | `null` | no |
| project\_id | Project in which Google Cloud Storage bucket will be created | `string` | n/a | yes |
| region | Default region for creating resources | `string` | n/a | yes |
| server\_conf | A sequence of qmgr commands in format as generated by qmgr -c 'print server' | `string` | `"# empty qmgr configuration file"` | no |
| service\_account | Service account to attach to the instance. See https://www.terraform.io/docs/providers/google/r/compute_instance_template.html#service_account. | <pre>object({<br>    email  = string,<br>    scopes = set(string)<br>  })</pre> | <pre>{<br>  "email": null,<br>  "scopes": [<br>    "https://www.googleapis.com/auth/devstorage.read_write",<br>    "https://www.googleapis.com/auth/logging.write",<br>    "https://www.googleapis.com/auth/monitoring.write",<br>    "https://www.googleapis.com/auth/servicecontrol",<br>    "https://www.googleapis.com/auth/service.management.readonly",<br>    "https://www.googleapis.com/auth/trace.append"<br>  ]<br>}</pre> | no |
| spot | Provision VMs using discounted Spot pricing, allowing for preemption | `bool` | `false` | no |
| startup\_script | Startup script used on the instance | `string` | `null` | no |
| subnetwork\_self\_link | The self link of the subnetwork to attach the VM. | `string` | `null` | no |
| tags | Network tags, provided as a list | `list(string)` | `[]` | no |
| threads\_per\_core | Sets the number of threads per physical core. By setting threads\_per\_core<br>to 2, Simultaneous Multithreading (SMT) is enabled extending the total number<br>of virtual cores. For example, a machine of type c2-standard-60 will have 60<br>virtual cores with threads\_per\_core equal to 2. With threads\_per\_core equal<br>to 1 (SMT turned off), only the 30 physical cores will be available on the VM.<br><br>The default value of \"0\" will turn off SMT for supported machine types, and<br>will fall back to GCE defaults for unsupported machine types (t2d, shared-core<br>instances, or instances with less than 2 vCPU).<br><br>Disabling SMT can be more performant in many HPC workloads, therefore it is<br>disabled by default where compatible.<br><br>null = SMT configuration will use the GCE defaults for the machine type<br>0 = SMT will be disabled where compatible (default)<br>1 = SMT will always be disabled (will fail on incompatible machine types)<br>2 = SMT will always be enabled (will fail on incompatible machine types) | `number` | `0` | no |
| zone | Default zone for creating resources | `string` | n/a | yes |

## Outputs

| Name | Description |
|------|-------------|
| pbs\_server | Name of the controller node |

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
