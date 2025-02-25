## Description

This module performs the following tasks:

- create an instance template from which execute points will be created
- create a managed instance group ([MIG][mig]) for execute points
- create a Toolkit runner to configure the autoscaler to scale the MIG

It is expected to be used with the [htcondor-install] and [htcondor-setup]
modules.

[htcondor-install]: ../../scripts/htcondor-install/README.md
[htcondor-setup]: ../../scheduler/htcondor-setup/README.md
[mig]: https://cloud.google.com/compute/docs/instance-groups/

### Known limitations

This module may be used multiple times in a blueprint to create sets of
execute points in an HTCondor pool. If used more than 1 time, the setting
[name_prefix](#input_name_prefix) must be set to a value that is unique across
all uses of the htcondor-execute-point module. If you do not follow this
constraint, you will likely receive an error while running `terraform apply`
similar to that shown below.

```text
Error: Invalid value for variable

  on modules/embedded/community/modules/scheduler/htcondor-access-point/main.tf line 136, in module "startup_script":
 136:   runners = local.all_runners
    ├────────────────
    │ var.runners is list of map of string with 5 elements

All startup-script runners must have a unique destination.
```

### How to configure jobs to select execute points

HTCondor access points provisioned by the Toolkit are specially configured to
honor an attribute named `RequireId` in each [Job ClassAd][jobad]. This value
must be set to the ID of a MIG created by an instance of this module. The
[htcondor-access-point] module includes a setting `var.default_mig_id` that will
set this value automatically to the MIG ID corresponding to the module's
execute points. If this setting is left unset each job must specify `+RequireId`
explicitly. In all cases, the default value can be overridden explicitly as shown
below:

```text
universe       = vanilla
executable     = /bin/echo
arguments      = "Hello, World!"
output         = out.$(ClusterId).$(ProcId)
error          = err.$(ClusterId).$(ProcId)
log            = log.$(ClusterId).$(ProcId)
request_cpus   = 1
request_memory = 100MB
+RequireId     = "htcondor-pool-ep-mig"
queue
```

[htcondor-access-point]: ../../scheduler/htcondor-access-point/README.md
[jobad]: https://htcondor.readthedocs.io/en/latest/users-manual/matchmaking-with-classads.html

### Example

A full example can be found in the [examples README][htc-example].

[htc-example]: ../../../../examples/README.md#htc-htcondoryaml--

The following code snippet creates a pool with 2 sets of HTCondor execute
points, one using On-demand pricing and the other using Spot pricing. They use
a startup script and network created in previous steps.

```yaml
- id: htcondor_execute_point
  source: community/modules/compute/htcondor-execute-point
  use:
  - network1
  - htcondor_secrets
  - htcondor_setup
  - htcondor_cm
  settings:
    instance_image:
      project: $(vars.project_id)
      family: $(vars.new_image_family)
    min_idle: 2

- id: htcondor_execute_point_spot
  source: community/modules/compute/htcondor-execute-point
  use:
  - network1
  - htcondor_secrets
  - htcondor_setup
  - htcondor_cm
  settings:
    instance_image:
      project: $(vars.project_id)
      family: $(vars.new_image_family)
    spot: true

- id: htcondor_access
  source: community/modules/scheduler/htcondor-access-point
  use:
  - network1
  - htcondor_secrets
  - htcondor_setup
  - htcondor_cm
  - htcondor_execute_point
  - htcondor_execute_point_spot
  settings:
    default_mig_id: $(htcondor_execute_point.mig_id)
    enable_public_ips: true
    instance_image:
      project: $(vars.project_id)
      family: $(vars.new_image_family)
  outputs:
  - access_point_ips
  - access_point_name
```

## Support

HTCondor is maintained by the [Center for High Throughput Computing][chtc] at
the University of Wisconsin-Madison. Support for HTCondor is available via:

- [Discussion lists](https://htcondor.org/mail-lists/)
- [HTCondor on GitHub](https://github.com/htcondor/htcondor/)
- [HTCondor manual](https://htcondor.readthedocs.io/en/latest/)

[chtc]: https://chtc.cs.wisc.edu/

## Behavior of Managed Instance Group (MIG)

Regional [MIGs][mig] are used to provision Execute Points. By default, VMs
will be provisioned in any of the zones available in that region, however, it
can be constrained to run in fewer zones (or a single zone) using
[var.zones](#input_zones).

When the configuration of an Execute Point is changed, the MIG can be configured
to [replace the VM][replacement] using a "proactive" or "opportunistic" policy.
By default, the policy is set to opportunistic. In practice, this means that
Execute Points will _NOT_ be automatically replaced by Terraform when changes to
the instance template / HTCondor configuration are made. We recommend leaving
this at the default value as it will allow the HTCondor autoscaler to replace
VMs when they become idle without disrupting running jobs.

However, if it is desired [var.update_policy](#input_update_policy) can be set
to "PROACTIVE" to enable automatic replacement. This will disrupt running jobs
and send them back to the queue. Alternatively, one can leave the setting at
the default value of "OPPORTUNISTIC" and update:

- intentionally by issuing an update via Cloud Console or using gcloud (below)
- VMs becomes unhealthy or are otherwise automatically replaced (e.g. regular
  Google Cloud maintenance)

For example, to manually update all instances in a MIG:

```text
gcloud compute instance-groups managed update-instances \
   <<NAME-OF-MIG>> --all-instances --region <<REGION>> \
   --project <<PROJECT_ID>> --minimal-action replace
```

[replacement]: https://cloud.google.com/compute/docs/instance-groups/rolling-out-updates-to-managed-instance-groups#type

## Known Issues

When using OS Login with "external users" (outside of the Google Cloud
organization), then Docker universe jobs will fail and cause the Docker daemon
to crash. This stems from the use of POSIX user ids (uid) outside the range
supported by Docker. Please consider disabling OS Login if this atypical
situation applies.

```yaml
vars:
  # add setting below to existing deployment variables
  enable_oslogin: DISABLE
```

## License

<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| allow\_automatic\_updates | If false, disables automatic system package updates on the created instances.  This feature is<br>only available on supported images (or images derived from them).  For more details, see<br>https://cloud.google.com/compute/docs/instances/create-hpc-vm#disable_automatic_updates | `bool` | `true` | no |
| central\_manager\_ips | List of IP addresses of HTCondor Central Managers | `list(string)` | n/a | yes |
| deployment\_name | Cluster Toolkit deployment name. HTCondor cloud resource names will include this value. | `string` | n/a | yes |
| disk\_size\_gb | Boot disk size in GB | `number` | `100` | no |
| disk\_type | Disk type for template | `string` | `"pd-balanced"` | no |
| distribution\_policy\_target\_shape | Target shape across zones for instance group managing execute points | `string` | `"ANY"` | no |
| enable\_oslogin | Enable or Disable OS Login with "ENABLE" or "DISABLE". Set to "INHERIT" to inherit project OS Login setting. | `string` | `"ENABLE"` | no |
| enable\_shielded\_vm | Enable the Shielded VM configuration (var.shielded\_instance\_config). | `bool` | `false` | no |
| execute\_point\_runner | A list of Toolkit runners for configuring an HTCondor execute point | `list(map(string))` | `[]` | no |
| execute\_point\_service\_account\_email | Service account for HTCondor execute point (e-mail format) | `string` | n/a | yes |
| guest\_accelerator | List of the type and count of accelerator cards attached to the instance. | <pre>list(object({<br>    type  = string,<br>    count = number<br>  }))</pre> | `[]` | no |
| htcondor\_bucket\_name | Name of HTCondor configuration bucket | `string` | n/a | yes |
| instance\_image | HTCondor execute point VM image<br><br>Expected Fields:<br>name: The name of the image. Mutually exclusive with family.<br>family: The image family to use. Mutually exclusive with name.<br>project: The project where the image is hosted. | `map(string)` | <pre>{<br>  "family": "hpc-rocky-linux-8",<br>  "project": "cloud-hpc-image-public"<br>}</pre> | no |
| labels | Labels to add to HTConodr execute points | `map(string)` | n/a | yes |
| machine\_type | Machine type to use for HTCondor execute points | `string` | `"n2-standard-4"` | no |
| max\_size | Maximum size of the HTCondor execute point pool. | `number` | `5` | no |
| metadata | Metadata to add to HTCondor execute points | `map(string)` | `{}` | no |
| min\_idle | Minimum number of idle VMs in the HTCondor pool (if pool reaches var.max\_size, this minimum is not guaranteed); set to ensure jobs beginning run more quickly. | `number` | `0` | no |
| name\_prefix | Name prefix given to hostnames in this group of execute points; must be unique across all instances of this module | `string` | n/a | yes |
| network\_self\_link | The self link of the network HTCondor execute points will join | `string` | `"default"` | no |
| network\_storage | An array of network attached storage mounts to be configured | <pre>list(object({<br>    server_ip             = string,<br>    remote_mount          = string,<br>    local_mount           = string,<br>    fs_type               = string,<br>    mount_options         = string,<br>    client_install_runner = map(string)<br>    mount_runner          = map(string)<br>  }))</pre> | `[]` | no |
| project\_id | Project in which the HTCondor execute points will be created | `string` | n/a | yes |
| region | The region in which HTCondor execute points will be created | `string` | n/a | yes |
| service\_account\_scopes | Scopes by which to limit service account attached to central manager. | `set(string)` | <pre>[<br>  "https://www.googleapis.com/auth/cloud-platform"<br>]</pre> | no |
| shielded\_instance\_config | Shielded VM configuration for the instance (must set var.enabled\_shielded\_vm) | <pre>object({<br>    enable_secure_boot          = bool<br>    enable_vtpm                 = bool<br>    enable_integrity_monitoring = bool<br>  })</pre> | <pre>{<br>  "enable_integrity_monitoring": true,<br>  "enable_secure_boot": true,<br>  "enable_vtpm": true<br>}</pre> | no |
| spot | Provision VMs using discounted Spot pricing, allowing for preemption | `bool` | `false` | no |
| subnetwork\_self\_link | The self link of the subnetwork HTCondor execute points will join | `string` | `null` | no |
| target\_size | Initial size of the HTCondor execute point pool; set to null (default) to avoid Terraform management of size. | `number` | `null` | no |
| update\_policy | Replacement policy for Access Point Managed Instance Group ("PROACTIVE" to replace immediately or "OPPORTUNISTIC" to replace upon instance power cycle) | `string` | `"OPPORTUNISTIC"` | no |
| windows\_startup\_ps1 | Startup script to run at boot-time for Windows-based HTCondor execute points | `list(string)` | `[]` | no |
| zones | Zone(s) in which execute points may be created. If not supplied, will default to all zones in var.region. | `list(string)` | `[]` | no |

## Outputs

| Name | Description |
|------|-------------|
| autoscaler\_runner | Toolkit runner to configure the HTCondor autoscaler |
| mig\_id | ID of the managed instance group containing the execute points |

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
