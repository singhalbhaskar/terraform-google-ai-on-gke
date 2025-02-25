# Description

This module creates the Vertex AI Notebook, to be used in tutorials.

Primarily used for FSI - MonteCarlo Tutorial: **[fsi-montecarlo-on-batch-tutorial]**.

[fsi-montecarlo-on-batch-tutorial]: ../docs/tutorials/fsi-montecarlo-on-batch/README.md

## Usage

This is a simple usage:

```yaml
  - id: bucket
    source: community/modules/file-system/cloud-storage-bucket
    settings: 
      name_prefix: my-bucket
      local_mount: /home/jupyter/my-bucket

  - id: notebook
    source: community/modules/compute/notebook
    use: [bucket]
    settings:
      name_prefix: notebook
      machine_type: n1-standard-4

```

## License

<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| deployment\_name | Name of the HPC deployment; used as part of name of the notebook. | `string` | n/a | yes |
| gcs\_bucket\_path | Bucket name, can be provided from the google-cloud-storage module | `string` | `null` | no |
| instance\_image | Instance Image | `map(string)` | <pre>{<br>  "family": "tf-latest-cpu",<br>  "name": null,<br>  "project": "deeplearning-platform-release"<br>}</pre> | no |
| labels | Labels to add to the resource Key-value pairs. | `map(string)` | n/a | yes |
| machine\_type | The machine type to employ | `string` | n/a | yes |
| mount\_runner | mount content from the google-cloud-storage module | `map(string)` | n/a | yes |
| project\_id | ID of project in which the notebook will be created. | `string` | n/a | yes |
| zone | The zone to deploy to | `string` | n/a | yes |

## Outputs

No outputs.

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
