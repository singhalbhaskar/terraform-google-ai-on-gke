# terraform-google-ai-on-gke

## Description

This module discovers a subnetwork that already exists in Google Cloud and
outputs subnetwork attributes that uniquely identify it for use by other modules.

For example, the blueprint below discovers the referred to subnetwork.
With the `use` keyword, the [vm-instance] module accepts the `subnetwork_self_link`
input variables that uniquely identify the subnetwork in which the VM will be created.

[vpc]: ../vpc/README.md
[vm-instance]: ../../compute/vm-instance/README.md

> **_NOTE:_** Additional IAM work is needed for this to work correctly.

### Example

```yaml
- id: network
  source: modules/network/pre-existing-subnetwork
  settings:
    subnetwork_self_link: https://www.googleapis.com/compute/v1/projects/name-of-host-project/regions/REGION/subnetworks/SUBNETNAME

- id: example_vm
  source: modules/compute/vm-instance
  use:
  - network
  settings:
    name_prefix: example
    machine_type: c2-standard-4
```

As described in documentation:
[https://registry.terraform.io/providers/hashicorp/google/latest/docs/data-sources/compute_subnetwork]

If subnetwork_self_link is provided then name,region,project is ignored.

## License

<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| project | Name of the project that owns the subnetwork | `string` | `null` | no |
| region | Region in which to search for primary subnetwork | `string` | `null` | no |
| subnetwork\_name | Name of the pre-existing VPC subnetwork | `string` | `null` | no |
| subnetwork\_self\_link | Self-link of the subnet in the VPC | `string` | `null` | no |

## Outputs

| Name | Description |
|------|-------------|
| subnetwork | Full subnetwork object in the primary region |
| subnetwork\_address | Subnetwork IP range in the primary region |
| subnetwork\_name | Name of the subnetwork in the primary region |
| subnetwork\_self\_link | Subnetwork self-link in the primary region |

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
