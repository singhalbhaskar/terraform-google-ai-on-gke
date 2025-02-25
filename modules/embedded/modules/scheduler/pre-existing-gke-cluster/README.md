## Description

This module discovers a Google Kubernetes Engine ([GKE](https://cloud.google.com/kubernetes-engine)) cluster that already exists in Google Cloud and
outputs cluster attributes that uniquely identify it for use by other modules.
The module outputs are aligned with the [gke-cluster module][gke-cluster] so that it can be used
as a drop-in substitute when a GKE cluster already exists.

The below sample blueprint discovers the existing GKE cluster named "my-gke-cluster" in "us-central1" region. With the `use` keyword, the
[gke-node-pool] module accepts the `cluser_id`
input variable that uniquely identifies the existing GKE cluster in which the
GKE node pool will be created.

[gke-cluster]: ../gke-cluster/README.md
[gke-node-pool]: ../../compute/gke-node-pool/README.md

### Example

```yaml
- id: existing-gke-cluster
  source: modules/scheduler/pre-existing-gke-cluster
  settings:
    project_id: $(vars.project_id)
    cluster_name: my-gke-cluster
    region: us-central1

- id: compute_pool
  source: modules/compute/gke-node-pool
  use: [existing-gke-cluster]
```

> **_NOTE:_** The `project_id` and `region` settings would be inferred from the
> deployment variables of the same name, but they are included here for clarity.

### Multi-networking

To create network objects in GKE cluster, you can pass a multivpc module to a pre-existing-gke-cluster module instead of [applying a manifest manually](https://cloud.google.com/kubernetes-engine/docs/how-to/gpu-bandwidth-gpudirect-tcpx#create-gke-environment).

```yaml
  - id: network
    source: modules/network/vpc

  - id: multinetwork
    source: modules/network/multivpc
    settings:
      network_name_prefix: multivpc-net
      network_count: 8
      global_ip_address_range: 172.16.0.0/12
      subnetwork_cidr_suffix: 16

  - id: existing-gke-cluster ## multinetworking must be enabled in advance when cluster creation
    source: modules/scheduler/pre-existing-gke-cluster
    use: [multinetwork]
    settings:
      cluster_name: $(vars.deployment_name)
```

## License

<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| additional\_networks | Additional network interface details for GKE, if any. Providing additional networks creates relevat network objects on the cluster. | <pre>list(object({<br>    network            = string<br>    subnetwork         = string<br>    subnetwork_project = string<br>    network_ip         = string<br>    nic_type           = string<br>    stack_type         = string<br>    queue_count        = number<br>    access_config = list(object({<br>      nat_ip       = string<br>      network_tier = string<br>    }))<br>    ipv6_access_config = list(object({<br>      network_tier = string<br>    }))<br>    alias_ip_range = list(object({<br>      ip_cidr_range         = string<br>      subnetwork_range_name = string<br>    }))<br>  }))</pre> | `[]` | no |
| cluster\_name | Name of the existing cluster | `string` | n/a | yes |
| project\_id | Project that hosts the existing cluster | `string` | n/a | yes |
| rdma\_subnetwork\_name\_prefix | Prefix of the RDMA subnetwork names | `string` | `null` | no |
| region | Region in which to search for the cluster | `string` | n/a | yes |

## Outputs

| Name | Description |
|------|-------------|
| cluster\_id | An identifier for the gke cluster with format projects/{{project\_id}}/locations/{{region}}/clusters/{{name}}. |
| gke\_cluster\_exists | A static flag that signals to downstream modules that a cluster exists. |
| gke\_version | GKE cluster's version. |

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
