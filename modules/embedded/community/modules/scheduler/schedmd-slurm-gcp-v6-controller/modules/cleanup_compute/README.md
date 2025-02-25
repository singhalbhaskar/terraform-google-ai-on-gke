<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| enable\_cleanup\_compute | Enables automatic cleanup of compute nodes and resource policies (e.g.<br>placement groups) managed by this module, when cluster is destroyed.<br><br>*WARNING*: Toggling this off will impact the running workload.<br>Deployed compute nodes will be destroyed. | `bool` | n/a | yes |
| endpoint\_versions | Version of the API to use (The compute service is the only API currently supported) | <pre>object({<br>    compute = string<br>  })</pre> | n/a | yes |
| gcloud\_path\_override | Directory of the gcloud executable to be used during cleanup | `string` | n/a | yes |
| nodeset | Nodeset to cleanup | <pre>object({<br>    nodeset_name         = string<br>    subnetwork_self_link = string<br>    additional_networks = list(object({<br>      subnetwork = string<br>    }))<br>  })</pre> | n/a | yes |
| project\_id | Project ID | `string` | n/a | yes |
| slurm\_cluster\_name | Name of the Slurm cluster | `string` | n/a | yes |
| universe\_domain | Domain address for alternate API universe | `string` | n/a | yes |

## Outputs

No outputs.

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
