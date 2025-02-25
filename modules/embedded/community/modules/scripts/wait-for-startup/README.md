# terraform-google-ai-on-gke

## Description

This module will insert a dependency on the completion of the startup script
for one or more specified compute VMs and report back if it fails. This can be useful when running
post-boot installation scripts that require the startup script to finish setting up a node.

> **_WARNING:_**: this module is experimental and not fully supported.

### Additional Dependencies

* [**gcloud**](https://cloud.google.com/sdk/gcloud) must be present in the path
  of the machine where `terraform apply` is run.

### Example

```yaml
- id: workstation
  source: modules/compute/vm-instance
  use:
  - network1
  - my-startup-script
  settings:
    instance_count: 4

# Wait for all instances of the above VM to finish running startup scripts.
- id: wait
  source: community/modules/scripts/wait-for-startup
  settings:
    instance_names: $(workstation.name)
```

## License

<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| gcloud\_path\_override | Directory of the gcloud executable to be used during cleanup | `string` | `""` | no |
| instance\_name | Name of the instance we are waiting for (can be null if 'instance\_names' is not empty) | `string` | `null` | no |
| instance\_names | A list of instance names we are waiting for, in addition to the one mentioned in 'instance\_name' (if any) | `list(string)` | `[]` | no |
| project\_id | Project in which the HPC deployment will be created | `string` | n/a | yes |
| timeout | Timeout in seconds | `number` | `1200` | no |
| zone | The GCP zone where the instance is running | `string` | n/a | yes |

## Outputs

No outputs.

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
