## Description

This module will create a set of startup-script runners that will install and
run [DellHPC Omnia](https://github.com/dellhpc/omnia) version 1.3 onto a set of
VMs representing a slurm controller and compute nodes. For a full example using
omnia-install, see the [omnia-cluster example].

**Warning**: This module will create a user named "omnia" by default which has
sudo permissions. You may want to remove this user and/or it's permissions from
each node.

[omnia-cluster example]: ../../../../community/examples/omnia-cluster.yaml

## License

<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| compute\_ips | IPs of the Omnia compute nodes | `list(string)` | n/a | yes |
| install\_dir | Path where omnia will be installed, defaults to omnia user home directory (/home/omnia).<br>If specifying this path, please make sure it is on a shared file system, accessible by all omnia nodes. | `string` | `""` | no |
| manager\_ips | IPs of the Omnia manager nodes | `list(string)` | n/a | yes |
| omnia\_username | Name of the user that installs omnia | `string` | `"omnia"` | no |
| slurm\_uid | User ID of the slurm user | `number` | `981` | no |

## Outputs

| Name | Description |
|------|-------------|
| copy\_inventory\_runner | Runner to copy the inventory to the omnia manager using the startup-script module |
| install\_omnia\_runner | Runner to install Omnia using an ansible playbook. The startup-script module<br>will automatically handle installation of ansible.<br>- id: example-startup-script<br>  source: modules/scripts/startup-script<br>  settings:<br>    runners:<br>    - $(your-omnia-id.install\_omnia\_runner)<br>... |
| inventory\_file | The inventory file for the omnia cluster |
| omnia\_user\_warning | Warn developers that the omnia user was created with sudo permissions |
| setup\_omnia\_node\_runner | Runner to create the omnia user using an ansible playbook. The startup-script<br>module will automatically handle installation of ansible.<br>- id: example-startup-script<br>  source: modules/scripts/startup-script<br>  settings:<br>    runners:<br>    - $(your-omnia-id.setup\_omnia\_node\_runner)<br>... |
| setup\_omnia\_node\_script | An ansible script that adds the user that install omnia |

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
