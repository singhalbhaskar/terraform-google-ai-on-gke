# terraform-google-ai-on-gke

## Description

This module enables topology on a Google Kubernetes Engine cluster.
This is implemented based on sources and instructions explained [here](https://github.com/GoogleCloudPlatform/container-engine-accelerators/tree/master/gpudirect-tcpxo/topology-scheduler).

## Prerequisites

For topology awareness to be enabled, a GKE node pool has to be created with
compact placement. Specifically, the `physical_host` attribute
[ref](https://cloud.google.com/compute/docs/instances/use-compact-placement-policies#verify-vm-location)
should be present for each GPU node in the cluster.

### Example

The following example installs topology scheduler on a GKE cluster.

```yaml
- id: topology_aware_scheduler_install
    source: community/modules/compute/gke-topology-scheduler
    use: [gke_cluster]
```

<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| cluster\_id | projects/{{project}}/locations/{{location}}/clusters/{{cluster}} | `string` | n/a | yes |
| project\_id | The project ID to host the cluster in. | `string` | n/a | yes |

## Outputs

No outputs.

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
