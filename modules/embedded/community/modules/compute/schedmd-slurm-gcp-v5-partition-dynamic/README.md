# terraform-google-ai-on-gke

## Description

> [!NOTE]
> Slurm-gcp-v5-partition-dynamic module is deprecated. See
> [this update](../../../../examples/README.md#completed-migration-to-slurm-gcp-v6)
> for specific recommendations and timelines.

This module creates a dynamic compute partition that can be used as input to the
[schedmd-slurm-gcp-v5-controller](../../scheduler/schedmd-slurm-gcp-v5-controller/README.md).
This will configure the slurm partition to contain nodes with the corresponding feature.
This supports externally created nodes that register as a dynamic node to also be placed
into their corresponding partition based on node feature.

> **Warning**: updating a partition and running `terraform apply` will not cause
> the slurm controller to update its own configurations (`slurm.conf`) unless
> `enable_reconfigure` is set to true in the partition and controller modules.

## Example

The following example creates a dynamic partition, which is then used by a slurm
controller. This partition will register nodes that have the partition feature
of "dyn".

```yaml
  - id: dynamic_partition
    source: community/modules/compute/schedmd-slurm-gcp-v5-partition-dynamic
    use: [network1]
    settings:
      partition_name: dynamic
      partition_feature: dyn

  - id: slurm_controller
    source: community/modules/scheduler/schedmd-slurm-gcp-v5-controller
    use: [network1, dynamic_partition]
```

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
| deployment\_name | Name of the deployment. | `string` | n/a | yes |
| exclusive | Exclusive job access to nodes. | `bool` | `true` | no |
| is\_default | Sets this partition as the default partition by updating the partition\_conf.<br>If "Default" is already set in partition\_conf, this variable will have no effect. | `bool` | `false` | no |
| partition\_conf | Slurm partition configuration as a map.<br>See https://slurm.schedmd.com/slurm.conf.html#SECTION_PARTITION-CONFIGURATION | `map(string)` | `{}` | no |
| partition\_feature | Any nodes with this feature will automatically be put into this partition.<br><br>NOTE: meant to be used for external dynamic nodes that register. | `string` | n/a | yes |
| partition\_name | The name of the slurm partition. | `string` | n/a | yes |
| project\_id | Project in which the HPC deployment will be created. | `string` | n/a | yes |
| region | The default region for Cloud resources. | `string` | n/a | yes |
| slurm\_cluster\_name | Cluster name, used for resource naming and slurm accounting. If not provided it will default to the first 8 characters of the deployment name (removing any invalid characters). | `string` | `null` | no |
| subnetwork\_project | The project the subnetwork belongs to. | `string` | `""` | no |
| subnetwork\_self\_link | Subnet to deploy to. | `string` | `null` | no |

## Outputs

| Name | Description |
|------|-------------|
| partition | Details of a slurm partition |

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
