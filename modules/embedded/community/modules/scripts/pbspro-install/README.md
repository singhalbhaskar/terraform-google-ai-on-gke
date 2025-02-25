## Description

This module creates Toolkit runners that download PBS Pro RPM packages and
installs them with configuration settings as documented in the PBS Pro ["Big
Book"][bigbook].

[bigbook]: https://help.altair.com/2022.1.0/PBS%20Professional/PBS2022.1.pdf

### Example

The following code snippet demonstrates use of this module with the
pbspro-preinstall module that uploads RPM packages to Cloud Storage.

```yaml
  - id: pbspro_setup
    source: community/modules/scripts/pbspro-preinstall
    settings:
      client_rpm: /path/to/pbspro-client.el7.x86_64.rpm
      execution_rpm: /path/to/pbspro-execution.el7.x86_64.rpm
      server_rpm: /path/to/pbspro-server.el7.x86_64.rpm

  - id: pbspro_install_server
    source: community/modules/scripts/pbspro-install
    use:
    - pbspro_setup
    settings:
      pbs_role: server
      rpm_url: $(pbspro_setup.pbs_server_rpm_url)
    outputs:
    - runner
```

## Support

PBS Professional is licensed and supported by [Altair][pbspro]. This module is
maintained and supported by the Cluster Toolkit team in collaboration with Altair.

[pbspro]: https://www.altair.com/pbs-professional

## License

<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| pbs\_data\_service\_user | PBS Data Service POSIX user | `string` | `"pbsdata"` | no |
| pbs\_exec | Root path in which to install PBS | `string` | `"/opt/pbs"` | no |
| pbs\_home | PBS working directory | `string` | `"/var/spool/pbs"` | no |
| pbs\_license\_server | IP address or DNS name of PBS license server (required only for PBS server hosts) | `string` | `"CHANGE_THIS_TO_PBS_PRO_LICENSE_SERVER_HOSTNAME"` | no |
| pbs\_license\_server\_port | Networking port of PBS license server | `number` | `6200` | no |
| pbs\_role | Type of PBS host to provision: server, client, execution | `string` | n/a | yes |
| pbs\_server | IP address or DNS name of PBS server host (required only for PBS client and execution hosts) | `string` | `"CHANGE_THIS_TO_PBS_PRO_SERVER_HOSTNAME"` | no |
| rpm\_url | Path to PBS Pro RPM file for select PBS host type (server, client, execution) | `string` | n/a | yes |

## Outputs

| Name | Description |
|------|-------------|
| runner | Toolkit runner to install the select PBS Pro host |

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
