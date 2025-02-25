## Description

This module creates a nodeset data structure intended to be input to the
[schedmd-slurm-gcp-v6-partition](../schedmd-slurm-gcp-v6-partition/) module.

Nodesets allow adding heterogeneous node types to a partition, and hence
running jobs that mix multiple node characteristics. See the [heterogeneous jobs
section][hetjobs] of the SchedMD documentation for more information.

To specify nodes from a specific nodesets in a partition, the [`--nodelist`]
(or `-w`) flag can be used, for example:

```bash
srun -N 3 -p compute --nodelist cluster-compute-group-[0-2] hostname
```

Where the 3 nodes will be selected from the nodes `cluster-compute-group-[0-2]`
in the compute partition.

Additionally, depending on how the nodes differ, a constraint can be added via
the [`--constraint`] (or `-C`) flag or other flags such as `--mincpus` can be
used to specify nodes with the desired characteristics.

[`--nodelist`]: https://slurm.schedmd.com/srun.html#OPT_nodelist
[`--constraint`]: https://slurm.schedmd.com/srun.html#OPT_constraint
[hetjobs]: https://slurm.schedmd.com/heterogeneous_jobs.html

### Example

The following code snippet creates a partition module using the `nodeset`
module as input with:

* a max node count of 200
* VM machine type of `c2-standard-30`
* partition name of "compute"
* default nodeset name of "ghpc"
* connected to the `network` module via `use`
* nodes mounted to homefs via `use`

```yaml
- id: nodeset
  source: community/modules/compute/schedmd-slurm-gcp-v6-nodeset
  use:
  - network
  settings:
    node_count_dynamic_max: 200
    machine_type: c2-standard-30

- id: compute_partition
  source: community/modules/compute/schedmd-slurm-gcp-v6-partition
  use:
  - homefs
  - nodeset
  settings:
    partition_name: compute
```

## Custom Images

For more information on creating valid custom images for the node group VM
instances or for custom instance templates, see our [vm-images.md] documentation
page.

[vm-images.md]: ../../../../docs/vm-images.md#slurm-on-gcp-custom-images

## GPU Support

More information on GPU support in Slurm on GCP and other Cluster Toolkit modules
can be found at [docs/gpu-support.md](../../../../docs/gpu-support.md)

### Compute VM Zone Policies

The Slurm on GCP nodeset module allows you to specify additional zones in
which to create VMs through [bulk creation][bulk]. This is valuable when
configuring partitions with popular VM families and you desire access to
more compute resources across zones.

[bulk]: https://cloud.google.com/compute/docs/instances/multiple/about-bulk-creation
[networkpricing]: https://cloud.google.com/vpc/network-pricing

> **_WARNING:_** Lenient zone policies can lead to additional egress costs when
> moving large amounts of data between zones in the same region. For example,
> traffic between VMs and traffic from VMs to shared filesystems such as
> Filestore. For more information on egress fees, see the
> [Network Pricing][networkpricing] Google Cloud documentation.
>
> To avoid egress charges, ensure your compute nodes are created in a single
> zone by setting var.zone and leaving var.zones to its default value of the
> empty list.
>
> **_NOTE:_** If a new zone is added to the region while the cluster is active,
> nodes in the partition may be created in that zone. In this case, the
> partition may need to be redeployed to ensure the newly added zone is denied.

In the zonal example below, the nodeset's zone implicitly defaults to the
deployment variable `vars.zone`:

```yaml
vars:
  zone: us-central1-f

- id: zonal-nodeset
  source: community/modules/compute/schedmd-slurm-gcp-v6-nodeset
```

In the example below, we enable creation in additional zones:

```yaml
vars:
  zone: us-central1-f

- id: multi-zonal-nodeset
  source: community/modules/compute/schedmd-slurm-gcp-v6-nodeset
  settings:
    zones:
    - us-central1-a
    - us-central1-b
```

## Support
The Cluster Toolkit team maintains the wrapper around the [slurm-on-gcp] terraform
modules. For support with the underlying modules, see the instructions in the
[slurm-gcp README][slurm-gcp-readme].

[slurm-on-gcp]: https://github.com/GoogleCloudPlatform/slurm-gcp
[slurm-gcp-readme]: https://github.com/GoogleCloudPlatform/slurm-gcp#slurm-on-google-cloud-platform

<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| access\_config | Access configurations, i.e. IPs via which the VM instance can be accessed via the Internet. | <pre>list(object({<br>    nat_ip       = string<br>    network_tier = string<br>  }))</pre> | `[]` | no |
| additional\_disks | Configurations of additional disks to be included on the partition nodes. | <pre>list(object({<br>    disk_name    = string<br>    device_name  = string<br>    disk_size_gb = number<br>    disk_type    = string<br>    disk_labels  = map(string)<br>    auto_delete  = bool<br>    boot         = bool<br>  }))</pre> | `[]` | no |
| additional\_networks | Additional network interface details for GCE, if any. | <pre>list(object({<br>    network            = optional(string)<br>    subnetwork         = string<br>    subnetwork_project = optional(string)<br>    network_ip         = optional(string, "")<br>    nic_type           = optional(string)<br>    stack_type         = optional(string)<br>    queue_count        = optional(number)<br>    access_config = optional(list(object({<br>      nat_ip       = string<br>      network_tier = string<br>    })), [])<br>    ipv6_access_config = optional(list(object({<br>      network_tier = string<br>    })), [])<br>    alias_ip_range = optional(list(object({<br>      ip_cidr_range         = string<br>      subnetwork_range_name = string<br>    })), [])<br>  }))</pre> | `[]` | no |
| allow\_automatic\_updates | If false, disables automatic system package updates on the created instances.  This feature is<br>only available on supported images (or images derived from them).  For more details, see<br>https://cloud.google.com/compute/docs/instances/create-hpc-vm#disable_automatic_updates | `bool` | `true` | no |
| bandwidth\_tier | Configures the network interface card and the maximum egress bandwidth for VMs.<br>  - Setting `platform_default` respects the Google Cloud Platform API default values for networking.<br>  - Setting `virtio_enabled` explicitly selects the VirtioNet network adapter.<br>  - Setting `gvnic_enabled` selects the gVNIC network adapter (without Tier 1 high bandwidth).<br>  - Setting `tier_1_enabled` selects both the gVNIC adapter and Tier 1 high bandwidth networking.<br>  - Note: both gVNIC and Tier 1 networking require a VM image with gVNIC support as well as specific VM families and shapes.<br>  - See [official docs](https://cloud.google.com/compute/docs/networking/configure-vm-with-high-bandwidth-configuration) for more details. | `string` | `"platform_default"` | no |
| can\_ip\_forward | Enable IP forwarding, for NAT instances for example. | `bool` | `false` | no |
| disable\_public\_ips | DEPRECATED: Use `enable_public_ips` instead. | `bool` | `null` | no |
| disk\_auto\_delete | Whether or not the boot disk should be auto-deleted. | `bool` | `true` | no |
| disk\_labels | Labels specific to the boot disk. These will be merged with var.labels. | `map(string)` | `{}` | no |
| disk\_size\_gb | Size of boot disk to create for the partition compute nodes. | `number` | `50` | no |
| disk\_type | Boot disk type, can be either hyperdisk-balanced, pd-ssd, pd-standard, pd-balanced, or pd-extreme. | `string` | `"pd-standard"` | no |
| dws\_flex | If set and `enabled = true`, will utilize the DWS Flex Start to provision nodes.<br> See: https://cloud.google.com/blog/products/compute/introducing-dynamic-workload-scheduler<br> Options:<br> - enable: Enable DWS Flex Start<br> - max\_run\_duration: Maximum duration in seconds for the job to run, should not exceed 1,209,600 (2 weeks).<br> - use\_job\_duration: Use the job duration to determine the max\_run\_duration, if job duration is not set, max\_run\_duration will be used.<br><br>Limitations:<br> - CAN NOT be used with reservations;<br> - CAN NOT be used with placement groups;<br> - If `use_job_duration` is enabled nodeset can be used in "exclusive" partitions only | <pre>object({<br>    enabled          = optional(bool, true)<br>    max_run_duration = optional(number, 1209600) # 2 weeks<br>    use_job_duration = optional(bool, false)<br>  })</pre> | <pre>{<br>  "enabled": false<br>}</pre> | no |
| enable\_confidential\_vm | Enable the Confidential VM configuration. Note: the instance image must support option. | `bool` | `false` | no |
| enable\_maintenance\_reservation | Enables slurm reservation for scheduled maintenance. | `bool` | `false` | no |
| enable\_opportunistic\_maintenance | On receiving maintenance notification, maintenance will be performed as soon as nodes becomes idle. | `bool` | `false` | no |
| enable\_oslogin | Enables Google Cloud os-login for user login and authentication for VMs.<br>See https://cloud.google.com/compute/docs/oslogin | `bool` | `true` | no |
| enable\_placement | Enable placement groups. | `bool` | `true` | no |
| enable\_public\_ips | If set to true. The node group VMs will have a random public IP assigned to it. Ignored if access\_config is set. | `bool` | `false` | no |
| enable\_shielded\_vm | Enable the Shielded VM configuration. Note: the instance image must support option. | `bool` | `false` | no |
| enable\_smt | Enables Simultaneous Multi-Threading (SMT) on instance. | `bool` | `false` | no |
| enable\_spot\_vm | Enable the partition to use spot VMs (https://cloud.google.com/spot-vms). | `bool` | `false` | no |
| future\_reservation | If set, will make use of the future reservation for the nodeset. Input can be either the future reservation name or its selfLink in the format 'projects/PROJECT\_ID/zones/ZONE/futureReservations/FUTURE\_RESERVATION\_NAME'.<br>See https://cloud.google.com/compute/docs/instances/future-reservations-overview | `string` | `""` | no |
| guest\_accelerator | List of the type and count of accelerator cards attached to the instance. | <pre>list(object({<br>    type  = string,<br>    count = number<br>  }))</pre> | `[]` | no |
| instance\_image | Defines the image that will be used in the Slurm node group VM instances.<br><br>Expected Fields:<br>name: The name of the image. Mutually exclusive with family.<br>family: The image family to use. Mutually exclusive with name.<br>project: The project where the image is hosted.<br><br>For more information on creating custom images that comply with Slurm on GCP<br>see the "Slurm on GCP Custom Images" section in docs/vm-images.md. | `map(string)` | <pre>{<br>  "family": "slurm-gcp-6-8-hpc-rocky-linux-8",<br>  "project": "schedmd-slurm-public"<br>}</pre> | no |
| instance\_image\_custom | A flag that designates that the user is aware that they are requesting<br>to use a custom and potentially incompatible image for this Slurm on<br>GCP module.<br><br>If the field is set to false, only the compatible families and project<br>names will be accepted.  The deployment will fail with any other image<br>family or name.  If set to true, no checks will be done.<br><br>See: https://goo.gle/hpc-slurm-images | `bool` | `false` | no |
| instance\_properties | Override the instance properties. Used to test features not supported by Slurm GCP,<br>recommended for advanced usage only.<br>See https://cloud.google.com/compute/docs/reference/rest/v1/regionInstances/bulkInsert<br>If any sub-field (e.g. scheduling) is set, it will override the values computed by<br>SlurmGCP and ignoring values of provided vars. | `any` | `null` | no |
| instance\_template | DEPRECATED: Instance template can not be specified for compute nodes. | `string` | `null` | no |
| labels | Labels to add to partition compute instances. Key-value pairs. | `map(string)` | `{}` | no |
| machine\_type | Compute Platform machine type to use for this partition compute nodes. | `string` | `"c2-standard-60"` | no |
| maintenance\_interval | Sets the maintenance interval for instances in this nodeset.<br>See https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/compute_instance#maintenance_interval. | `string` | `null` | no |
| metadata | Metadata, provided as a map. | `map(string)` | `{}` | no |
| min\_cpu\_platform | The name of the minimum CPU platform that you want the instance to use. | `string` | `null` | no |
| name | Name of the nodeset. Automatically populated by the module id if not set.<br>If setting manually, ensure a unique value across all nodesets. | `string` | n/a | yes |
| network\_storage | An array of network attached storage mounts to be configured on nodes. | <pre>list(object({<br>    server_ip     = string,<br>    remote_mount  = string,<br>    local_mount   = string,<br>    fs_type       = string,<br>    mount_options = string,<br>  }))</pre> | `[]` | no |
| node\_conf | Map of Slurm node line configuration. | `map(any)` | `{}` | no |
| node\_count\_dynamic\_max | Maximum number of auto-scaling nodes allowed in this partition. | `number` | `10` | no |
| node\_count\_static | Number of nodes to be statically created. | `number` | `0` | no |
| on\_host\_maintenance | Instance availability Policy.<br><br>Note: Placement groups are not supported when on\_host\_maintenance is set to<br>"MIGRATE" and will be deactivated regardless of the value of<br>enable\_placement. To support enable\_placement, ensure on\_host\_maintenance is<br>set to "TERMINATE". | `string` | `"TERMINATE"` | no |
| placement\_max\_distance | Maximum distance between nodes in the placement group. Requires enable\_placement to be true. Values must be supported by the chosen machine type. | `number` | `null` | no |
| preemptible | Should use preemptibles to burst. | `bool` | `false` | no |
| project\_id | Project ID to create resources in. | `string` | n/a | yes |
| region | The default region for Cloud resources. | `string` | n/a | yes |
| reservation\_name | Name of the reservation to use for VM resources, should be in one of the following formats:<br>- projects/PROJECT\_ID/reservations/RESERVATION\_NAME[/SUFF/IX]<br>- RESERVATION\_NAME[/SUFF/IX]<br><br>Must be a "SPECIFIC" reservation<br>Set to empty string if using no reservation or automatically-consumed reservations | `string` | `""` | no |
| service\_account | DEPRECATED: Use `service_account_email` and `service_account_scopes` instead. | <pre>object({<br>    email  = string<br>    scopes = set(string)<br>  })</pre> | `null` | no |
| service\_account\_email | Service account e-mail address to attach to the compute instances. | `string` | `null` | no |
| service\_account\_scopes | Scopes to attach to the compute instances. | `set(string)` | <pre>[<br>  "https://www.googleapis.com/auth/cloud-platform"<br>]</pre> | no |
| shielded\_instance\_config | Shielded VM configuration for the instance. Note: not used unless<br>enable\_shielded\_vm is 'true'.<br>- enable\_integrity\_monitoring : Compare the most recent boot measurements to the<br>  integrity policy baseline and return a pair of pass/fail results depending on<br>  whether they match or not.<br>- enable\_secure\_boot : Verify the digital signature of all boot components, and<br>  halt the boot process if signature verification fails.<br>- enable\_vtpm : Use a virtualized trusted platform module, which is a<br>  specialized computer chip you can use to encrypt objects like keys and<br>  certificates. | <pre>object({<br>    enable_integrity_monitoring = bool<br>    enable_secure_boot          = bool<br>    enable_vtpm                 = bool<br>  })</pre> | <pre>{<br>  "enable_integrity_monitoring": true,<br>  "enable_secure_boot": true,<br>  "enable_vtpm": true<br>}</pre> | no |
| spot\_instance\_config | Configuration for spot VMs. | <pre>object({<br>    termination_action = string<br>  })</pre> | `null` | no |
| startup\_script | Startup script used by VMs in this nodeset.<br>NOTE: will be executed after `compute_startup_script` defined on controller module. | `string` | `"# no-op"` | no |
| subnetwork\_self\_link | Subnet to deploy to. | `string` | n/a | yes |
| tags | Network tag list. | `list(string)` | `[]` | no |
| zone | Zone in which to create compute VMs. Additional zones in the same region can be specified in var.zones. | `string` | n/a | yes |
| zone\_target\_shape | Strategy for distributing VMs across zones in a region.<br>ANY<br>  GCE picks zones for creating VM instances to fulfill the requested number of VMs<br>  within present resource constraints and to maximize utilization of unused zonal<br>  reservations.<br>ANY\_SINGLE\_ZONE (default)<br>  GCE always selects a single zone for all the VMs, optimizing for resource quotas,<br>  available reservations and general capacity.<br>BALANCED<br>  GCE prioritizes acquisition of resources, scheduling VMs in zones where resources<br>  are available while distributing VMs as evenly as possible across allowed zones<br>  to minimize the impact of zonal failure. | `string` | `"ANY_SINGLE_ZONE"` | no |
| zones | Additional zones in which to allow creation of partition nodes. Google Cloud<br>will find zone based on availability, quota and reservations.<br>Should not be set if SPECIFIC reservation is used. | `set(string)` | `[]` | no |

## Outputs

| Name | Description |
|------|-------------|
| nodeset | Details of the nodeset. Typically used as input to `schedmd-slurm-gcp-v6-partition`. |

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
