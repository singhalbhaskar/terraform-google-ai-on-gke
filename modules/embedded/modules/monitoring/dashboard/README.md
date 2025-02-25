# terraform-google-ai-on-gke

## Description

Creates a [monitoring dashboard][gcp-dash] for the HPC cluster deployment. The
module includes a default HPC-focused dashboard with the ability to add custom
widgets as well as the option to add an empty dashboard and add widgets as
needed.

[gcp-dash]: https://cloud.google.com/monitoring/charts/predefined-dashboards

## Example

```yaml
- id: hpc_dash
  source: modules/monitoring/dashboard
  settings:
    widgets:
    - |
      {
        "text": {
          "content": "## Header",
          "format": "MARKDOWN"
        },
        "title": "Custom Text Block Widget"
      }
```

This module creates a dashboard based on the HPC dashboard (default) with an
extra text widget added as a multi-line string representing a JSON block.

## License

<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| base\_dashboard | Baseline dashboard template, select from HPC or Empty | `string` | `"HPC"` | no |
| deployment\_name | The name of the current deployment | `string` | n/a | yes |
| labels | Labels to add to the monitoring dashboard instance. Key-value pairs. | `map(string)` | n/a | yes |
| project\_id | Project in which the HPC deployment will be created | `string` | n/a | yes |
| title | Title of the created dashboard | `string` | `"Cluster Toolkit Dashboard"` | no |
| widgets | List of additional widgets to add to the base dashboard. | `list(string)` | `[]` | no |

## Outputs

| Name | Description |
|------|-------------|
| instructions | Instructions for accessing the monitoring dashboard |

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
