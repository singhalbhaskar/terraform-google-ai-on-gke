# terraform-google-ai-on-gke

## Description
### Tagline
This is an auto-generated module.

### Detailed
This module was generated from [terraform-google-module-template](https://github.com/terraform-google-modules/terraform-google-module-template/), which by default generates a module that simply creates a GCS bucket. As the module develops, this README should be updated.

The resources/services/activations/deletions that this module will create/trigger are:

- Create a GCS bucket with the provided name

### PreDeploy
To deploy this blueprint you must have an active billing account and billing permissions.

## Architecture
![alt text for diagram](https://www.link-to-architecture-diagram.com)
1. Architecture description step no. 1
2. Architecture description step no. 2
3. Architecture description step no. N

## Documentation
- [Hosting a Static Website](https://cloud.google.com/storage/docs/hosting-static-website)

## Deployment Duration
Configuration: X mins
Deployment: Y mins

## Cost
[Blueprint cost details](https://cloud.google.com/products/calculator?id=02fb0c45-cc29-4567-8cc6-f72ac9024add)

## Usage

Basic usage of this module is as follows:

```hcl
module "ai_on_gke" {
  source  = "terraform-google-modules/ai-on-gke/google"
  version = "~> 0.1"

  project_id  = "<PROJECT ID>"
  bucket_name = "gcs-test-bucket"
}
```

Functional examples are included in the
[examples](./examples/) directory.

<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| a3\_mega\_zone | Toolkit deployment variable: a3\_mega\_zone | `string` | n/a | yes |
| a3\_ultra\_zone | Toolkit deployment variable: a3\_ultra\_zone | `string` | n/a | yes |
| authorized\_cidr | Toolkit deployment variable: authorized\_cidr | `string` | `"0.0.0.0/0"` | no |
| goog\_cm\_deployment\_name | The name of the deployment and VM instance. | `string` | n/a | yes |
| gpu\_type | Toolkit deployment variable: gpu\_type | `string` | n/a | yes |
| labels | Toolkit deployment variable: labels | `any` | `{}` | no |
| node\_count\_gke | Toolkit deployment variable: node\_count\_gke | `number` | n/a | yes |
| node\_count\_gke\_nccl | Toolkit deployment variable: node\_count\_gke\_nccl | `number` | n/a | yes |
| node\_count\_llama\_3\_70b | Toolkit deployment variable: node\_count\_llama\_3\_70b | `number` | n/a | yes |
| node\_count\_llama\_3\_7b | Toolkit deployment variable: node\_count\_llama\_3\_7b | `number` | n/a | yes |
| placement\_policy\_name | Toolkit deployment variable: placement\_policy\_name | `string` | n/a | yes |
| project\_id | Toolkit deployment variable: project\_id | `string` | `""` | no |
| recipe | Toolkit deployment variable: recipe | `string` | n/a | yes |
| reservation | Toolkit deployment variable: reservation | `string` | n/a | yes |
| reservation\_block | Toolkit deployment variable: reservation\_block | `string` | n/a | yes |

## Outputs

| Name | Description |
|------|-------------|
| gke\_cluster | Link to GKE cluster |
| result\_bucket | Link to result GCS bucket |

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->

## Requirements

These sections describe requirements for using this module.

### Software

The following dependencies must be available:

- [Terraform][terraform] v0.13
- [Terraform Provider for GCP][terraform-provider-gcp] plugin v3.0

### Service Account

A service account with the following roles must be used to provision
the resources of this module:

- Storage Admin: `roles/storage.admin`

The [Project Factory module][project-factory-module] and the
[IAM module][iam-module] may be used in combination to provision a
service account with the necessary roles applied.

### APIs

A project with the following APIs enabled must be used to host the
resources of this module:

- Google Cloud Storage JSON API: `storage-api.googleapis.com`

The [Project Factory module][project-factory-module] can be used to
provision a project with the necessary APIs enabled.

## Contributing

Refer to the [contribution guidelines](./CONTRIBUTING.md) for
information on contributing to this module.

[iam-module]: https://registry.terraform.io/modules/terraform-google-modules/iam/google
[project-factory-module]: https://registry.terraform.io/modules/terraform-google-modules/project-factory/google
[terraform-provider-gcp]: https://www.terraform.io/docs/providers/google/index.html
[terraform]: https://www.terraform.io/downloads.html

## Security Disclosures

Please see our [security disclosure process](./SECURITY.md).
