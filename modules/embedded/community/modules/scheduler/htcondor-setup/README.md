# terraform-google-ai-on-gke

## Description

This module creates a bucket in which to store HTCondor configurations and
a firewall rule that allows Managed Instance Group health checks to probe the
health of HTCondor VMs.

### Example

The following code snippet uses this module to create a startup script that
installs HTCondor software and configures an HTCondor Central Manager. A full
example can be found in the [examples README][htc-example].

[htc-example]: ../../../../examples/README.md#htc-htcondoryaml--

```yaml
- id: network1
  source: modules/network/pre-existing-vpc

- id: htcondor_install
  source: community/modules/scripts/htcondor-install

- id: htcondor_service_accounts
  source: community/modules/scheduler/htcondor-service-accounts

- id: htcondor_setup
  source: community/modules/scheduler/htcondor-setup
  use:
  - network1
  - htcondor_service_accounts

- id: htcondor_secrets
  source: community/modules/scheduler/htcondor-pool-secrets
  use:
  - htcondor_service_accounts

- id: htcondor_cm
  source: community/modules/scheduler/htcondor-central-manager
  use:
  - network1
  - htcondor_secrets
  - htcondor_service_accounts
  - htcondor_setup
  settings:
    instance_image:
      project: $(vars.project_id)
      family: $(vars.new_image_family)
  outputs:
  - central_manager_name
```

## Support

HTCondor is maintained by the [Center for High Throughput Computing][chtc] at
the University of Wisconsin-Madison. Support for HTCondor is available via:

- [Discussion lists](https://htcondor.org/mail-lists/)
- [HTCondor on GitHub](https://github.com/htcondor/htcondor/)
- [HTCondor manual](https://htcondor.readthedocs.io/en/latest/)

[chtc]: https://chtc.cs.wisc.edu/

## License

<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| access\_point\_service\_account\_email | Service account e-mail for HTCondor Access Point | `string` | n/a | yes |
| central\_manager\_service\_account\_email | Service account e-mail for HTCondor Central Manager | `string` | n/a | yes |
| deployment\_name | Cluster Toolkit deployment name. HTCondor cloud resource names will include this value. | `string` | n/a | yes |
| execute\_point\_service\_account\_email | Service account e-mail for HTCondor Execute Points | `string` | n/a | yes |
| labels | Labels to add to resources. List key, value pairs. | `map(string)` | n/a | yes |
| project\_id | Project in which HTCondor pool will be created | `string` | n/a | yes |
| region | Default region for creating resources | `string` | n/a | yes |
| subnetwork\_self\_link | The self link of the subnetwork in which Central Managers will be placed. | `string` | n/a | yes |

## Outputs

| Name | Description |
|------|-------------|
| htcondor\_bucket\_name | Name of the HTCondor configuration bucket |

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
