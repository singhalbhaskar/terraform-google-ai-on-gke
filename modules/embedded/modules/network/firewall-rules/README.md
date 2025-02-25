# terraform-google-ai-on-gke

## Description

This module facilitates the creation of custom firewall rules for existing
networks.

## Example usage

This module can be used by other Toolkit modules to create application-specific
firewall rules or in conjunction with the [pre-existing-vpc] module to enable
traffic in existing networks. The snippet below is drawn from the
[ml-slurm.yaml] example:

```yaml
- group: primary
  modules:
  - id: network
    source: modules/network/pre-existing-vpc

  # this example anticipates that the VPC default network has internal traffic
  # allowed and IAP tunneling for SSH connections
  - id: firewall_rule
    source: modules/network/firewall-rules
    use:
    - network
    settings:
      ingress_rules:
      - name: $(vars.deployment_name)-allow-internal-traffic
        description: Allow internal traffic
        destination_ranges:
        - $(network.subnetwork_address)
        source_ranges:
        - $(network.subnetwork_address)
        allow:
        - protocol: tcp
          ports:
          - 0-65535
        - protocol: udp
          ports:
          - 0-65535
        - protocol: icmp
      - name: $(vars.deployment_name)-allow-iap-ssh
        description: Allow IAP-tunneled SSH connections
        destination_ranges:
        - $(network.subnetwork_address)
        source_ranges:
        - 35.235.240.0/20
        allow:
        - protocol: tcp
          ports:
          - 22
```

<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| egress\_rules | List of egress rules | <pre>list(object({<br>    name                    = string<br>    description             = optional(string, null)<br>    disabled                = optional(bool, null)<br>    priority                = optional(number, null)<br>    destination_ranges      = optional(list(string), [])<br>    source_ranges           = optional(list(string), [])<br>    source_tags             = optional(list(string))<br>    source_service_accounts = optional(list(string))<br>    target_tags             = optional(list(string))<br>    target_service_accounts = optional(list(string))<br><br>    allow = optional(list(object({<br>      protocol = string<br>      ports    = optional(list(string))<br>    })), [])<br>    deny = optional(list(object({<br>      protocol = string<br>      ports    = optional(list(string))<br>    })), [])<br>    log_config = optional(object({<br>      metadata = string<br>    }))<br>  }))</pre> | `[]` | no |
| ingress\_rules | List of ingress rules | <pre>list(object({<br>    name                    = string<br>    description             = optional(string, null)<br>    disabled                = optional(bool, null)<br>    priority                = optional(number, null)<br>    destination_ranges      = optional(list(string), [])<br>    source_ranges           = optional(list(string), [])<br>    source_tags             = optional(list(string))<br>    source_service_accounts = optional(list(string))<br>    target_tags             = optional(list(string))<br>    target_service_accounts = optional(list(string))<br><br>    allow = optional(list(object({<br>      protocol = string<br>      ports    = optional(list(string))<br>    })), [])<br>    deny = optional(list(object({<br>      protocol = string<br>      ports    = optional(list(string))<br>    })), [])<br>    log_config = optional(object({<br>      metadata = string<br>    }))<br>  }))</pre> | `[]` | no |
| subnetwork\_self\_link | The self link of the subnetwork whose global network firewall rules will be modified. | `string` | n/a | yes |

## Outputs

No outputs.

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->

[pre-existing-vpc]: ../pre-existing-vpc/README.md
[ml-slurm.yaml]: ../../../examples/ml-slurm.yaml
