# terraform-google-ai-on-gke

# GCS bucket used in the RAG on GKE demo

This repository contains a Terraform template for creating the GCS bucket used
in the RAG on GKE demo.

<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| bucket\_name | GCS bucket name | `string` | n/a | yes |
| project\_id | GCP project id | `string` | n/a | yes |
| region | GCS bucket region | `string` | `"us-central1"` | no |

## Outputs

No outputs.

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->