# terraform-google-ai-on-gke

## Description

This module creates a slurm controller node via the [slurm-gcp]
[slurm\_controller\_instance] and [slurm\_instance\_template] modules.

More information about Slurm On GCP can be found at the
[project's GitHub page][slurm-gcp] and in the
[Slurm on Google Cloud User Guide][slurm-ug].

The [user guide][slurm-ug] provides detailed instructions on customizing and
enhancing the Slurm on GCP cluster as well as recommendations on configuring the
controller for optimal performance at different scales.

[slurm-gcp]: https://github.com/GoogleCloudPlatform/slurm-gcp/tree/6.8.6
[slurm\_controller\_instance]: https://github.com/GoogleCloudPlatform/slurm-gcp/tree/6.8.6/terraform/slurm_cluster/modules/slurm_controller_instance
[slurm\_instance\_template]: https://github.com/GoogleCloudPlatform/slurm-gcp/tree/6.8.6/terraform/slurm_cluster/modules/slurm_instance_template
[slurm-ug]: https://goo.gle/slurm-gcp-user-guide.
[enable\_cleanup\_compute]: #input\_enable\_cleanup\_compute
[enable\_cleanup\_subscriptions]: #input\_enable\_cleanup\_subscriptions
[enable\_reconfigure]: #input\_enable\_reconfigure

### Example

```yaml
- id: slurm_controller
  source: community/modules/scheduler/schedmd-slurm-gcp-v6-controller
  use:
  - network
  - homefs
  - compute_partition
  settings:
    machine_type: c2-standard-8
```

This creates a controller node with the following attributes:

* connected to the primary subnetwork of `network`
* the filesystem with the ID `homefs` (defined elsewhere in the blueprint)
  mounted
* One partition with the ID `compute_partition` (defined elsewhere in the
  blueprint)
* machine type upgraded from the default `c2-standard-4` to `c2-standard-8`

### Live Cluster Reconfiguration

The `schedmd-slurm-gcp-v6-controller` module supports the reconfiguration of
partitions and slurm configuration in a running, active cluster.

To reconfigure a running cluster:

1. Edit the blueprint with the desired configuration changes
2. Call `gcluster create <blueprint> -w` to overwrite the deployment directory
3. Follow instructions in terminal to deploy

The following are examples of updates that can be made to a running cluster:

* Add or remove a partition to the cluster
* Resize an existing partition
* Attach new network storage to an existing partition

> **NOTE**: Changing the VM `machine_type` of a partition may not work.
> It is better to create a new partition and delete the old one.

## Custom Images

For more information on creating valid custom images for the controller VM
instance or for custom instance templates, see our [vm-images.md] documentation
page.

[vm-images.md]: ../../../../docs/vm-images.md#slurm-on-gcp-custom-images

## GPU Support

More information on GPU support in Slurm on GCP and other Cluster Toolkit modules
can be found at [docs/gpu-support.md](../../../../docs/gpu-support.md)

## Reservation for Scheduled Maintenance

A [maintenance event](https://cloud.google.com/compute/docs/instances/host-maintenance-overview#maintenanceevents) is when a compute engine stops a VM to perform a hardware or
software update which is determined by the host maintenance policy. This can
also affect the running jobs if the maintenance kicks in. Now, Customers can
protect jobs from getting terminated due to maintenance using the cluster
toolkit. You can enable creation of reservation for scheduled maintenance for
your compute nodeset and Slurm will reserve your node for maintenance during the
maintenance window. If you try to schedule any jobs which overlap with the
maintenance reservation, Slurm would not schedule any job.

You can specify in your blueprint like

```yaml
  - id: compute_nodeset
    source: community/modules/compute/schedmd-slurm-gcp-v6-nodeset
    use: [network]
    settings:
      enable_maintenance_reservation: true
```

To enable creation of reservation for maintenance.

While running job on slurm cluster, you can specify total run time of the job
using [-t flag](https://slurm.schedmd.com/srun.html#OPT_time).This would only
run the job outside of the maintenance window.

```shell
srun -n1 -pcompute -t 10:00 <job.sh>
```

Currently upcoming maintenance notification is supported in ALPHA version of
compute API. You can update the API version from your blueprint,

```yaml
  - id: slurm_controller
    source: community/modules/scheduler/schedmd-slurm-gcp-v6-controller
    settings:
      endpoint_versions:
        compute: "alpha"
```

## Opportunistic GCP maintenance in Slurm

Customers can also enable running GCP maintenance as Slurm job opportunistically
to perform early maintenance. If a node is detected for maintenance, Slurm will
create a job to perform maintenance and put it in the job queue.

If [backfill](https://slurm.schedmd.com/sched_config.html#backfill) scheduler is
used, Slurm will backfill maintenance job if it can find any empty time window.

Customer can also choose builtin scheduler type. In this case, Slurm would run
maintenance job in strictly priority order. If the maintenance job doesn't kick
in, then forced maintenance will take place at scheduled window.

Customer can enable this feature at nodeset level by,

```yaml
  - id: debug_nodeset
    source: community/modules/compute/schedmd-slurm-gcp-v6-nodeset
    use: [network]
    settings:
      enable_opportunistic_maintenance: true
```

## Placement Max Distance

When using
[enable_placement](../../../../community/modules/compute/schedmd-slurm-gcp-v6-nodeset/README.md#input_enable_placement)
with Slurm, Google Compute Engine will attempt to place VMs as physically close
together as possible. Capacity constraints at the time of VM creation may still
force VMs to be spread across multiple racks. Google provides the `max-distance`
flag which can used to control the maximum spreading allowed. Read more about
`max-distance` in the
[official docs](https://cloud.google.com/compute/docs/instances/use-compact-placement-policies
).

You can use the `placement_max_distance` setting on the nodeset module to control the `max-distance` behavior. See the following example:

```yaml
  - id: nodeset
    source: community/modules/compute/schedmd-slurm-gcp-v6-nodeset
    use: [ network ]
    settings:
      machine_type: c2-standard-4
      node_count_dynamic_max: 30
      enable_placement: true
      placement_max_distance: 1

> [!NOTE]
> `schedmd-slurm-gcp-v6-nodeset.settings.enable_placement: true` must also be
> set for placement_max_distance to take effect.

In the above case using a value of 1 will restrict VM to be placed on the same
rack. You can confirm that the `max-distance` was applied by calling the
following command while jobs are running:

```shell
gcloud beta compute resource-policies list \
  --format='yaml(name,groupPlacementPolicy.maxDistance)'
```

> [!WARNING]
> If a zone lacks capacity, using a lower `max-distance` value (such as 1) is
> more likely to cause VMs creation to fail.

## TreeWidth and Node Communication

Slurm uses a fan out mechanism to communicate large groups of nodes. The shape
of this fan out tree is determined by the
[TreeWidth](https://slurm.schedmd.com/slurm.conf.html#OPT_TreeWidth)
configuration variable.

In the cloud, this fan out mechanism can become unstable when nodes restart with
new IP addresses. You can enforce that all nodes communicate directly with the
controller by setting TreeWidth to a value >= largest partition.

If the largest partition was 200 nodes, configure the blueprint as follows:

```yaml
  - id: slurm_controller
    source: community/modules/scheduler/schedmd-slurm-gcp-v6-controller
    ...
    settings:
      cloud_parameters:
        tree_width: 200
```

The default has been set to 128. Values above this have not been fully tested
and may cause congestion on the controller. A more scalable solution is under
way.

## Support
The Cluster Toolkit team maintains the wrapper around the [slurm-on-gcp] terraform
modules. For support with the underlying modules, see the instructions in the
[slurm-gcp README][slurm-gcp-readme].

[slurm-on-gcp]: https://github.com/GoogleCloudPlatform/slurm-gcp
[slurm-gcp-readme]: https://github.com/GoogleCloudPlatform/slurm-gcp#slurm-on-google-cloud-platform

## License

<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| additional\_disks | List of maps of disks. | <pre>list(object({<br>    disk_name    = string<br>    device_name  = string<br>    disk_type    = string<br>    disk_size_gb = number<br>    disk_labels  = map(string)<br>    auto_delete  = bool<br>    boot         = bool<br>  }))</pre> | `[]` | no |
| allow\_automatic\_updates | If false, disables automatic system package updates on the created instances.  This feature is<br>only available on supported images (or images derived from them).  For more details, see<br>https://cloud.google.com/compute/docs/instances/create-hpc-vm#disable_automatic_updates | `bool` | `true` | no |
| bandwidth\_tier | Configures the network interface card and the maximum egress bandwidth for VMs.<br>  - Setting `platform_default` respects the Google Cloud Platform API default values for networking.<br>  - Setting `virtio_enabled` explicitly selects the VirtioNet network adapter.<br>  - Setting `gvnic_enabled` selects the gVNIC network adapter (without Tier 1 high bandwidth).<br>  - Setting `tier_1_enabled` selects both the gVNIC adapter and Tier 1 high bandwidth networking.<br>  - Note: both gVNIC and Tier 1 networking require a VM image with gVNIC support as well as specific VM families and shapes.<br>  - See [official docs](https://cloud.google.com/compute/docs/networking/configure-vm-with-high-bandwidth-configuration) for more details. | `string` | `"platform_default"` | no |
| bucket\_dir | Bucket directory for cluster files to be put into. If not specified, then one will be chosen based on slurm\_cluster\_name. | `string` | `null` | no |
| bucket\_name | Name of GCS bucket.<br>Ignored when 'create\_bucket' is true. | `string` | `null` | no |
| can\_ip\_forward | Enable IP forwarding, for NAT instances for example. | `bool` | `false` | no |
| cgroup\_conf\_tpl | Slurm cgroup.conf template file path. | `string` | `null` | no |
| cloud\_parameters | cloud.conf options. Defaults inherited from [Slurm GCP repo](https://github.com/GoogleCloudPlatform/slurm-gcp/blob/master/terraform/slurm_cluster/modules/slurm_files/README_TF.md#input_cloud_parameters) | <pre>object({<br>    no_comma_params      = optional(bool, false)<br>    private_data         = optional(list(string))<br>    scheduler_parameters = optional(list(string))<br>    resume_rate          = optional(number)<br>    resume_timeout       = optional(number)<br>    suspend_rate         = optional(number)<br>    suspend_timeout      = optional(number)<br>    topology_plugin      = optional(string)<br>    topology_param       = optional(string)<br>    tree_width           = optional(number)<br>  })</pre> | `{}` | no |
| cloudsql | Use this database instead of the one on the controller.<br>  server\_ip : Address of the database server.<br>  user      : The user to access the database as.<br>  password  : The password, given the user, to access the given database. (sensitive)<br>  db\_name   : The database to access.<br>  user\_managed\_replication : The list of location and (optional) kms\_key\_name for secret | <pre>object({<br>    server_ip = string<br>    user      = string<br>    password  = string # sensitive<br>    db_name   = string<br>    user_managed_replication = optional(list(object({<br>      location     = string<br>      kms_key_name = optional(string)<br>    })), [])<br>  })</pre> | `null` | no |
| compute\_startup\_script | Startup script used by the compute VMs. | `string` | `"# no-op"` | no |
| compute\_startup\_scripts\_timeout | The timeout (seconds) applied to each script in compute\_startup\_scripts. If<br>any script exceeds this timeout, then the instance setup process is considered<br>failed and handled accordingly.<br><br>NOTE: When set to 0, the timeout is considered infinite and thus disabled. | `number` | `300` | no |
| controller\_startup\_script | Startup script used by the controller VM. | `string` | `"# no-op"` | no |
| controller\_startup\_scripts\_timeout | The timeout (seconds) applied to each script in controller\_startup\_scripts. If<br>any script exceeds this timeout, then the instance setup process is considered<br>failed and handled accordingly.<br><br>NOTE: When set to 0, the timeout is considered infinite and thus disabled. | `number` | `300` | no |
| create\_bucket | Create GCS bucket instead of using an existing one. | `bool` | `true` | no |
| deployment\_name | Name of the deployment. | `string` | n/a | yes |
| disable\_controller\_public\_ips | DEPRECATED: Use `enable_controller_public_ips` instead. | `bool` | `null` | no |
| disable\_default\_mounts | DEPRECATED: Use `enable_default_mounts` instead. | `bool` | `null` | no |
| disable\_smt | DEPRECATED: Use `enable_smt` instead. | `bool` | `null` | no |
| disk\_auto\_delete | Whether or not the boot disk should be auto-deleted. | `bool` | `true` | no |
| disk\_labels | Labels specific to the boot disk. These will be merged with var.labels. | `map(string)` | `{}` | no |
| disk\_size\_gb | Boot disk size in GB. | `number` | `50` | no |
| disk\_type | Boot disk type, can be either hyperdisk-balanced, pd-ssd, pd-standard, pd-balanced, or pd-extreme. | `string` | `"pd-ssd"` | no |
| enable\_bigquery\_load | Enables loading of cluster job usage into big query.<br><br>NOTE: Requires Google Bigquery API. | `bool` | `false` | no |
| enable\_cleanup\_compute | Enables automatic cleanup of compute nodes and resource policies (e.g.<br>placement groups) managed by this module, when cluster is destroyed.<br><br>*WARNING*: Toggling this off will impact the running workload.<br>Deployed compute nodes will be destroyed. | `bool` | `true` | no |
| enable\_confidential\_vm | Enable the Confidential VM configuration. Note: the instance image must support option. | `bool` | `false` | no |
| enable\_controller\_public\_ips | If set to true. The controller will have a random public IP assigned to it. Ignored if access\_config is set. | `bool` | `false` | no |
| enable\_debug\_logging | Enables debug logging mode. | `bool` | `false` | no |
| enable\_default\_mounts | Enable default global network storage from the controller<br>- /home<br>- /apps<br>Warning: If these are disabled, the slurm etc and munge dirs must be added<br>manually, or some other mechanism must be used to synchronize the slurm conf<br>files and the munge key across the cluster. | `bool` | `true` | no |
| enable\_devel | DEPRECATED: `enable_devel` is always on. | `bool` | `null` | no |
| enable\_external\_prolog\_epilog | Automatically enable a script that will execute prolog and epilog scripts<br>shared by NFS from the controller to compute nodes. Find more details at:<br>https://github.com/GoogleCloudPlatform/slurm-gcp/blob/master/tools/prologs-epilogs/README.md | `bool` | `null` | no |
| enable\_oslogin | Enables Google Cloud os-login for user login and authentication for VMs.<br>See https://cloud.google.com/compute/docs/oslogin | `bool` | `true` | no |
| enable\_shielded\_vm | Enable the Shielded VM configuration. Note: the instance image must support option. | `bool` | `false` | no |
| enable\_slurm\_gcp\_plugins | Enables calling hooks in scripts/slurm\_gcp\_plugins during cluster resume and suspend. | `any` | `false` | no |
| enable\_smt | Enables Simultaneous Multi-Threading (SMT) on instance. | `bool` | `false` | no |
| endpoint\_versions | Version of the API to use (The compute service is the only API currently supported) | <pre>object({<br>    compute = string<br>  })</pre> | <pre>{<br>  "compute": "beta"<br>}</pre> | no |
| epilog\_scripts | List of scripts to be used for Epilog. Programs for the slurmd to execute<br>on every node when a user's job completes.<br>See https://slurm.schedmd.com/slurm.conf.html#OPT_Epilog. | <pre>list(object({<br>    filename = string<br>    content  = optional(string)<br>    source   = optional(string)<br>  }))</pre> | `[]` | no |
| extra\_logging\_flags | The only available flag is `trace_api` | `map(bool)` | `{}` | no |
| gcloud\_path\_override | Directory of the gcloud executable to be used during cleanup | `string` | `""` | no |
| guest\_accelerator | List of the type and count of accelerator cards attached to the instance. | <pre>list(object({<br>    type  = string,<br>    count = number<br>  }))</pre> | `[]` | no |
| instance\_image | Defines the image that will be used in the Slurm controller VM instance.<br><br>Expected Fields:<br>name: The name of the image. Mutually exclusive with family.<br>family: The image family to use. Mutually exclusive with name.<br>project: The project where the image is hosted.<br><br>For more information on creating custom images that comply with Slurm on GCP<br>see the "Slurm on GCP Custom Images" section in docs/vm-images.md. | `map(string)` | <pre>{<br>  "family": "slurm-gcp-6-8-hpc-rocky-linux-8",<br>  "project": "schedmd-slurm-public"<br>}</pre> | no |
| instance\_image\_custom | A flag that designates that the user is aware that they are requesting<br>to use a custom and potentially incompatible image for this Slurm on<br>GCP module.<br><br>If the field is set to false, only the compatible families and project<br>names will be accepted.  The deployment will fail with any other image<br>family or name.  If set to true, no checks will be done.<br><br>See: https://goo.gle/hpc-slurm-images | `bool` | `false` | no |
| instance\_template | DEPRECATED: Instance template can not be specified for controller. | `string` | `null` | no |
| labels | Labels, provided as a map. | `map(string)` | `{}` | no |
| login\_network\_storage | An array of network attached storage mounts to be configured on all login nodes. | <pre>list(object({<br>    server_ip     = string,<br>    remote_mount  = string,<br>    local_mount   = string,<br>    fs_type       = string,<br>    mount_options = string,<br>  }))</pre> | `[]` | no |
| login\_nodes | List of slurm login instance definitions. | <pre>list(object({<br>    name_prefix = string<br>    access_config = optional(list(object({<br>      nat_ip       = string<br>      network_tier = string<br>    })))<br>    additional_disks = optional(list(object({<br>      disk_name    = optional(string)<br>      device_name  = optional(string)<br>      disk_size_gb = optional(number)<br>      disk_type    = optional(string)<br>      disk_labels  = optional(map(string), {})<br>      auto_delete  = optional(bool, true)<br>      boot         = optional(bool, false)<br>    })), [])<br>    additional_networks = optional(list(object({<br>      access_config = optional(list(object({<br>        nat_ip       = string<br>        network_tier = string<br>      })), [])<br>      alias_ip_range = optional(list(object({<br>        ip_cidr_range         = string<br>        subnetwork_range_name = string<br>      })), [])<br>      ipv6_access_config = optional(list(object({<br>        network_tier = string<br>      })), [])<br>      network            = optional(string)<br>      network_ip         = optional(string, "")<br>      nic_type           = optional(string)<br>      queue_count        = optional(number)<br>      stack_type         = optional(string)<br>      subnetwork         = optional(string)<br>      subnetwork_project = optional(string)<br>    })), [])<br>    bandwidth_tier         = optional(string, "platform_default")<br>    can_ip_forward         = optional(bool, false)<br>    disable_smt            = optional(bool, false)<br>    disk_auto_delete       = optional(bool, true)<br>    disk_labels            = optional(map(string), {})<br>    disk_size_gb           = optional(number)<br>    disk_type              = optional(string, "n1-standard-1")<br>    enable_confidential_vm = optional(bool, false)<br>    enable_oslogin         = optional(bool, true)<br>    enable_shielded_vm     = optional(bool, false)<br>    gpu = optional(object({<br>      count = number<br>      type  = string<br>    }))<br>    labels              = optional(map(string), {})<br>    machine_type        = optional(string)<br>    metadata            = optional(map(string), {})<br>    min_cpu_platform    = optional(string)<br>    num_instances       = optional(number, 1)<br>    on_host_maintenance = optional(string)<br>    preemptible         = optional(bool, false)<br>    region              = optional(string)<br>    service_account = optional(object({<br>      email  = optional(string)<br>      scopes = optional(list(string), ["https://www.googleapis.com/auth/cloud-platform"])<br>    }))<br>    shielded_instance_config = optional(object({<br>      enable_integrity_monitoring = optional(bool, true)<br>      enable_secure_boot          = optional(bool, true)<br>      enable_vtpm                 = optional(bool, true)<br>    }))<br>    source_image_family  = optional(string)<br>    source_image_project = optional(string)<br>    source_image         = optional(string)<br>    static_ips           = optional(list(string), [])<br>    subnetwork           = string<br>    spot                 = optional(bool, false)<br>    tags                 = optional(list(string), [])<br>    zone                 = optional(string)<br>    termination_action   = optional(string)<br>  }))</pre> | `[]` | no |
| login\_startup\_script | Startup script used by the login VMs. | `string` | `"# no-op"` | no |
| login\_startup\_scripts\_timeout | The timeout (seconds) applied to each script in login\_startup\_scripts. If<br>any script exceeds this timeout, then the instance setup process is considered<br>failed and handled accordingly.<br><br>NOTE: When set to 0, the timeout is considered infinite and thus disabled. | `number` | `300` | no |
| machine\_type | Machine type to create. | `string` | `"c2-standard-4"` | no |
| metadata | Metadata, provided as a map. | `map(string)` | `{}` | no |
| min\_cpu\_platform | Specifies a minimum CPU platform. Applicable values are the friendly names of<br>CPU platforms, such as Intel Haswell or Intel Skylake. See the complete list:<br>https://cloud.google.com/compute/docs/instances/specify-min-cpu-platform | `string` | `null` | no |
| network\_storage | An array of network attached storage mounts to be configured on all instances. | <pre>list(object({<br>    server_ip             = string,<br>    remote_mount          = string,<br>    local_mount           = string,<br>    fs_type               = string,<br>    mount_options         = string,<br>    client_install_runner = optional(map(string))<br>    mount_runner          = optional(map(string))<br>  }))</pre> | `[]` | no |
| nodeset | Define nodesets, as a list. | <pre>list(object({<br>    node_count_static      = optional(number, 0)<br>    node_count_dynamic_max = optional(number, 1)<br>    node_conf              = optional(map(string), {})<br>    nodeset_name           = string<br>    additional_disks = optional(list(object({<br>      disk_name    = optional(string)<br>      device_name  = optional(string)<br>      disk_size_gb = optional(number)<br>      disk_type    = optional(string)<br>      disk_labels  = optional(map(string), {})<br>      auto_delete  = optional(bool, true)<br>      boot         = optional(bool, false)<br>    })), [])<br>    bandwidth_tier                   = optional(string, "platform_default")<br>    can_ip_forward                   = optional(bool, false)<br>    disable_smt                      = optional(bool, false)<br>    disk_auto_delete                 = optional(bool, true)<br>    disk_labels                      = optional(map(string), {})<br>    disk_size_gb                     = optional(number)<br>    disk_type                        = optional(string)<br>    enable_confidential_vm           = optional(bool, false)<br>    enable_placement                 = optional(bool, false)<br>    placement_max_distance           = optional(number, null)<br>    enable_oslogin                   = optional(bool, true)<br>    enable_shielded_vm               = optional(bool, false)<br>    enable_maintenance_reservation   = optional(bool, false)<br>    enable_opportunistic_maintenance = optional(bool, false)<br>    gpu = optional(object({<br>      count = number<br>      type  = string<br>    }))<br>    dws_flex = object({<br>      enabled          = bool<br>      max_run_duration = number<br>      use_job_duration = bool<br>    })<br>    labels                   = optional(map(string), {})<br>    machine_type             = optional(string)<br>    maintenance_interval     = optional(string)<br>    instance_properties_json = string<br>    metadata                 = optional(map(string), {})<br>    min_cpu_platform         = optional(string)<br>    network_tier             = optional(string, "STANDARD")<br>    network_storage = optional(list(object({<br>      server_ip             = string<br>      remote_mount          = string<br>      local_mount           = string<br>      fs_type               = string<br>      mount_options         = string<br>      client_install_runner = optional(map(string))<br>      mount_runner          = optional(map(string))<br>    })), [])<br>    on_host_maintenance = optional(string)<br>    preemptible         = optional(bool, false)<br>    region              = optional(string)<br>    service_account = optional(object({<br>      email  = optional(string)<br>      scopes = optional(list(string), ["https://www.googleapis.com/auth/cloud-platform"])<br>    }))<br>    shielded_instance_config = optional(object({<br>      enable_integrity_monitoring = optional(bool, true)<br>      enable_secure_boot          = optional(bool, true)<br>      enable_vtpm                 = optional(bool, true)<br>    }))<br>    source_image_family  = optional(string)<br>    source_image_project = optional(string)<br>    source_image         = optional(string)<br>    subnetwork_self_link = string<br>    additional_networks = optional(list(object({<br>      network            = string<br>      subnetwork         = string<br>      subnetwork_project = string<br>      network_ip         = string<br>      nic_type           = string<br>      stack_type         = string<br>      queue_count        = number<br>      access_config = list(object({<br>        nat_ip       = string<br>        network_tier = string<br>      }))<br>      ipv6_access_config = list(object({<br>        network_tier = string<br>      }))<br>      alias_ip_range = list(object({<br>        ip_cidr_range         = string<br>        subnetwork_range_name = string<br>      }))<br>    })))<br>    access_config = optional(list(object({<br>      nat_ip       = string<br>      network_tier = string<br>    })))<br>    spot               = optional(bool, false)<br>    tags               = optional(list(string), [])<br>    termination_action = optional(string)<br>    reservation_name   = optional(string)<br>    future_reservation = string<br>    startup_script = optional(list(object({<br>      filename = string<br>    content = string })), [])<br><br>    zone_target_shape = string<br>    zone_policy_allow = set(string)<br>    zone_policy_deny  = set(string)<br>  }))</pre> | `[]` | no |
| nodeset\_dyn | Defines dynamic nodesets, as a list. | <pre>list(object({<br>    nodeset_name    = string<br>    nodeset_feature = string<br>  }))</pre> | `[]` | no |
| nodeset\_tpu | Define TPU nodesets, as a list. | <pre>list(object({<br>    node_count_static      = optional(number, 0)<br>    node_count_dynamic_max = optional(number, 5)<br>    nodeset_name           = string<br>    enable_public_ip       = optional(bool, false)<br>    node_type              = string<br>    accelerator_config = optional(object({<br>      topology = string<br>      version  = string<br>      }), {<br>      topology = ""<br>      version  = ""<br>    })<br>    tf_version   = string<br>    preemptible  = optional(bool, false)<br>    preserve_tpu = optional(bool, false)<br>    zone         = string<br>    data_disks   = optional(list(string), [])<br>    docker_image = optional(string, "")<br>    network_storage = optional(list(object({<br>      server_ip             = string<br>      remote_mount          = string<br>      local_mount           = string<br>      fs_type               = string<br>      mount_options         = string<br>      client_install_runner = optional(map(string))<br>      mount_runner          = optional(map(string))<br>    })), [])<br>    subnetwork = string<br>    service_account = optional(object({<br>      email  = optional(string)<br>      scopes = optional(list(string), ["https://www.googleapis.com/auth/cloud-platform"])<br>    }))<br>    project_id = string<br>    reserved   = optional(string, false)<br>  }))</pre> | `[]` | no |
| on\_host\_maintenance | Instance availability Policy. | `string` | `"MIGRATE"` | no |
| partitions | Cluster partitions as a list. See module slurm\_partition. | <pre>list(object({<br>    partition_name        = string<br>    partition_conf        = optional(map(string), {})<br>    partition_nodeset     = optional(list(string), [])<br>    partition_nodeset_dyn = optional(list(string), [])<br>    partition_nodeset_tpu = optional(list(string), [])<br>    enable_job_exclusive  = optional(bool, false)<br>  }))</pre> | n/a | yes |
| preemptible | Allow the instance to be preempted. | `bool` | `false` | no |
| project\_id | Project ID to create resources in. | `string` | n/a | yes |
| prolog\_scripts | List of scripts to be used for Prolog. Programs for the slurmd to execute<br>whenever it is asked to run a job step from a new job allocation.<br>See https://slurm.schedmd.com/slurm.conf.html#OPT_Prolog. | <pre>list(object({<br>    filename = string<br>    content  = optional(string)<br>    source   = optional(string)<br>  }))</pre> | `[]` | no |
| region | The default region to place resources in. | `string` | n/a | yes |
| service\_account | DEPRECATED: Use `service_account_email` and `service_account_scopes` instead. | <pre>object({<br>    email  = string<br>    scopes = set(string)<br>  })</pre> | `null` | no |
| service\_account\_email | Service account e-mail address to attach to the controller instance. | `string` | `null` | no |
| service\_account\_scopes | Scopes to attach to the controller instance. | `set(string)` | <pre>[<br>  "https://www.googleapis.com/auth/cloud-platform"<br>]</pre> | no |
| shielded\_instance\_config | Shielded VM configuration for the instance. Note: not used unless<br>enable\_shielded\_vm is 'true'.<br>  enable\_integrity\_monitoring : Compare the most recent boot measurements to the<br>  integrity policy baseline and return a pair of pass/fail results depending on<br>  whether they match or not.<br>  enable\_secure\_boot : Verify the digital signature of all boot components, and<br>  halt the boot process if signature verification fails.<br>  enable\_vtpm : Use a virtualized trusted platform module, which is a<br>  specialized computer chip you can use to encrypt objects like keys and<br>  certificates. | <pre>object({<br>    enable_integrity_monitoring = bool<br>    enable_secure_boot          = bool<br>    enable_vtpm                 = bool<br>  })</pre> | <pre>{<br>  "enable_integrity_monitoring": true,<br>  "enable_secure_boot": true,<br>  "enable_vtpm": true<br>}</pre> | no |
| slurm\_cluster\_name | Cluster name, used for resource naming and slurm accounting.<br>If not provided it will default to the first 8 characters of the deployment name (removing any invalid characters). | `string` | `null` | no |
| slurm\_conf\_tpl | Slurm slurm.conf template file path. | `string` | `null` | no |
| slurmdbd\_conf\_tpl | Slurm slurmdbd.conf template file path. | `string` | `null` | no |
| static\_ips | List of static IPs for VM instances. | `list(string)` | `[]` | no |
| subnetwork\_self\_link | Subnet to deploy to. | `string` | n/a | yes |
| tags | Network tag list. | `list(string)` | `[]` | no |
| universe\_domain | Domain address for alternate API universe | `string` | `"googleapis.com"` | no |
| zone | Zone where the instances should be created. If not specified, instances will be<br>spread across available zones in the region. | `string` | `null` | no |

## Outputs

| Name | Description |
|------|-------------|
| instructions | Post deployment instructions. |
| slurm\_bucket\_path | Bucket path used by cluster. |
| slurm\_cluster\_name | Slurm cluster name. |
| slurm\_controller\_instance | Compute instance of controller node |
| slurm\_login\_instances | Compute instances of login nodes |

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
