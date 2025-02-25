# terraform-google-ai-on-gke

<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| base\_instance\_name | Base name for the instances in the MIG | `string` | `null` | no |
| deployment\_name | Name of the deployment, will be used to name MIG if `var.name` is not provided | `string` | n/a | yes |
| ghpc\_module\_id | Internal GHPC field, do not set this value | `string` | `null` | no |
| labels | Labels to add to the MIG | `map(string)` | n/a | yes |
| name | Name of the MIG. If not provided, will be generated from `var.deployment_name` | `string` | `null` | no |
| project\_id | Project in which the MIG will be created | `string` | n/a | yes |
| target\_size | Target number of instances in the MIG | `number` | `0` | no |
| versions | Application versions managed by this instance group. Each version deals with a specific instance template | <pre>list(object({<br>    name              = string<br>    instance_template = string<br>    target_size = optional(object({<br>      fixed   = optional(number)<br>      percent = optional(number)<br>    }))<br>  }))</pre> | n/a | yes |
| wait\_for\_instances | Whether to wait for all instances to be created/updated before returning | `bool` | `false` | no |
| zone | Compute Platform zone. Required, currently only zonal MIGs are supported | `string` | n/a | yes |

## Outputs

| Name | Description |
|------|-------------|
| self\_link | The URL of the created MIG |

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
