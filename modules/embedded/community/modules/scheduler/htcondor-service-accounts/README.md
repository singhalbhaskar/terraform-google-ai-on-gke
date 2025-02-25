## Description

This module creates the service accounts for use by the primary elements of an
[HTCondor pool][pool]:

- Central Managers
- Access Points
- Execute Points

Each service account is assigned common roles necessary for the VM to function
properly. In particular, nearly every VM requires the ability to read from Cloud
Storage buckets and write Cloud Logging entries. These roles are configurable
as described below.

[pool]: https://htcondor.readthedocs.io/en/latest/admin-manual/introduction-admin-manual.html#the-different-roles-a-machine-can-play

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
| access\_point\_roles | Project-wide roles for HTCondor Access Point service account | `list(string)` | <pre>[<br>  "compute.instanceAdmin.v1",<br>  "monitoring.metricWriter",<br>  "logging.logWriter",<br>  "storage.objectViewer"<br>]</pre> | no |
| central\_manager\_roles | Project-wide roles for HTCondor Central Manager service account | `list(string)` | <pre>[<br>  "monitoring.metricWriter",<br>  "logging.logWriter",<br>  "storage.objectViewer"<br>]</pre> | no |
| deployment\_name | Cluster Toolkit deployment name. HTCondor cloud resource names will include this value. | `string` | n/a | yes |
| execute\_point\_roles | Project-wide roles for HTCondor Execute Point service account | `list(string)` | <pre>[<br>  "monitoring.metricWriter",<br>  "logging.logWriter",<br>  "storage.objectViewer"<br>]</pre> | no |
| project\_id | Project in which HTCondor pool will be created | `string` | n/a | yes |

## Outputs

| Name | Description |
|------|-------------|
| access\_point\_service\_account\_email | HTCondor Access Point Service Account (e-mail format) |
| central\_manager\_service\_account\_email | HTCondor Central Manager Service Account (e-mail format) |
| execute\_point\_service\_account\_email | HTCondor Execute Point Service Account (e-mail format) |

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
