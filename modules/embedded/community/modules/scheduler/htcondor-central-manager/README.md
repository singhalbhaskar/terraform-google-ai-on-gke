# terraform-google-ai-on-gke

## Description

This module provisions a highly available HTCondor central manager using a [Managed
Instance Group (MIG)][mig] with auto-healing.

[mig]: https://cloud.google.com/compute/docs/instance-groups

## Usage

This module provisions an HTCondor central manager with a standard
configuration. For the node to function correctly, you must supply the input
variable described below:

- [var.central_manager_runner](#input_central_manager_runner)
  - Runner must download a POOL password / signing key and create an [IDTOKEN]
  with no scopes (full authorization).

A reference implementation is included in the Toolkit module
[htcondor-pool-secrets]. You may substitute implementations so long as they
duplicate the functionality in the references. Usage is demonstrated in the
[HTCondor example][htc-example].

[htc-example]: ../../../../examples/README.md#htc-htcondoryaml--
[htcondor-pool-secrets]: ../htcondor-pool-secrets/README.md
[IDTOKEN]: https://htcondor.readthedocs.io/en/latest/admin-manual/security.html#introducing-idtokens

## Behavior of Managed Instance Group (MIG)

A regional [MIG][mig] is used to provision the central manager, although only
1 node will ever be active at a time. By default, the node will be provisioned
in any of the zones available in that region, however, it can be constrained to
run in fewer zones (or a single zone) using [var.zones](#input_zones).

When the configuration of the Central Manager is changed, the MIG can be
configured to [replace the VM][replacement] using a "proactive" or
"opportunistic" policy. By default, the Central Manager replacement policy is
set to proactive. In practice, this means that the Central Manager will be
replaced by Terraform when changes to the instance template / HTCondor
configuration are made. The Central Manager is safe to replace automatically as
it gathers its state information from periodic messages exchanged with the rest
of the HTCondor pool.

This mode can be configured by setting [var.update_policy](#input_update_policy)
to either "PROACTIVE" (default) or "OPPORTUNISTIC". If set to opportunistic
replacement, the Central Manager will be replaced only when:

- intentionally by issuing an update via Cloud Console or using gcloud (below)
- the VM becomes unhealthy or is otherwise automatically replaced (e.g. regular
  Google Cloud maintenance)

For example, to manually update all instances in a MIG:

```text
gcloud compute instance-groups managed update-instances \
   <<NAME-OF-MIG>> --all-instances --region <<REGION>> \
   --project <<PROJECT_ID>> --minimal-action replace
```

[replacement]: https://cloud.google.com/compute/docs/instance-groups/rolling-out-updates-to-managed-instance-groups#type

## Limiting inter-zone egress

Because all the elements of the HTCondor pool use regional MIGs, they may be
subject to [interzone egress fees][network-pricing]. The primary traffic between
nodes of an HTCondor pool running embarrassingly parallel jobs is expected to
be limited to API traffic for job scheduling and monitoring. Please review the
[network pricing][network-pricing] documentation and determine if this cost is
a concern. If it is, use [var.zones](#input_zones) to constrain each node within
your HTCondor pool to operate within a single zone.

[network-pricing]: https://cloud.google.com/vpc/network-pricing

## License

<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| allow\_automatic\_updates | If false, disables automatic system package updates on the created instances.  This feature is<br>only available on supported images (or images derived from them).  For more details, see<br>https://cloud.google.com/compute/docs/instances/create-hpc-vm#disable_automatic_updates | `bool` | `true` | no |
| central\_manager\_runner | A list of Toolkit runners for configuring an HTCondor central manager | `list(map(string))` | `[]` | no |
| central\_manager\_service\_account\_email | Service account e-mail for central manager (can be supplied by htcondor-setup module) | `string` | n/a | yes |
| deployment\_name | Cluster Toolkit deployment name. HTCondor cloud resource names will include this value. | `string` | n/a | yes |
| disk\_size\_gb | Boot disk size in GB | `number` | `20` | no |
| distribution\_policy\_target\_shape | Target shape for instance group managing high availability of central manager | `string` | `"ANY_SINGLE_ZONE"` | no |
| enable\_oslogin | Enable or Disable OS Login with "ENABLE" or "DISABLE". Set to "INHERIT" to inherit project OS Login setting. | `string` | `"ENABLE"` | no |
| enable\_shielded\_vm | Enable the Shielded VM configuration (var.shielded\_instance\_config). | `bool` | `false` | no |
| htcondor\_bucket\_name | Name of HTCondor configuration bucket | `string` | n/a | yes |
| instance\_image | Custom VM image with HTCondor installed using the htcondor-install module."<br><br>Expected Fields:<br>name: The name of the image. Mutually exclusive with family.<br>family: The image family to use. Mutually exclusive with name.<br>project: The project where the image is hosted. | `map(string)` | n/a | yes |
| labels | Labels to add to resources. List key, value pairs. | `map(string)` | n/a | yes |
| machine\_type | Machine type to use for HTCondor central managers | `string` | `"n2-standard-4"` | no |
| metadata | Metadata to add to HTCondor central managers | `map(string)` | `{}` | no |
| network\_self\_link | The self link of the network in which the HTCondor central manager will be created. | `string` | `null` | no |
| network\_storage | An array of network attached storage mounts to be configured | <pre>list(object({<br>    server_ip             = string,<br>    remote_mount          = string,<br>    local_mount           = string,<br>    fs_type               = string,<br>    mount_options         = string,<br>    client_install_runner = map(string)<br>    mount_runner          = map(string)<br>  }))</pre> | `[]` | no |
| project\_id | Project in which HTCondor central manager will be created | `string` | n/a | yes |
| region | Default region for creating resources | `string` | n/a | yes |
| service\_account\_scopes | Scopes by which to limit service account attached to central manager. | `set(string)` | <pre>[<br>  "https://www.googleapis.com/auth/cloud-platform"<br>]</pre> | no |
| shielded\_instance\_config | Shielded VM configuration for the instance (must set var.enabled\_shielded\_vm) | <pre>object({<br>    enable_secure_boot          = bool<br>    enable_vtpm                 = bool<br>    enable_integrity_monitoring = bool<br>  })</pre> | <pre>{<br>  "enable_integrity_monitoring": true,<br>  "enable_secure_boot": true,<br>  "enable_vtpm": true<br>}</pre> | no |
| subnetwork\_self\_link | The self link of the subnetwork in which the HTCondor central manager will be created. | `string` | `null` | no |
| update\_policy | Replacement policy for Central Manager ("PROACTIVE" to replace immediately or "OPPORTUNISTIC" to replace upon instance power cycle). | `string` | `"PROACTIVE"` | no |
| zones | Zone(s) in which central manager may be created. If not supplied, will default to all zones in var.region. | `list(string)` | `[]` | no |

## Outputs

| Name | Description |
|------|-------------|
| central\_manager\_ips | IP addresses of the central managers provisioned by this module |
| central\_manager\_name | Name of the central managers provisioned by this module |
| list\_instances\_command | Command to list central managers provisioned by this module |

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
