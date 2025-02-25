## Description

Creates a BigQuery dataset.

Primarily used for FSI - MonteCarlo Tutorial: **[fsi-montecarlo-on-batch-tutorial]**.

[fsi-montecarlo-on-batch-tutorial]: ../docs/tutorials/fsi-montecarlo-on-batch/README.md

## Usage
This is a simple usage.

```yaml
  - id: bq-dataset
    source: community/modules/database/bigquery-dataset
    settings:
      dataset_id: my_dataset
```

## License

<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| dataset\_id | The name of the dataset to be created | `string` | `null` | no |
| deployment\_name | The name of the current deployment | `string` | n/a | yes |
| labels | Labels to add to the dataset. Key-value pairs. | `map(string)` | n/a | yes |
| project\_id | Project in which the HPC deployment will be created | `string` | n/a | yes |

## Outputs

| Name | Description |
|------|-------------|
| dataset\_id | Name of the dataset that was created. |

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
