## Description

Allows creation of service accounts for a Google Cloud Platform project.

### Example

```yaml
- id: service_acct
  source: community/modules/project/service-account
  settings:
    project_id: $(vars.project_id)
    name: instance_acct
    project_roles:
    - logging.logWriter
    - monitoring.metricWriter
    - storage.objectViewer
```

This creates a service account in GCP project "project_id" with the name
"instance_acct". It will have the 3 roles listed for all resources within the
project.

### Usage with startup-script module

When this module is used in conjunction with the [startup-script] module, the
service account must be granted (at least) read access to the bucket. This can
be achieved by granting project-wide access as shown above or by specifying the
service account as a bucket viewer in the startup-script module:

```yaml
- id: service_acct
  source: community/modules/project/service-account
  settings:
    project_id: $(vars.project_id)
    name: instance_acct
    project_roles:
    - logging.logWriter
    - monitoring.metricWriter
- id: script
  source: modules/scripts/startup-script
  settings:
    bucket_viewers:
    - $(service_acct.service_account_iam_email)
```

[startup-script]: ../../../../modules/scripts/startup-script/README.md

## License

<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| billing\_account\_id | If assigning billing role, specify a billing account (default is to assign at the organizational level). | `string` | `""` | no |
| deployment\_name | Name of the deployment (will be prepended to service account name) | `string` | n/a | yes |
| description | Description of the created service account. | `string` | `"Service Account"` | no |
| descriptions | Deprecated; create single service accounts using var.description. | `list(string)` | `null` | no |
| display\_name | Display name of the created service account. | `string` | `"Service Account"` | no |
| generate\_keys | Generate keys for service account. | `bool` | `false` | no |
| grant\_billing\_role | Grant billing user role. | `bool` | `false` | no |
| grant\_xpn\_roles | Grant roles for shared VPC management. | `bool` | `true` | no |
| name | Name of the service account to create. | `string` | n/a | yes |
| names | Deprecated; create single service accounts using var.name. | `list(string)` | `null` | no |
| org\_id | Id of the organization for org-level roles. | `string` | `""` | no |
| prefix | Deprecated; prefix now set using var.deployment\_name | `string` | `null` | no |
| project\_id | ID of the project | `string` | n/a | yes |
| project\_roles | List of roles to grant to service account (e.g. "storage.objectViewer" or "compute.instanceAdmin.v1" | `list(string)` | n/a | yes |

## Outputs

| Name | Description |
|------|-------------|
| key | Service account key (if creation was requested) |
| service\_account\_email | Service account e-mail address |
| service\_account\_iam\_email | Service account IAM binding format (serviceAccount:name@example.com) |

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
