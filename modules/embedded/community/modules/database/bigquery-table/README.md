## Description

Creates a BigQuery table with a specified schema.

Primarily used for FSI - MonteCarlo Tutorial: **[fsi-montecarlo-on-batch-tutorial]**.

[fsi-montecarlo-on-batch-tutorial]: ../docs/tutorials/fsi-montecarlo-on-batch/README.md

## Usage

```yaml
id: bq-table
    source: community/modules/database/bigquery-table
    use: [bq-dataset]
    settings:
      table_schema:
        '
        [
          {
            "name": "id", "type": "STRING"
          }
        ]
        '
```

## License

<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| dataset\_id | Dataset name to be used to create the new BQ Table | `string` | n/a | yes |
| deployment\_name | The name of the current deployment | `string` | n/a | yes |
| labels | Labels to add to the tables. Key-value pairs. | `map(string)` | n/a | yes |
| project\_id | Project in which the HPC deployment will be created | `string` | n/a | yes |
| table\_id | Table name to be used to create the new BQ Table | `string` | `null` | no |
| table\_schema | Schema used to create the new BQ Table | `string` | n/a | yes |

## Outputs

| Name | Description |
|------|-------------|
| dataset\_id | ID of BQ dataset |
| table\_id | ID of created BQ table |
| table\_name | Name of created BQ table |

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
