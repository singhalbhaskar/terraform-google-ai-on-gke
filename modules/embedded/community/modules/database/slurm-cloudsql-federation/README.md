## Description

terraform-google-sql makes it easy to create a Google CloudSQL instance and
implement high availability settings. This module is meant for use with
Terraform 0.13+ and tested using Terraform 1.0+.

The cloudsql created here is used to integrate with the slurm cluster to enable
accounting data storage.

### Example

```yaml
- id: project
  source: community/modules/database/cloudsql-federation
  use: [network1]
  settings:
    sql_instance_name: slurm-sql6-demo
    tier: "db-f1-micro"
```

This creates a cloud sql instance, including a database, user that would allow
the slurm cluster to use as an external DB. In addition, it will allow BigQuery
to run federated query through it.

## License

<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| authorized\_networks | IP address ranges as authorized networks of the Cloud SQL for MySQL instances | `list(string)` | `[]` | no |
| data\_cache\_enabled | Whether data cache is enabled for the instance. Can be used with ENTERPRISE\_PLUS edition. | `bool` | `false` | no |
| database\_version | The version of the database to be created. | `string` | `"MYSQL_5_7"` | no |
| deletion\_protection | Whether or not to allow Terraform to destroy the instance. | `string` | `false` | no |
| deployment\_name | The name of the current deployment | `string` | n/a | yes |
| disk\_autoresize | Set to false to disable automatic disk grow. | `bool` | `true` | no |
| disk\_size\_gb | Size of the database disk in GiB. | `number` | `null` | no |
| edition | value | `string` | `"ENTERPRISE"` | no |
| enable\_backups | Set true to enable backups | `bool` | `false` | no |
| labels | Labels to add to the instances. Key-value pairs. | `map(string)` | n/a | yes |
| network\_id | The ID of the GCE VPC network to which the instance is going to be created in.:<br>`projects/<project_id>/global/networks/<network_name>`" | `string` | n/a | yes |
| private\_vpc\_connection\_peering | The name of the VPC Network peering connection, used only as dependency for Cloud SQL creation. | `string` | `null` | no |
| project\_id | Project in which the HPC deployment will be created | `string` | n/a | yes |
| region | The region where SQL instance will be configured | `string` | n/a | yes |
| sql\_instance\_name | name given to the sql instance for ease of identificaion | `string` | n/a | yes |
| sql\_password | Password for the SQL database. | `any` | `null` | no |
| sql\_username | Username for the SQL database | `string` | `"slurm"` | no |
| subnetwork\_self\_link | Self link of the network where Cloud SQL instance PSC endpoint will be created | `string` | `null` | no |
| tier | The machine type to use for the SQL instance | `string` | n/a | yes |
| use\_psc\_connection | Create Private Service Connection instead of using Private Service Access peering | `bool` | `false` | no |
| user\_managed\_replication | Replication parameters that will be used for defined secrets | <pre>list(object({<br>    location     = string<br>    kms_key_name = optional(string)<br>  }))</pre> | `[]` | no |

## Outputs

| Name | Description |
|------|-------------|
| cloudsql | Describes the cloudsql instance. |

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
