# terraform-google-ai-on-gke

## Description

> [!NOTE]
> Slurm-gcp-v5-controller module is deprecated. See
> [this update](../../../../examples/README.md#completed-migration-to-slurm-gcp-v6)
> for specific recommendations and timelines.

This module creates a slurm controller node via the [SchedMD/slurm-gcp]
[slurm\_controller\_instance] and [slurm\_instance\_template] modules.

More information about Slurm On GCP can be found at the
[project's GitHub page][SchedMD/slurm-gcp] and in the
[Slurm on Google Cloud User Guide][slurm-ug].

The [user guide][slurm-ug] provides detailed instructions on customizing and
enhancing the Slurm on GCP cluster as well as recommendations on configuring the
controller for optimal performance at different scales.

> **Warning**: The variables `enable_reconfigure`,
> `enable_cleanup_compute`, and `enable_cleanup_subscriptions`, if set to
> `true`, require additional dependencies **to be installed on the system deploying the infrastructure**.
>
> ```shell
> # Install Python3 and run
> pip3 install -r https://raw.githubusercontent.com/GoogleCloudPlatform/slurm-gcp/5.12.0/scripts/requirements.txt
> ```

[SchedMD/slurm-gcp]: https://github.com/GoogleCloudPlatform/slurm-gcp/tree/5.12.0
[slurm\_controller\_instance]: https://github.com/GoogleCloudPlatform/slurm-gcp/tree/5.12.0/terraform/slurm_cluster/modules/slurm_controller_instance
[slurm\_instance\_template]: https://github.com/GoogleCloudPlatform/slurm-gcp/tree/5.12.0/terraform/slurm_cluster/modules/slurm_instance_template
[slurm-ug]: https://goo.gle/slurm-gcp-user-guide.
[requirements.txt]: https://github.com/GoogleCloudPlatform/slurm-gcp/blob/5.12.0/scripts/requirements.txt
[enable\_cleanup\_compute]: #input\_enable\_cleanup\_compute
[enable\_cleanup\_subscriptions]: #input\_enable\_cleanup\_subscriptions
[enable\_reconfigure]: #input\_enable\_reconfigure

### Example

```yaml
- id: slurm_controller
  source: community/modules/scheduler/schedmd-slurm-gcp-v5-controller
  use:
  - network1
  - homefs
  - compute_partition
  settings:
    machine_type: c2-standard-8
```

This creates a controller node with the following attributes:

* connected to the primary subnetwork of `network1`
* the filesystem with the ID `homefs` (defined elsewhere in the blueprint)
  mounted
* One partition with the ID `compute_partition` (defined elsewhere in the
  blueprint)
* machine type upgraded from the default `c2-standard-4` to `c2-standard-8`

For a complete example using this module, see
[slurm-gcp-v5-cluster.yaml](../../../examples/slurm-gcp-v5-cluster.yaml).

### Live Cluster Reconfiguration (`enable_reconfigure`)

The schedmd-slurm-gcp-v5-controller module supports the reconfiguration of
partitions and slurm configuration in a running, active cluster. This option is
activated through the `enable_reconfigure` setting:

```yaml
- id: slurm_controller
  source: community/modules/scheduler/schedmd-slurm-gcp-v5-controller
  settings:
    enable_reconfigure: true
```

To reconfigure a running cluster:

1. Edit the blueprint with the desired configuration changes
1. Call `gcluster create <blueprint> -w` to overwrite the deployment directory
1. Follow instructions in terminal to deploy

The following are examples of updates that can be made to a running cluster:

* Add or remove a partition to the cluster
* Resize an existing partition
* Attach new network storage to an existing partition

> **NOTE**: Changing the VM `machine_type` of a partition may not work with
> `enable_reconfigure`. It is better to create a new partition and delete the
> old one.

This option has some additional requirements:

* The Pub/Sub API must be activated in the target project:
  `gcloud services enable pubsub.googleapis.com --project "<<PROJECT_ID>>"`
* The authenticated user in the local development environment (or where
  `terraform apply` is called) must have the Pub/Sub Admin (roles/pubsub.admin)
  IAM role.
* Python and some python packages need to be installed with pip in the local
  development environment deploying the cluster. One can use following commands:

  ```bash
  pip3 install -r https://raw.githubusercontent.com/GoogleCloudPlatform/slurm-gcp/5.12.0/scripts/requirements.txt
  ```

  For more information, see the [description][optdeps] of this module.

[optdeps]: https://github.com/GoogleCloudPlatform/slurm-gcp/tree/5.12.0/terraform/slurm_cluster#optional

## Custom Images

For more information on creating valid custom images for the controller VM
instance or for custom instance templates, see our [vm-images.md] documentation
page.

[vm-images.md]: ../../../../docs/vm-images.md#slurm-on-gcp-custom-images

## GPU Support

More information on GPU support in Slurm on GCP and other Cluster Toolkit modules
can be found at [docs/gpu-support.md](../../../../docs/gpu-support.md)

## Placement Max Distance

When using
[enable_placement](../../compute/schedmd-slurm-gcp-v5-partition/README.md#input_enable_placement)
with Slurm, Google Compute Engine will attempt to place VMs as physically close
together as possible. Capacity constraints at the time of VM creation may still
force VMs to be spread across multiple racks. Google provides the `max-distance`
flag which can used to control the maximum spreading allowed. Read more about
`max-distance` in the
[official docs](https://cloud.google.com/compute/docs/instances/use-compact-placement-policies
).

After deploying a Slurm cluster, you can use the following steps to manually
configure the max-distance parameter.

1. Make sure your blueprint has `enable_placement: true` setting for Slurm
   partitions.
2. Deploy the Slurm cluster and wait for the deployment to complete.
3. SSH to the deployed Slurm controller
4. Apply the following edit to `/slurm/scripts/config.yaml`:

    ```yaml
    # Replace
    enable_slurm_gcp_plugins: false

    # With
    enable_slurm_gcp_plugins:
      max_hops:
        max_hops: 1
    ```

The `max_hops` parameter will be used for the `max-distance` argument. In the
above case using a value of 1 will restrict VM to be placed on the same rack.

You can confirm that the `max-distance`` was applied by calling the following
command while jobs are running:

```shell
gcloud beta compute resource-policies list \
  --format='yaml(name,groupPlacementPolicy.maxDistance)'
```

> [!WARNING]
> If a zone lacks capacity, using a lower `max-distance` value (such as 1) is
> more likely to cause VMs creation to fail.

<!---->

> [!WARNING]
> `/slurm/scripts/config.yaml` will be overwritten if the blueprint is
> re-deployed using the `enable_reconfigure` flag.

## Hybrid Slurm Clusters
For more information on how to configure an on premise slurm cluster with hybrid
cloud partitions, see the [schedmd-slurm-gcp-v5-hybrid] module and our
extended instructions in our [docs](../../../../docs/hybrid-slurm-cluster/).

[schedmd-slurm-gcp-v5-hybrid]: ../schedmd-slurm-gcp-v5-hybrid/README.md

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
| access\_config | Access configurations, i.e. IPs via which the VM instance can be accessed via the Internet. | <pre>list(object({<br>    nat_ip       = string<br>    network_tier = string<br>  }))</pre> | `[]` | no |
| additional\_disks | List of maps of disks. | <pre>list(object({<br>    disk_name    = string<br>    device_name  = string<br>    disk_type    = string<br>    disk_size_gb = number<br>    disk_labels  = map(string)<br>    auto_delete  = bool<br>    boot         = bool<br>  }))</pre> | `[]` | no |
| allow\_automatic\_updates | If false, disables automatic system package updates on the created instances.  This feature is<br>only available on supported images (or images derived from them).  For more details, see<br>https://cloud.google.com/compute/docs/instances/create-hpc-vm#disable_automatic_updates | `bool` | `true` | no |
| can\_ip\_forward | Enable IP forwarding, for NAT instances for example. | `bool` | `false` | no |
| cgroup\_conf\_tpl | Slurm cgroup.conf template file path. | `string` | `null` | no |
| cloud\_parameters | cloud.conf options. | <pre>object({<br>    no_comma_params = bool<br>    resume_rate     = number<br>    resume_timeout  = number<br>    suspend_rate    = number<br>    suspend_timeout = number<br>  })</pre> | <pre>{<br>  "no_comma_params": false,<br>  "resume_rate": 0,<br>  "resume_timeout": 300,<br>  "suspend_rate": 0,<br>  "suspend_timeout": 300<br>}</pre> | no |
| cloudsql | Use this database instead of the one on the controller.<br>  server\_ip : Address of the database server.<br>  user      : The user to access the database as.<br>  password  : The password, given the user, to access the given database. (sensitive)<br>  db\_name   : The database to access. | <pre>object({<br>    server_ip = string<br>    user      = string<br>    password  = string # sensitive<br>    db_name   = string<br>  })</pre> | `null` | no |
| compute\_startup\_script | Startup script used by the compute VMs. | `string` | `""` | no |
| compute\_startup\_scripts\_timeout | The timeout (seconds) applied to the compute\_startup\_script. If<br>any script exceeds this timeout, then the instance setup process is considered<br>failed and handled accordingly.<br><br>NOTE: When set to 0, the timeout is considered infinite and thus disabled. | `number` | `300` | no |
| controller\_startup\_script | Startup script used by the controller VM. | `string` | `""` | no |
| controller\_startup\_scripts\_timeout | The timeout (seconds) applied to the controller\_startup\_script. If<br>any script exceeds this timeout, then the instance setup process is considered<br>failed and handled accordingly.<br><br>NOTE: When set to 0, the timeout is considered infinite and thus disabled. | `number` | `300` | no |
| deployment\_name | Name of the deployment. | `string` | n/a | yes |
| disable\_controller\_public\_ips | If set to false. The controller will have a random public IP assigned to it. Ignored if access\_config is set. | `bool` | `true` | no |
| disable\_default\_mounts | Disable default global network storage from the controller<br>- /usr/local/etc/slurm<br>- /etc/munge<br>- /home<br>- /apps<br>Warning: If these are disabled, the slurm etc and munge dirs must be added<br>manually, or some other mechanism must be used to synchronize the slurm conf<br>files and the munge key across the cluster. | `bool` | `false` | no |
| disable\_smt | Disables Simultaneous Multi-Threading (SMT) on instance. | `bool` | `true` | no |
| disk\_auto\_delete | Whether or not the boot disk should be auto-deleted. | `bool` | `true` | no |
| disk\_labels | Labels specific to the boot disk. These will be merged with var.labels. | `map(string)` | `{}` | no |
| disk\_size\_gb | Boot disk size in GB. | `number` | `50` | no |
| disk\_type | Boot disk type. | `string` | `"pd-ssd"` | no |
| enable\_bigquery\_load | Enable loading of cluster job usage into big query. | `bool` | `false` | no |
| enable\_cleanup\_compute | Enables automatic cleanup of compute nodes and resource policies (e.g.<br>placement groups) managed by this module, when cluster is destroyed.<br><br>NOTE: Requires Python and pip packages listed at the following link:<br>https://github.com/GoogleCloudPlatform/slurm-gcp/blob/3979e81fc5e4f021b5533a23baa474490f4f3614/scripts/requirements.txt<br><br>*WARNING*: Toggling this may impact the running workload. Deployed compute nodes<br>may be destroyed and their jobs will be requeued. | `bool` | `false` | no |
| enable\_cleanup\_subscriptions | Enables automatic cleanup of pub/sub subscriptions managed by this module, when<br>cluster is destroyed.<br><br>NOTE: Requires Python and pip packages listed at the following link:<br>https://github.com/GoogleCloudPlatform/slurm-gcp/blob/3979e81fc5e4f021b5533a23baa474490f4f3614/scripts/requirements.txt<br><br>*WARNING*: Toggling this may temporarily impact var.enable\_reconfigure behavior. | `bool` | `false` | no |
| enable\_confidential\_vm | Enable the Confidential VM configuration. Note: the instance image must support option. | `bool` | `false` | no |
| enable\_devel | Enables development mode. Not for production use. | `bool` | `false` | no |
| enable\_external\_prolog\_epilog | Automatically enable a script that will execute prolog and epilog scripts<br>shared under /opt/apps from the controller to compute nodes. | `bool` | `false` | no |
| enable\_oslogin | Enables Google Cloud os-login for user login and authentication for VMs.<br>See https://cloud.google.com/compute/docs/oslogin | `bool` | `true` | no |
| enable\_reconfigure | Enables automatic Slurm reconfiguration when Slurm configuration changes (e.g.<br>slurm.conf.tpl, partition details). Compute instances and resource policies<br>(e.g. placement groups) will be destroyed to align with new configuration.<br>NOTE: Requires Python and Google Pub/Sub API.<br>*WARNING*: Toggling this will impact the running workload. Deployed compute nodes<br>will be destroyed and their jobs will be requeued. | `bool` | `false` | no |
| enable\_shielded\_vm | Enable the Shielded VM configuration. Note: the instance image must support option. | `bool` | `false` | no |
| enable\_slurm\_gcp\_plugins | Enables calling hooks in scripts/slurm\_gcp\_plugins during cluster resume and suspend. | `any` | `false` | no |
| epilog\_scripts | List of scripts to be used for Epilog. Programs for the slurmd to execute<br>on every node when a user's job completes.<br>See https://slurm.schedmd.com/slurm.conf.html#OPT_Epilog. | <pre>list(object({<br>    filename = string<br>    content  = string<br>  }))</pre> | `[]` | no |
| gpu | DEPRECATED: use var.guest\_accelerator | <pre>object({<br>    type  = string<br>    count = number<br>  })</pre> | `null` | no |
| guest\_accelerator | List of the type and count of accelerator cards attached to the instance. | <pre>list(object({<br>    type  = string,<br>    count = number<br>  }))</pre> | `[]` | no |
| instance\_image | Defines the image that will be used in the Slurm controller VM instance.<br><br>Expected Fields:<br>name: The name of the image. Mutually exclusive with family.<br>family: The image family to use. Mutually exclusive with name.<br>project: The project where the image is hosted.<br><br>For more information on creating custom images that comply with Slurm on GCP<br>see the "Slurm on GCP Custom Images" section in docs/vm-images.md. | `map(string)` | <pre>{<br>  "family": "slurm-gcp-5-12-hpc-centos-7",<br>  "project": "schedmd-slurm-public"<br>}</pre> | no |
| instance\_image\_custom | A flag that designates that the user is aware that they are requesting<br>to use a custom and potentially incompatible image for this Slurm on<br>GCP module.<br><br>If the field is set to false, only the compatible families and project<br>names will be accepted.  The deployment will fail with any other image<br>family or name.  If set to true, no checks will be done.<br><br>See: https://goo.gle/hpc-slurm-images | `bool` | `false` | no |
| instance\_template | Self link to a custom instance template. If set, other VM definition<br>variables such as machine\_type and instance\_image will be ignored in favor<br>of the provided instance template.<br><br>For more information on creating custom images for the instance template<br>that comply with Slurm on GCP see the "Slurm on GCP Custom Images" section<br>in docs/vm-images.md. | `string` | `null` | no |
| labels | Labels, provided as a map. | `map(string)` | `{}` | no |
| login\_startup\_scripts\_timeout | The timeout (seconds) applied to the login startup script. If<br>any script exceeds this timeout, then the instance setup process is considered<br>failed and handled accordingly.<br><br>NOTE: When set to 0, the timeout is considered infinite and thus disabled. | `number` | `300` | no |
| machine\_type | Machine type to create. | `string` | `"c2-standard-4"` | no |
| metadata | Metadata, provided as a map. | `map(string)` | `{}` | no |
| min\_cpu\_platform | Specifies a minimum CPU platform. Applicable values are the friendly names of<br>CPU platforms, such as Intel Haswell or Intel Skylake. See the complete list:<br>https://cloud.google.com/compute/docs/instances/specify-min-cpu-platform | `string` | `null` | no |
| network\_ip | DEPRECATED: Use `static_ips` variable to assign an internal static ip address. | `string` | `null` | no |
| network\_self\_link | Network to deploy to. Either network\_self\_link or subnetwork\_self\_link must be specified. | `string` | `null` | no |
| network\_storage | An array of network attached storage mounts to be configured on all instances. | <pre>list(object({<br>    server_ip             = string,<br>    remote_mount          = string,<br>    local_mount           = string,<br>    fs_type               = string,<br>    mount_options         = string,<br>    client_install_runner = map(string)<br>    mount_runner          = map(string)<br>  }))</pre> | `[]` | no |
| on\_host\_maintenance | Instance availability Policy. | `string` | `"MIGRATE"` | no |
| partition | Cluster partitions as a list. | <pre>list(object({<br>    compute_list = list(string)<br>    partition = object({<br>      enable_job_exclusive    = bool<br>      enable_placement_groups = bool<br>      network_storage = list(object({<br>        server_ip     = string<br>        remote_mount  = string<br>        local_mount   = string<br>        fs_type       = string<br>        mount_options = string<br>      }))<br>      partition_conf    = map(string)<br>      partition_feature = string<br>      partition_name    = string<br>      partition_nodes = map(object({<br>        access_config = list(object({<br>          network_tier = string<br>        }))<br>        bandwidth_tier         = string<br>        node_count_dynamic_max = number<br>        node_count_static      = number<br>        enable_spot_vm         = bool<br>        group_name             = string<br>        instance_template      = string<br>        maintenance_interval   = string<br>        node_conf              = map(string)<br>        reservation_name       = string<br>        spot_instance_config = object({<br>          termination_action = string<br>        })<br>      }))<br>      partition_startup_scripts_timeout = number<br>      subnetwork                        = string<br>      zone_policy_allow                 = list(string)<br>      zone_policy_deny                  = list(string)<br>      zone_target_shape                 = string<br>    })<br>  }))</pre> | `[]` | no |
| preemptible | Allow the instance to be preempted. | `bool` | `false` | no |
| project\_id | Project ID to create resources in. | `string` | n/a | yes |
| prolog\_scripts | List of scripts to be used for Prolog. Programs for the slurmd to execute<br>whenever it is asked to run a job step from a new job allocation.<br>See https://slurm.schedmd.com/slurm.conf.html#OPT_Prolog. | <pre>list(object({<br>    filename = string<br>    content  = string<br>  }))</pre> | `[]` | no |
| region | Region where the instances should be created. | `string` | `null` | no |
| service\_account | Service account to attach to the controller instance. If not set, the<br>default compute service account for the given project will be used with the<br>"https://www.googleapis.com/auth/cloud-platform" scope. | <pre>object({<br>    email  = string<br>    scopes = set(string)<br>  })</pre> | `null` | no |
| shielded\_instance\_config | Shielded VM configuration for the instance. Note: not used unless<br>enable\_shielded\_vm is 'true'.<br>  enable\_integrity\_monitoring : Compare the most recent boot measurements to the<br>  integrity policy baseline and return a pair of pass/fail results depending on<br>  whether they match or not.<br>  enable\_secure\_boot : Verify the digital signature of all boot components, and<br>  halt the boot process if signature verification fails.<br>  enable\_vtpm : Use a virtualized trusted platform module, which is a<br>  specialized computer chip you can use to encrypt objects like keys and<br>  certificates. | <pre>object({<br>    enable_integrity_monitoring = bool<br>    enable_secure_boot          = bool<br>    enable_vtpm                 = bool<br>  })</pre> | <pre>{<br>  "enable_integrity_monitoring": true,<br>  "enable_secure_boot": true,<br>  "enable_vtpm": true<br>}</pre> | no |
| slurm\_cluster\_name | Cluster name, used for resource naming and slurm accounting. If not provided it will default to the first 8 characters of the deployment name (removing any invalid characters). | `string` | `null` | no |
| slurm\_conf\_tpl | Slurm slurm.conf template file path. | `string` | `null` | no |
| slurmdbd\_conf\_tpl | Slurm slurmdbd.conf template file path. | `string` | `null` | no |
| source\_image | DEPRECATED: Use `instance_image` instead. | `string` | `null` | no |
| source\_image\_family | DEPRECATED: Use `instance_image` instead. | `string` | `null` | no |
| source\_image\_project | DEPRECATED: Use `instance_image` instead. | `string` | `null` | no |
| static\_ips | List of static IPs for VM instances. | `list(string)` | `[]` | no |
| subnetwork\_project | The project that subnetwork belongs to. | `string` | `null` | no |
| subnetwork\_self\_link | Subnet to deploy to. Either network\_self\_link or subnetwork\_self\_link must be specified. | `string` | `null` | no |
| tags | Network tag list. | `list(string)` | `[]` | no |
| zone | Zone where the instances should be created. If not specified, instances will be<br>spread across available zones in the region. | `string` | `null` | no |

## Outputs

| Name | Description |
|------|-------------|
| cloud\_logging\_filter | Cloud Logging filter to cluster errors. |
| controller\_instance\_id | The server-assigned unique identifier of the controller compute instance. |
| pubsub\_topic | Cluster Pub/Sub topic. |

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
