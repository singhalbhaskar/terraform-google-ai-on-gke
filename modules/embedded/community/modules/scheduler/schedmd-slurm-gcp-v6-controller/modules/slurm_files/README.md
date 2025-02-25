<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| bucket\_dir | Bucket directory for cluster files to be put into. | `string` | `null` | no |
| bucket\_name | Name of GCS bucket to use. | `string` | n/a | yes |
| cgroup\_conf\_tpl | Slurm cgroup.conf template file path. | `string` | `null` | no |
| cloud\_parameters | cloud.conf options. Default behavior defined in scripts/conf.py | <pre>object({<br>    no_comma_params      = optional(bool, false)<br>    private_data         = optional(list(string))<br>    scheduler_parameters = optional(list(string))<br>    resume_rate          = optional(number)<br>    resume_timeout       = optional(number)<br>    suspend_rate         = optional(number)<br>    suspend_timeout      = optional(number)<br>    topology_plugin      = optional(string)<br>    topology_param       = optional(string)<br>    tree_width           = optional(number)<br>  })</pre> | `{}` | no |
| cloudsql\_secret | Secret URI to cloudsql secret. | `string` | `null` | no |
| compute\_startup\_scripts | List of scripts to be ran on compute VM startup. | <pre>list(object({<br>    filename = string<br>    content  = string<br>  }))</pre> | `[]` | no |
| compute\_startup\_scripts\_timeout | The timeout (seconds) applied to each script in compute\_startup\_scripts. If<br>any script exceeds this timeout, then the instance setup process is considered<br>failed and handled accordingly.<br><br>NOTE: When set to 0, the timeout is considered infinite and thus disabled. | `number` | `300` | no |
| controller\_startup\_scripts | List of scripts to be ran on controller VM startup. | <pre>list(object({<br>    filename = string<br>    content  = string<br>  }))</pre> | `[]` | no |
| controller\_startup\_scripts\_timeout | The timeout (seconds) applied to each script in controller\_startup\_scripts. If<br>any script exceeds this timeout, then the instance setup process is considered<br>failed and handled accordingly.<br><br>NOTE: When set to 0, the timeout is considered infinite and thus disabled. | `number` | `300` | no |
| disable\_default\_mounts | Disable default global network storage from the controller<br>- /usr/local/etc/slurm<br>- /etc/munge<br>- /home<br>- /apps<br>If these are disabled, the slurm etc and munge dirs must be added manually,<br>or some other mechanism must be used to synchronize the slurm conf files<br>and the munge key across the cluster. | `bool` | `false` | no |
| enable\_bigquery\_load | Enables loading of cluster job usage into big query.<br><br>NOTE: Requires Google Bigquery API. | `bool` | `false` | no |
| enable\_debug\_logging | Enables debug logging mode. Not for production use. | `bool` | `false` | no |
| enable\_external\_prolog\_epilog | Automatically enable a script that will execute prolog and epilog scripts<br>shared by NFS from the controller to compute nodes. Find more details at:<br>https://github.com/GoogleCloudPlatform/slurm-gcp/blob/v5/tools/prologs-epilogs/README.md | `bool` | `false` | no |
| enable\_hybrid | Enables use of hybrid controller mode. When true, controller\_hybrid\_config will<br>be used instead of controller\_instance\_config and will disable login instances. | `bool` | `false` | no |
| enable\_slurm\_gcp\_plugins | Enables calling hooks in scripts/slurm\_gcp\_plugins during cluster resume and suspend. | `any` | `false` | no |
| endpoint\_versions | Version of the API to use (The compute service is the only API currently supported) | <pre>object({<br>    compute = string<br>  })</pre> | <pre>{<br>  "compute": null<br>}</pre> | no |
| epilog\_scripts | List of scripts to be used for Epilog. Programs for the slurmd to execute<br>on every node when a user's job completes.<br>See https://slurm.schedmd.com/slurm.conf.html#OPT_Epilog. | <pre>list(object({<br>    filename = string<br>    content  = optional(string)<br>    source   = optional(string)<br>  }))</pre> | `[]` | no |
| extra\_logging\_flags | The only available flag is `trace_api` | `map(bool)` | `{}` | no |
| google\_app\_cred\_path | Path to Google Application Credentials. | `string` | `null` | no |
| install\_dir | Directory where the hybrid configuration directory will be installed on the<br>on-premise controller (e.g. /etc/slurm/hybrid). This updates the prefix path<br>for the resume and suspend scripts in the generated `cloud.conf` file.<br><br>This variable should be used when the TerraformHost and the SlurmctldHost<br>are different.<br><br>This will default to var.output\_dir if null. | `string` | `null` | no |
| login\_network\_storage | Storage to mounted on login and controller instances<br>- server\_ip     : Address of the storage server.<br>- remote\_mount  : The location in the remote instance filesystem to mount from.<br>- local\_mount   : The location on the instance filesystem to mount to.<br>- fs\_type       : Filesystem type (e.g. "nfs").<br>- mount\_options : Options to mount with. | <pre>list(object({<br>    server_ip     = string<br>    remote_mount  = string<br>    local_mount   = string<br>    fs_type       = string<br>    mount_options = string<br>  }))</pre> | `[]` | no |
| login\_startup\_scripts | List of scripts to be ran on login VM startup. | <pre>list(object({<br>    filename = string<br>    content  = string<br>  }))</pre> | `[]` | no |
| login\_startup\_scripts\_timeout | The timeout (seconds) applied to each script in login\_startup\_scripts. If<br>any script exceeds this timeout, then the instance setup process is considered<br>failed and handled accordingly.<br><br>NOTE: When set to 0, the timeout is considered infinite and thus disabled. | `number` | `300` | no |
| munge\_mount | Remote munge mount for compute and login nodes to acquire the munge.key.<br><br>By default, the munge mount server will be assumed to be the<br>`var.slurm_control_host` (or `var.slurm_control_addr` if non-null) when<br>`server_ip=null`. | <pre>object({<br>    server_ip     = string<br>    remote_mount  = string<br>    fs_type       = string<br>    mount_options = string<br>  })</pre> | <pre>{<br>  "fs_type": "nfs",<br>  "mount_options": "",<br>  "remote_mount": "/etc/munge/",<br>  "server_ip": null<br>}</pre> | no |
| network\_storage | Storage to mounted on all instances.<br>- server\_ip     : Address of the storage server.<br>- remote\_mount  : The location in the remote instance filesystem to mount from.<br>- local\_mount   : The location on the instance filesystem to mount to.<br>- fs\_type       : Filesystem type (e.g. "nfs").<br>- mount\_options : Options to mount with. | <pre>list(object({<br>    server_ip     = string<br>    remote_mount  = string<br>    local_mount   = string<br>    fs_type       = string<br>    mount_options = string<br>  }))</pre> | `[]` | no |
| nodeset | Cluster nodenets, as a list. | `list(any)` | `[]` | no |
| nodeset\_dyn | Cluster nodenets (dynamic), as a list. | `list(any)` | `[]` | no |
| nodeset\_startup\_scripts | List of scripts to be ran on compute VM startup in the specific nodeset. | <pre>map(list(object({<br>    filename = string<br>    content  = string<br>  })))</pre> | `{}` | no |
| nodeset\_tpu | Cluster nodenets (TPU), as a list. | `list(any)` | `[]` | no |
| output\_dir | Directory where this module will write its files to. These files include:<br>cloud.conf; cloud\_gres.conf; config.yaml; resume.py; suspend.py; and util.py. | `string` | `null` | no |
| partitions | Cluster partitions as a list. | `list(any)` | `[]` | no |
| project\_id | The GCP project ID. | `string` | n/a | yes |
| prolog\_scripts | List of scripts to be used for Prolog. Programs for the slurmd to execute<br>whenever it is asked to run a job step from a new job allocation.<br>See https://slurm.schedmd.com/slurm.conf.html#OPT_Prolog. | <pre>list(object({<br>    filename = string<br>    content  = optional(string)<br>    source   = optional(string)<br>  }))</pre> | `[]` | no |
| slurm\_bin\_dir | Path to directory of Slurm binary commands (e.g. scontrol, sinfo). If 'null',<br>then it will be assumed that binaries are in $PATH. | `string` | `null` | no |
| slurm\_cluster\_name | The cluster name, used for resource naming and slurm accounting. | `string` | n/a | yes |
| slurm\_conf\_tpl | Slurm slurm.conf template file path. | `string` | `null` | no |
| slurm\_control\_addr | The IP address or a name by which the address can be identified.<br><br>This value is passed to slurm.conf such that:<br>SlurmctldHost={var.slurm\_control\_host}\({var.slurm\_control\_addr}\)<br><br>See https://slurm.schedmd.com/slurm.conf.html#OPT_SlurmctldHost | `string` | `null` | no |
| slurm\_control\_host | The short, or long, hostname of the machine where Slurm control daemon is<br>executed (i.e. the name returned by the command "hostname -s").<br><br>This value is passed to slurm.conf such that:<br>SlurmctldHost={var.slurm\_control\_host}\({var.slurm\_control\_addr}\)<br><br>See https://slurm.schedmd.com/slurm.conf.html#OPT_SlurmctldHost | `string` | `null` | no |
| slurm\_control\_host\_port | The port number that the Slurm controller, slurmctld, listens to for work.<br><br>See https://slurm.schedmd.com/slurm.conf.html#OPT_SlurmctldPort | `string` | `"6818"` | no |
| slurm\_log\_dir | Directory where Slurm logs to. | `string` | `"/var/log/slurm"` | no |
| slurmdbd\_conf\_tpl | Slurm slurmdbd.conf template file path. | `string` | `null` | no |

## Outputs

| Name | Description |
|------|-------------|
| config | Cluster configuration. |
| nodeset | Cluster nodesets. |
| nodeset\_dyn | Cluster nodesets (dynamic). |
| nodeset\_tpu | Cluster nodesets (TPU). |
| partitions | Cluster partitions. |
| slurm\_bucket\_path | GCS Bucket URI of Slurm cluster file storage. |

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
