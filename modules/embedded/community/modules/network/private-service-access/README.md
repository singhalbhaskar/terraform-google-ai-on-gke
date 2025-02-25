## Description

This module configures [private service access][psa] for the VPC specified by
the `network_id` variable. It can be used by the
[Cloud SQL Federation module][sql].

It will automatically perform the following steps, as described in the
[Private Service Access][psa-creation] creation page:

* Create an IP Allocation with the prefix_length specified by the
  `ip_alloc_prefix_length` variable.
* Create a private connection that establishes a [VPC Network Peering][vpcnp]
  connection between your VPC network and the service producer's network

[sql]: https://github.com/GoogleCloudPlatform/hpc-toolkit/tree/main/community/modules/database/slurm-cloudsql-federation
[psa]: https://cloud.google.com/vpc/docs/configure-private-services-access
[psa-creation]: https://cloud.google.com/vpc/docs/configure-private-services-access#procedure
[vpcnp]: https://cloud.google.com/vpc/docs/vpc-peering

### Example

```yaml
  - source: modules/network/vpc
    id: network

  - source: community/modules/network/private-service-access
    id: ps_connect
    use: [network]
```

## License

<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| address | The IP address or beginning of the address range allocated for the private service access. | `string` | `null` | no |
| labels | Labels to add to supporting resources. Key-value pairs. | `map(string)` | n/a | yes |
| network\_id | The ID of the GCE VPC network to configure private service Access.:<br>`projects/<project_id>/global/networks/<network_name>`" | `string` | n/a | yes |
| prefix\_length | The prefix length of the IP range allocated for the private service access. | `number` | `16` | no |
| project\_id | ID of project in which Private Service Access will be created. | `string` | n/a | yes |

## Outputs

| Name | Description |
|------|-------------|
| cidr\_range | CIDR range of the created google\_compute\_global\_address |
| connect\_mode | Services that use Private Service Access typically specify connect\_mode<br>"PRIVATE\_SERVICE\_ACCESS". This output value sets connect\_mode and additionally<br>blocks terraform actions until the VPC connection has been created. |
| private\_vpc\_connection\_peering | The name of the VPC Network peering connection that was created by the service provider. |
| reserved\_ip\_range | Named IP range to be used by services connected with Private Service Access. |

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
