# terraform-google-ai-on-gke

## Description

This module discovers a VPC network that already exists in Google Cloud and
outputs network attributes that uniquely identify it for use by other modules.
The module outputs are aligned with the [vpc module][vpc] so that it can be used
as a drop-in substitute when a VPC already exists.

For example, the blueprint below discovers the "default" global network and the
"default" regional subnetwork in us-central1. With the `use` keyword, the
[vm-instance] module accepts the `network_self_link` and `subnetwork_self_link`
input variables that uniquely identify the network and subnetwork in which the
VM will be created.

[vpc]: ../vpc/README.md
[vm-instance]: ../../compute/vm-instance/README.md

### Example

```yaml
- id: network1
  source: modules/network/pre-existing-vpc
  settings:
    project_id: $(vars.project_id)
    region: us-central1

- id: example_vm
  source: modules/compute/vm-instance
  use:
  - network1
  settings:
    name_prefix: example
    machine_type: c2-standard-4
```

> **_NOTE:_** The `project_id` and `region` settings would be inferred from the
> deployment variables of the same name, but they are included here for clarity.

### Use shared-vpc

If a network is created in different project, this module can be used to
reference the network. To use a network from a different project first make sure
you have a [cloud nat][cloudnat] and [IAP][iap] forwarding. For more details,
refer [shared-vpc][shared-vpc-doc]

[cloudnat]: https://cloud.google.com/nat/docs/overview
[iap]: https://cloud.google.com/iap/docs/using-tcp-forwarding
[shared-vpc-doc]: ../../../examples/README.md#hpc-slurm-sharedvpcyaml-community-badge-experimental-badge

## License

<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| network\_name | Name of the existing VPC network | `string` | `"default"` | no |
| project\_id | Project in which the HPC deployment will be created | `string` | n/a | yes |
| region | Region in which to search for primary subnetwork | `string` | n/a | yes |
| subnetwork\_name | Name of the pre-existing VPC subnetwork; defaults to var.network\_name if set to null. | `string` | `null` | no |

## Outputs

| Name | Description |
|------|-------------|
| network\_id | ID of the existing VPC network |
| network\_name | Name of the existing VPC network |
| network\_self\_link | Self link of the existing VPC network |
| subnetwork | Full subnetwork object in the primary region |
| subnetwork\_address | Subnetwork IP range in the primary region |
| subnetwork\_name | Name of the subnetwork in the primary region |
| subnetwork\_self\_link | Subnetwork self-link in the primary region |

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
