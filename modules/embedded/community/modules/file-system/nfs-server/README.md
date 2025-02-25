# terraform-google-ai-on-gke

## Description

This module creates a Network File Sharing (NFS) file system based on a VM
instance and [compute disk][disk]. This file system can share directories and
files with other clients over a network. `nfs-server` can be used by
[vm-instance](../../../../modules/compute/vm-instance/README.md) and SchedMD
community modules that create compute VMs.

For more information on this and other network storage options in the Cluster
Toolkit, see the extended [Network Storage documentation](../../../../docs/network_storage.md).

> **_WARNING:_** This module has only been tested against the HPC centos7 OS
> disk image (the default). Using other images may work, but have not been
> verified.

[disk]: https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/compute_disk

### Example

```yaml
- id: homefs
  source: community/modules/file-system/nfs-server
  use: [network1]
  settings:
    auto_delete_disk: true
```

This creates a NFS on a virtual machine which allow other VMs to mount the
volume as an external file system.

> **_NOTE:_** `auto_delete_disk` is set to true in this example, which means
> that running `terraform destroy` also deletes the disk. To retain the disk
> after `terraform destroy` either set this to false or don't include the
> settings so it defaults to false. Note that with `auto_delete_disk: false`,
> you will need to manually delete the disk after destroying a deployment group
> with `nfs-server`.

## Mounting

To mount the NFS Server you must first ensure that the NFS client has been
installed the and then call the proper `mount` command.

Both of these steps are automatically handled with the use of the `use` command
in a selection of Cluster Toolkit modules. See the [compatibility matrix][matrix] in
the network storage doc for a complete list of supported modules.
See the [hpc-centos-ss.yaml] test config for an example of using this module
with a `vm-instance` module.

If mounting is not automatically handled as described above, the `nfs-server`
module outputs runners that can be used with the startup-script module to
install the client and mount the file system. See the following example:

```yaml
  - id: nfs
    source: community/modules/file-system/nfs-server
    use: [network1]
    settings: {local_mounts: [/mnt1]}

  - id: mount-at-startup
    source: modules/scripts/startup-script
    settings:
      runners:
      - $(nfs.install_nfs_client_runner)
      - $(nfs.mount_runner)

```

[hpc-centos-ss.yaml]: ../../../../tools/validate_configs/test_configs/hpc-centos-ss.yaml
[matrix]: ../../../../docs/network_storage.md#compatibility-matrix

## License

<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| auto\_delete\_disk | Whether or not the nfs disk should be auto-deleted | `bool` | `false` | no |
| deployment\_name | Name of the HPC deployment, used as name of the NFS instance if no name is specified. | `string` | n/a | yes |
| disk\_size | Storage size gb | `number` | `"100"` | no |
| image | DEPRECATED: The VM image used by the nfs server | `string` | `null` | no |
| instance\_image | The VM image used by the nfs server.<br><br>Expected Fields:<br>name: The name of the image. Mutually exclusive with family.<br>family: The image family to use. Mutually exclusive with name.<br>project: The project where the image is hosted. | `map(string)` | <pre>{<br>  "family": "hpc-rocky-linux-8",<br>  "project": "cloud-hpc-image-public"<br>}</pre> | no |
| labels | Labels to add to the NFS instance. Key-value pairs. | `map(string)` | n/a | yes |
| local\_mounts | Mountpoint for this NFS compute instance | `list(string)` | <pre>[<br>  "/data"<br>]</pre> | no |
| machine\_type | Type of the VM instance to use | `string` | `"n2d-standard-2"` | no |
| metadata | Metadata, provided as a map | `map(string)` | `{}` | no |
| name | The resource name of the instance. | `string` | `null` | no |
| network\_self\_link | The self link of the network to attach the nfs VM. | `string` | `"default"` | no |
| project\_id | Project in which the HPC deployment will be created | `string` | n/a | yes |
| scopes | Scopes to apply to the controller | `list(string)` | <pre>[<br>  "https://www.googleapis.com/auth/cloud-platform"<br>]</pre> | no |
| service\_account | Service Account for the NFS Server | `string` | `null` | no |
| subnetwork\_self\_link | The self link of the subnetwork to attach the nfs VM. | `string` | `null` | no |
| type | The service tier of the instance. | `string` | `"pd-ssd"` | no |
| zone | The zone name where the nfs instance located in. | `string` | n/a | yes |

## Outputs

| Name | Description |
|------|-------------|
| install\_nfs\_client | Script for installing NFS client |
| install\_nfs\_client\_runner | Runner to install NFS client using the startup-script module |
| mount\_runner | Runner to mount the file-system using an ansible playbook. The startup-script<br>module will automatically handle installation of ansible.<br>- id: example-startup-script<br>  source: modules/scripts/startup-script<br>  settings:<br>    runners:<br>    - $(your-fs-id.mount\_runner)<br>... |
| network\_storage | export of all desired folder directories |

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
