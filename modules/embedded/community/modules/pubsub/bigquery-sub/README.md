# Description

This module creates a Bigquery Pub/Sub Subscription.

Primarily used for FSI - MonteCarlo Tutorial:
**[fsi-montecarlo-on-batch-tutorial]**.

[fsi-montecarlo-on-batch-tutorial]: ../docs/tutorials/fsi-montecarlo-on-batch/README.md

## Example

The following example creates a Bigquery subscription using a Bigquery table and
Pub/Sub topic.

```yaml
  - id: bq_subscription
    source: community/modules/pubsub/bigquery-sub
    use: [bq-table, pubsub_topic]
```

Also see usages in this
[example blueprint](../../../examples/fsi-montecarlo-on-batch.yaml).

## License

<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| dataset\_id | Name of the dataset that was created. Can be provided by the bigquery-table module | `string` | n/a | yes |
| deployment\_name | The name of the current deployment | `string` | n/a | yes |
| labels | Labels to add to the instances. Key-value pairs. | `map(string)` | n/a | yes |
| project\_id | Project in which the HPC deployment will be created | `string` | n/a | yes |
| subscription\_id | The name of the pubsub subscription to be created | `string` | `null` | no |
| table\_id | ID of created BQ table. Can be provided by the bigquery-table module | `string` | n/a | yes |
| topic\_id | The name of the pubsub topic to subscribe to. Can be provided by the pubsub/topic module | `string` | n/a | yes |

## Outputs

| Name | Description |
|------|-------------|
| subscription\_id | Name of the subscription that was created. |

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
