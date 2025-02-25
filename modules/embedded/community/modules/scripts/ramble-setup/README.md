## Description

This module will create a set of startup-script runners that will setup Ramble,
and install Ramble’s dependencies.

Ramble is a multi-platform experimentation framework capable of driving
software installation, acquiring input files, configuring experiments, and
extracting results. For more information about ramble, see:
https://github.com/GoogleCloudPlatform/ramble

This module outputs two startup script runners, which can be added to startup
scripts to setup, ramble and its dependencies.

For this module to be completely functional, it depends on a spack
installation. For more information, see Cluster-Toolkit’s Spack module.

> **_NOTE:_** This is an experimental module and the functionality and
> documentation will likely be updated in the near future. This module has only
> been tested in limited capacity.

# Examples

## Basic Example

```yaml
- id: ramble-setup
  source: community/modules/scripts/ramble-setup
```

This example simply installs ramble on a VM.

## Full Example

```yaml
- id: ramble-setup
  source: community/modules/scripts/ramble-setup
  settings:
    install_dir: /ramble
    ramble_url: https://github.com/GoogleCloudPlatform/ramble
    ramble_ref: v0.2.1
    log_file: /var/log/ramble.log
    chown_owner: “owner”
    chgrp_group: “user_group”
    chmod_mode: “a+r”
```

This example simply installs ramble into a VM at the location `/ramble`, checks
out the v0.2.1 tag, changes the owner and group to “owner” and “user_group”,
and chmod’s the clone to make it world readable.

Also see a more complete [Ramble example blueprint](../../../examples/ramble.yaml).

## License

<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| chmod\_mode | Mode to chmod the Ramble clone to. Defaults to `""` (i.e. do not modify).<br>For usage information see:<br>https://docs.ansible.com/ansible/latest/collections/ansible/builtin/file_module.html#parameter-mode | `string` | `""` | no |
| deployment\_name | Name of deployment, used to name bucket containing startup script. | `string` | n/a | yes |
| install\_dir | Destination directory of installation of Ramble. | `string` | `"/apps/ramble"` | no |
| labels | Key-value pairs of labels to be added to created resources. | `map(string)` | n/a | yes |
| project\_id | Project in which the HPC deployment will be created. | `string` | n/a | yes |
| ramble\_profile\_script\_path | Path to the Ramble profile.d script. Created by this module | `string` | `"/etc/profile.d/ramble.sh"` | no |
| ramble\_ref | Git ref to checkout for Ramble. | `string` | `"develop"` | no |
| ramble\_url | URL for Ramble repository to clone. | `string` | `"https://github.com/GoogleCloudPlatform/ramble"` | no |
| ramble\_virtualenv\_path | Virtual environment path in which to install Ramble Python interpreter and other dependencies | `string` | `"/usr/local/ramble-python"` | no |
| region | Region to place bucket containing startup script. | `string` | n/a | yes |
| system\_user\_gid | GID used when creating system user group. Ignored if `system_user_name` already exists on system. Default of 1104762904 is arbitrary. | `number` | `1104762904` | no |
| system\_user\_name | Name of system user that will perform installation of Ramble. It will be created if it does not exist. | `string` | `"ramble"` | no |
| system\_user\_uid | UID used when creating system user. Ignored if `system_user_name` already exists on system. Default of 1104762904 is arbitrary. | `number` | `1104762904` | no |

## Outputs

| Name | Description |
|------|-------------|
| controller\_startup\_script | Ramble installation script, duplicate for SLURM controller. |
| gcs\_bucket\_path | Bucket containing the startup scripts for Ramble, to be reused by ramble-execute module. |
| ramble\_path | Location ramble is installed into. |
| ramble\_profile\_script\_path | Path to Ramble profile script. |
| ramble\_ref | Git ref the ramble install is checked out to use |
| ramble\_runner | Runner to be used with startup-script module or passed to ramble-execute module.<br>- installs Ramble dependencies<br>- installs Ramble<br>- generates profile.d script to enable access to Ramble<br>This is safe to run in parallel by multiple machines. |
| startup\_script | Ramble installation script. |
| system\_user\_name | The system user used to install Ramble. It can be reused by ramble-execute module to execute Ramble commands. |

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
