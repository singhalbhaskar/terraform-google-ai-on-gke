# terraform-google-ai-on-gke

## Description

This modules create a [resource policy for compute engines](https://cloud.google.com/compute/docs/instances/placement-policies-overview). This policy can be passed to a gke-node-pool module to apply the policy on the node-pool's nodes.

Note: By default, you can't apply compact placement policies with a max distance value to A3 VMs. To request access to this feature, contact your [Technical Account Manager (TAM)](https://cloud.google.com/tam) or the [Sales team](https://cloud.google.com/contact).

### Example

The following example creates a group placement resource policy and applies it to a gke-node-pool.

```yaml
  - id: group_placement_1
    source: modules/compute/resource-policy
    settings:
      name: gp-np-1
      group_placement_max_distance: 2

  - id: node_pool_1
    source: modules/compute/gke-node-pool
    use: [group_placement_1]
    settings:
      machine_type: e2-standard-8
    outputs: [instructions]
```

<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| group\_placement\_max\_distance | The max distance for group placement policy to use for the node pool's nodes. If set it will add a compact group placement policy.<br>Note: Placement policies have the [following](https://cloud.google.com/compute/docs/instances/placement-policies-overview#restrictions-compact-policies) restrictions. | `number` | `0` | no |
| name | The resource policy's name. | `string` | n/a | yes |
| project\_id | The project ID for the resource policy. | `string` | n/a | yes |
| region | The region for the the resource policy. | `string` | n/a | yes |

## Outputs

| Name | Description |
|------|-------------|
| placement\_policy | Group placement policy to use for placing VMs or GKE nodes placement. `COMPACT` is the only supported value for `type` currently. `name` is the name of the placement policy.<br>It is assumed that the specified policy exists. To create a placement policy refer to https://cloud.google.com/sdk/gcloud/reference/compute/resource-policies/create/group-placement.<br>Note: Placement policies have the [following](https://cloud.google.com/compute/docs/instances/placement-policies-overview#restrictions-compact-policies) restrictions. |

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
