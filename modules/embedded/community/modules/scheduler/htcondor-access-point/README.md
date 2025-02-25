# terraform-google-ai-on-gke

## Description

This module provisions a highly available HTCondor access point using a [Managed
Instance Group (MIG)][mig] with auto-healing.

[mig]: https://cloud.google.com/compute/docs/instance-groups

## Usage

Although this provisions an HTCondor access point with standard configuration,
for a functioning node, you must supply Toolkit runners as described below:

- [var.access_point_runner](#input_access_point_runner)
  - Runner must download or otherwise create an [IDTOKEN] with ADVERTISE_MASTER,
    ADVERTISE_SCHEDD, and DAEMON scopes
- [var.autoscaler_runner](#input_autoscaler_runner)
  - 1 runner for each set of execute points to add to the pool

Reference implementations for each are included in the Toolkit modules
[htcondor-pool-secrets] and [htcondor-execute-point]. You may substitute
implementations (e.g. alternative secret management) so long as they duplicate
the functionality in these references. Their usage is demonstrated in the
[HTCondor example][htc-example].

[htc-example]: ../../../../examples/README.md#htc-htcondoryaml--
[htcondor-execute-point]: ../../compute/htcondor-execute-point/README.md
[htcondor-pool-secrets]: ../htcondor-pool-secrets/README.md
[IDTOKEN]: https://htcondor.readthedocs.io/en/latest/admin-manual/security.html#introducing-idtokens

## Behavior of Managed Instance Group (MIG)

A regional [MIG][mig] is used to provision the Access Point, although only
1 node will ever be active at a time. By default, the node will be provisioned
in any of the zones available in that region, however, it can be constrained to
run in fewer zones (or a single zone) using [var.zones](#input_zones).

When the configuration of the Central Manager is changed, the MIG can be
configured to [replace the VM][replacement] using a "proactive" or
"opportunistic" policy. By default, the Access Point replacement policy is
opportunistic. In practice, this means that the Access Point will _NOT_ be
automatically replaced by Terraform when changes to the instance template /
HTCondor configuration are made. The Access Point is _NOT_ safe to replace
automatically as its local storage contains the state of the job queue. By
default, the Access Point will be replaced only when:

- intentionally by issuing an update via Cloud Console or using gcloud (below)
- the VM becomes unhealthy or is otherwise automatically replaced (e.g. regular
  Google Cloud maintenance)

For example, to manually update all instances in a MIG:

```text
gcloud compute instance-groups managed update-instances \
   <<NAME-OF-MIG>> --all-instances --region <<REGION>> \
   --project <<PROJECT_ID>> --minimal-action replace
```

This mode can be switched to proactive (automatic) replacement by setting
[var.update_policy](#input_update_policy) to "PROACTIVE". In this case we
recommend the use of Filestore to store the job queue state ("spool") and
setting [var.spool_parent_dir][#input_spool_parent_dir] to its mount point:

```yaml
  - id: spoolfs
    source: modules/file-system/filestore
    use:
    - network1
    settings:
      filestore_tier: ENTERPRISE
      local_mount: /shared

...

  - id: htcondor_access
    source: community/modules/scheduler/htcondor-access-point
    use:
    - network1
    - spoolfs
    - htcondor_secrets
    - htcondor_setup
    - htcondor_cm
    - htcondor_execute_point_group
    settings:
      spool_parent_dir: /shared
```

[replacement]: https://cloud.google.com/compute/docs/instance-groups/rolling-out-updates-to-managed-instance-groups#type

<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| access\_point\_runner | A list of Toolkit runners for configuring an HTCondor access point | `list(map(string))` | `[]` | no |
| access\_point\_service\_account\_email | Service account for access point (e-mail format) | `string` | n/a | yes |
| allow\_automatic\_updates | If false, disables automatic system package updates on the created instances.  This feature is<br>only available on supported images (or images derived from them).  For more details, see<br>https://cloud.google.com/compute/docs/instances/create-hpc-vm#disable_automatic_updates | `bool` | `true` | no |
| autoscaler\_runner | A list of Toolkit runners for configuring autoscaling daemons | `list(map(string))` | `[]` | no |
| central\_manager\_ips | List of IP addresses of HTCondor Central Managers | `list(string)` | n/a | yes |
| default\_mig\_id | Default MIG ID for HTCondor jobs; if unset, jobs must specify MIG id | `string` | `""` | no |
| deployment\_name | Cluster Toolkit deployment name. HTCondor cloud resource names will include this value. | `string` | n/a | yes |
| disk\_size\_gb | Boot disk size in GB | `number` | `32` | no |
| disk\_type | Boot disk size in GB | `string` | `"pd-balanced"` | no |
| distribution\_policy\_target\_shape | Target shape acoss zones for instance group managing high availability of access point | `string` | `"ANY_SINGLE_ZONE"` | no |
| enable\_high\_availability | Provision HTCondor access point in high availability mode | `bool` | `false` | no |
| enable\_oslogin | Enable or Disable OS Login with "ENABLE" or "DISABLE". Set to "INHERIT" to inherit project OS Login setting. | `string` | `"ENABLE"` | no |
| enable\_public\_ips | Enable Public IPs on the access points | `bool` | `false` | no |
| enable\_shielded\_vm | Enable the Shielded VM configuration (var.shielded\_instance\_config). | `bool` | `false` | no |
| htcondor\_bucket\_name | Name of HTCondor configuration bucket | `string` | n/a | yes |
| instance\_image | Custom VM image with HTCondor and Toolkit support installed."<br><br>Expected Fields:<br>name: The name of the image. Mutually exclusive with family.<br>family: The image family to use. Mutually exclusive with name.<br>project: The project where the image is hosted. | `map(string)` | n/a | yes |
| labels | Labels to add to resources. List key, value pairs. | `map(string)` | n/a | yes |
| machine\_type | Machine type to use for HTCondor central managers | `string` | `"n2-standard-4"` | no |
| metadata | Metadata to add to HTCondor central managers | `map(string)` | `{}` | no |
| mig\_id | List of Managed Instance Group IDs containing execute points in this pool (supplied by htcondor-execute-point module) | `list(string)` | `[]` | no |
| network\_self\_link | The self link of the network in which the HTCondor central manager will be created. | `string` | `null` | no |
| network\_storage | An array of network attached storage mounts to be configured | <pre>list(object({<br>    server_ip             = string,<br>    remote_mount          = string,<br>    local_mount           = string,<br>    fs_type               = string,<br>    mount_options         = string,<br>    client_install_runner = map(string)<br>    mount_runner          = map(string)<br>  }))</pre> | `[]` | no |
| project\_id | Project in which HTCondor pool will be created | `string` | n/a | yes |
| region | Default region for creating resources | `string` | n/a | yes |
| service\_account\_scopes | Scopes by which to limit service account attached to central manager. | `set(string)` | <pre>[<br>  "https://www.googleapis.com/auth/cloud-platform"<br>]</pre> | no |
| shielded\_instance\_config | Shielded VM configuration for the instance (must set var.enabled\_shielded\_vm) | <pre>object({<br>    enable_secure_boot          = bool<br>    enable_vtpm                 = bool<br>    enable_integrity_monitoring = bool<br>  })</pre> | <pre>{<br>  "enable_integrity_monitoring": true,<br>  "enable_secure_boot": true,<br>  "enable_vtpm": true<br>}</pre> | no |
| spool\_disk\_size\_gb | Boot disk size in GB | `number` | `32` | no |
| spool\_disk\_type | Boot disk size in GB | `string` | `"pd-ssd"` | no |
| spool\_parent\_dir | HTCondor access point configuration SPOOL will be set to subdirectory named "spool" | `string` | `"/var/lib/condor"` | no |
| subnetwork\_self\_link | The self link of the subnetwork in which the HTCondor central manager will be created. | `string` | `null` | no |
| update\_policy | Replacement policy for Access Point Managed Instance Group ("PROACTIVE" to replace immediately or "OPPORTUNISTIC" to replace upon instance power cycle) | `string` | `"OPPORTUNISTIC"` | no |
| zones | Zone(s) in which access point may be created. If not supplied, defaults to 2 randomly-selected zones in var.region. | `list(string)` | `[]` | no |

## Outputs

| Name | Description |
|------|-------------|
| access\_point\_ips | IP addresses of the access points provisioned by this module |
| access\_point\_name | Name of the access point provisioned by this module |

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
