# terraform-google-ai-on-gke

<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| content | The YAML body to apply to gke cluster. | `string` | `null` | no |
| server\_side\_apply | Allow using kubectl server-side apply method. | `bool` | `false` | no |
| source\_path | The source for manifest(s) to apply to gke cluster. Acceptable sources are a local yaml or template (.tftpl) file path, a directory (ends with '/') containing yaml or template files, and a url for a yaml file. | `string` | `null` | no |
| template\_vars | The values to populate template file(s) with. | `map(any)` | `null` | no |
| wait\_for\_rollout | Wait or not for Deployments and APIService to complete rollout. See [kubectl wait](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_wait/) for more details. | `bool` | `true` | no |

## Outputs

No outputs.

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
