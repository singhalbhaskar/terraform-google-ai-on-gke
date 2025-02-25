<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| guest\_accelerator | List of the type and count of accelerator cards attached to the instance. | <pre>list(object({<br>    type  = string,<br>    count = number<br>  }))</pre> | `[]` | no |
| machine\_type | Machine type to use for the instance creation | `string` | n/a | yes |

## Outputs

| Name | Description |
|------|-------------|
| guest\_accelerator | Sanitized list of the type and count of accelerator cards attached to the instance. |
| machine\_type\_guest\_accelerator | List of the type and count of accelerator cards attached to the specified machine type. |

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
