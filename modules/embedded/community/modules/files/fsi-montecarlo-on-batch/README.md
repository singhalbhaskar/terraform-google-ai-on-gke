## Description

Copy files to a target GCS bucket.

Primarily used for FSI - MonteCarlo Tutorial **[fsi-montecarlo-on-batch-tutorial]**.

[fsi-montecarlo-on-batch-tutorial]:
../docs/tutorials/fsi-montecarlo-on-batch/README.md

## Usage
This copies the module files to the specified GCS bucket. It is expected that
the bucket will be mounted on the target VM.

Some of the files are templates, and `main.tf` translates the files with the
passed variable values. This way the user does not have to change things like
pointing to the correct bigquery table or adding in the project_id.

```yaml
  - id: fsi_tutorial_files
    source: community/modules/files/fsi-montecarlo-on-batch
    use: [bq-dataset, bq-table, fsi_bucket, pubsub_topic]
```

## License

<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| dataset\_id | Bigquery dataset id | `string` | n/a | yes |
| gcs\_bucket\_path | Bucket name | `string` | `null` | no |
| project\_id | ID of project in which GCS bucket will be created. | `string` | n/a | yes |
| region | Region to run project | `string` | n/a | yes |
| table\_id | Bigquery table id | `string` | n/a | yes |
| topic\_id | Pubsub Topic Name | `string` | n/a | yes |
| topic\_schema | Pubsub Topic schema | `string` | n/a | yes |

## Outputs

No outputs.

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
