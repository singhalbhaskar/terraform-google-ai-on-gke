## Description

Creates a Pub/Sub topic

Primarily used for FSI - MonteCarlo Tutorial: **[fsi-montecarlo-on-batch-tutorial]**.

[fsi-montecarlo-on-batch-tutorial]: ../docs/tutorials/fsi-montecarlo-on-batch/README.md

### Example

The following example creates a Pub/Sub topic.

```yaml
  - id: pubsub_topic
    source: community/modules/pubsub/topic
```

Also see usages in this
[example blueprint](../../../examples/fsi-montecarlo-on-batch.yaml).

## License

<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| deployment\_name | The name of the current deployment | `string` | n/a | yes |
| labels | Labels to add to the instances. Key-value pairs. | `map(string)` | n/a | yes |
| project\_id | Project in which the HPC deployment will be created | `string` | n/a | yes |
| schema\_id | The name of the pubsub schema to be created | `string` | `null` | no |
| schema\_json | The JSON definition of the pubsub topic schema | `string` | `"{  \n  \"name\" : \"Avro\",  \n  \"type\" : \"record\", \n  \"fields\" : \n      [\n       {\"name\" : \"ticker\", \"type\" : \"string\"},\n       {\"name\" : \"epoch_time\", \"type\" : \"int\"},\n       {\"name\" : \"iteration\", \"type\" : \"int\"},\n       {\"name\" : \"start_date\", \"type\" : \"string\"},\n       {\"name\" : \"end_date\", \"type\" : \"string\"},\n       {\n           \"name\":\"simulation_results\",\n           \"type\":{\n               \"type\": \"array\",  \n               \"items\":{\n                   \"name\":\"Child\",\n                   \"type\":\"record\",\n                   \"fields\":[\n                       {\"name\":\"price\", \"type\":\"double\"}\n                   ]\n               }\n           }\n       }\n      ]\n }\n"` | no |
| topic\_id | The name of the pubsub topic to be created | `string` | `null` | no |

## Outputs

| Name | Description |
|------|-------------|
| topic\_id | Name of the topic that was created. |
| topic\_schema | Name of the topic schema that was created. |

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
