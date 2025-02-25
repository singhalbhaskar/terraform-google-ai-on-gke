# terraform-google-ai-on-gke

## Description

This module allows you to create opinionated Google Cloud Platform projects. It
creates projects and configures aspects like Shared VPC connectivity, IAM
access, Service Accounts, and API enablement to follow best practices.

This module is meant for use with Terraform 0.13.

**Note:** This module has been removed from the Cluster Toolkit. The upstream module (`terraform-google-project-factory`) is now the recommended way to create and manage GCP projects.

### Example

```yaml
- id: project
  source: github.com/terraform-google-modules/terraform-google-project-factory?rev=v17.0.0&depth=1
```

This creates a new project with pre-defined project ID, a designated folder and
organization and associated billing account which will be used to pay for
services consumed.

## License

<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| activate\_api\_identities | The list of service identities (Google Managed service account for the API) to force-create for the project (e.g. in order to grant additional roles).<br>    APIs in this list will automatically be appended to `activate_apis`.<br>    Not including the API in this list will follow the default behaviour for identity creation (which is usually when the first resource using the API is created).<br>    Any roles (e.g. service agent role) must be explicitly listed. See https://cloud.google.com/iam/docs/understanding-roles#service-agent-roles-roles for a list of related roles. | <pre>list(object({<br>    api   = string<br>    roles = list(string)<br>  }))</pre> | `[]` | no |
| activate\_apis | The list of apis to activate within the project | `list(string)` | <pre>[<br>  "compute.googleapis.com",<br>  "serviceusage.googleapis.com",<br>  "storage.googleapis.com"<br>]</pre> | no |
| auto\_create\_network | Create the default network | `bool` | `false` | no |
| billing\_account | The ID of the billing account to associate this project with | `string` | n/a | yes |
| bucket\_force\_destroy | Force the deletion of all objects within the GCS bucket when deleting the bucket (optional) | `bool` | `false` | no |
| bucket\_labels | A map of key/value label pairs to assign to the bucket (optional) | `map(string)` | `{}` | no |
| bucket\_location | The location for a GCS bucket to create (optional) | `string` | `"US"` | no |
| bucket\_name | A name for a GCS bucket to create (in the bucket\_project project), useful for Terraform state (optional) | `string` | `""` | no |
| bucket\_project | A project to create a GCS bucket (bucket\_name) in, useful for Terraform state (optional) | `string` | `""` | no |
| bucket\_ula | Enable Uniform Bucket Level Access | `bool` | `true` | no |
| bucket\_versioning | Enable versioning for a GCS bucket to create (optional) | `bool` | `false` | no |
| budget\_alert\_pubsub\_topic | The name of the Cloud Pub/Sub topic where budget related messages will be published, in the form of `projects/{project_id}/topics/{topic_id}` | `string` | `null` | no |
| budget\_alert\_spent\_percents | A list of percentages of the budget to alert on when threshold is exceeded | `list(number)` | <pre>[<br>  0.5,<br>  0.7,<br>  1<br>]</pre> | no |
| budget\_amount | The amount to use for a budget alert | `number` | `null` | no |
| budget\_display\_name | The display name of the budget. If not set defaults to `Budget For <projects[0]|All Projects>` | `string` | `null` | no |
| budget\_monitoring\_notification\_channels | A list of monitoring notification channels in the form `[projects/{project_id}/notificationChannels/{channel_id}]`. A maximum of 5 channels are allowed. | `list(string)` | `[]` | no |
| consumer\_quotas | The quotas configuration you want to override for the project. | <pre>list(object({<br>    service = string,<br>    metric  = string,<br>    limit   = string,<br>    value   = string,<br>  }))</pre> | `[]` | no |
| create\_project\_sa | Whether the default service account for the project shall be created | `bool` | `true` | no |
| default\_network\_tier | Default Network Service Tier for resources created in this project. If unset, the value will not be modified. See https://cloud.google.com/network-tiers/docs/using-network-service-tiers and https://cloud.google.com/network-tiers. | `string` | `""` | no |
| default\_service\_account | Project default service account setting: can be one of `delete`, `deprivilege`, `disable`, or `keep`. | `string` | `"keep"` | no |
| disable\_dependent\_services | Whether services that are enabled and which depend on this service should also be disabled when this service is destroyed. | `bool` | `true` | no |
| disable\_services\_on\_destroy | Whether project services will be disabled when the resources are destroyed | `bool` | `true` | no |
| domain | The domain name (optional). | `string` | `""` | no |
| enable\_shared\_vpc\_host\_project | If this project is a shared VPC host project. If true, you must *not* set svpc\_host\_project\_id variable. Default is false. | `bool` | `false` | no |
| folder\_id | The ID of a folder to host this project | `string` | `""` | no |
| grant\_services\_network\_role | Whether or not to grant service agents the network roles on the host project | `bool` | `true` | no |
| grant\_services\_security\_admin\_role | Whether or not to grant Kubernetes Engine Service Agent the Security Admin role on the host project so it can manage firewall rules | `bool` | `false` | no |
| group\_name | A group to control the project by being assigned group\_role (defaults to project editor) | `string` | `""` | no |
| group\_role | The role to give the controlling group (group\_name) over the project (defaults to project editor) | `string` | `"roles/editor"` | no |
| labels | Map of labels for project | `map(string)` | `{}` | no |
| lien | Add a lien on the project to prevent accidental deletion | `bool` | `false` | no |
| name | The name for the project | `string` | `null` | no |
| org\_id | The organization ID. | `string` | n/a | yes |
| project\_id | The ID to give the project. If not provided, the `name` will be used. | `string` | `""` | no |
| project\_sa\_name | Default service account name for the project. | `string` | `"project-service-account"` | no |
| random\_project\_id | Adds a suffix of 4 random characters to the `project_id` | `bool` | `false` | no |
| sa\_role | A role to give the default Service Account for the project (defaults to none) | `string` | `""` | no |
| shared\_vpc\_subnets | List of subnets fully qualified subnet IDs (ie. projects/$project\_id/regions/$region/subnetworks/$subnet\_id) | `list(string)` | `[]` | no |
| svpc\_host\_project\_id | The ID of the host project which hosts the shared VPC | `string` | `""` | no |
| usage\_bucket\_name | Name of a GCS bucket to store GCE usage reports in (optional) | `string` | `""` | no |
| usage\_bucket\_prefix | Prefix in the GCS bucket to store GCE usage reports in (optional) | `string` | `""` | no |
| vpc\_service\_control\_attach\_enabled | Whether the project will be attached to a VPC Service Control Perimeter | `bool` | `false` | no |
| vpc\_service\_control\_perimeter\_name | The name of a VPC Service Control Perimeter to add the created project to | `string` | `null` | no |

## Outputs

| Name | Description |
|------|-------------|
| api\_s\_account | API service account email |
| api\_s\_account\_fmt | API service account email formatted for terraform use |
| budget\_name | The name of the budget if created |
| domain | The organization's domain |
| enabled\_api\_identities | Enabled API identities in the project |
| enabled\_apis | Enabled APIs in the project |
| group\_email | The email of the G Suite group with group\_name |
| project\_bucket\_self\_link | Project's bucket selfLink |
| project\_bucket\_url | Project's bucket url |
| project\_id | ID of the project that was created |
| project\_name | Name of the project that was created |
| project\_number | Number of the project that was created |
| service\_account\_display\_name | The display name of the default service account |
| service\_account\_email | The email of the default service account |
| service\_account\_id | The id of the default service account |
| service\_account\_name | The fully-qualified name of the default service account |
| service\_account\_unique\_id | The unique id of the default service account |

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
