# terraform-google-ai-on-gke

## Description

This module creates a Google Kubernetes Engine
([GKE](https://cloud.google.com/kubernetes-engine)) cluster.

> **_NOTE:_** This is an experimental module and the functionality and
> documentation will likely be updated in the near future. This module has only
> been tested in limited capacity.

### Example

The following example creates a GKE cluster and a VPC designed to work with GKE.
See [VPC Network](#vpc-network) section for more information about network
requirements.

```yaml
  - id: network1
    source: modules/network/vpc
    settings:
      subnetwork_name: gke-subnet
      secondary_ranges:
        gke-subnet:
        - range_name: pods
          ip_cidr_range: 10.4.0.0/14
        - range_name: services
          ip_cidr_range: 10.0.32.0/20

  - id: gke_cluster
    source: modules/scheduler/gke-cluster
    use: [network1]
```

Also see a full [GKE example blueprint](../../../examples/hpc-gke.yaml).

### VPC Network

This module is configured to create a
[VPC-native cluster](https://cloud.google.com/kubernetes-engine/docs/concepts/alias-ips).
This means that alias IPs are used and that the subnetwork requires secondary
ranges for pods and services. In the example shown above these secondary ranges
are created in the VPC module. By default the `gke-cluster` module will look for
ranges with the names `pods` and `services`. These names can be configured using
the `pods_ip_range_name` and `services_ip_range_name` settings.

### Multi-networking

To [enable Multi-networking](https://cloud.google.com/kubernetes-engine/docs/how-to/gpu-bandwidth-gpudirect-tcpx#create-gke-environment), pass multivpc module to gke-cluster module as described in example below. Passing a multivpc module enables multi networking and [Dataplane V2](https://cloud.google.com/kubernetes-engine/docs/concepts/dataplane-v2?hl=en) on the cluster.

```yaml
  - id: network
    source: modules/network/vpc
    settings:
      subnetwork_name: gke-subnet
      secondary_ranges:
        gke-subnet:
        - range_name: pods
          ip_cidr_range: 10.4.0.0/14
        - range_name: services
          ip_cidr_range: 10.0.32.0/20

  - id: multinetwork
    source: modules/network/multivpc
    settings:
      network_name_prefix: multivpc-net
      network_count: 8
      global_ip_address_range: 172.16.0.0/12
      subnetwork_cidr_suffix: 16

  - id: gke-cluster
    source: modules/scheduler/gke-cluster
    use: [network, multinetwork] ## enables multi networking and Dataplane V2 on cluster
    settings:
      cluster_name: $(vars.deployment_name)
```

Find an example of multi networking in GKE [here](../../../examples/gke-a3-megagpu.yaml).

### Cluster Limitations

The current implementations has the following limitations:

- Autopilot is disabled
- Auto-provisioning of new node pools is disabled
- Network policies are not supported
- General addon configuration is not supported
- Only regional cluster is supported

## License

<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| additional\_networks | Additional network interface details for GKE, if any. Providing additional networks enables multi networking and creates relevat network objects on the cluster. | <pre>list(object({<br>    network            = string<br>    subnetwork         = string<br>    subnetwork_project = string<br>    network_ip         = string<br>    nic_type           = string<br>    stack_type         = string<br>    queue_count        = number<br>    access_config = list(object({<br>      nat_ip       = string<br>      network_tier = string<br>    }))<br>    ipv6_access_config = list(object({<br>      network_tier = string<br>    }))<br>    alias_ip_range = list(object({<br>      ip_cidr_range         = string<br>      subnetwork_range_name = string<br>    }))<br>  }))</pre> | `[]` | no |
| authenticator\_security\_group | The name of the RBAC security group for use with Google security groups in Kubernetes RBAC. Group name must be in format gke-security-groups@yourdomain.com | `string` | `null` | no |
| autoscaling\_profile | (Beta) Optimize for utilization or availability when deciding to remove nodes. Can be BALANCED or OPTIMIZE\_UTILIZATION. | `string` | `"OPTIMIZE_UTILIZATION"` | no |
| cluster\_availability\_type | Type of cluster availability. Possible values are: {REGIONAL, ZONAL} | `string` | `"REGIONAL"` | no |
| cluster\_reference\_type | How the google\_container\_node\_pool.system\_node\_pools refers to the cluster. Possible values are: {SELF\_LINK, NAME} | `string` | `"SELF_LINK"` | no |
| configure\_workload\_identity\_sa | When true, a kubernetes service account will be created and bound using workload identity to the service account used to create the cluster. | `bool` | `false` | no |
| default\_max\_pods\_per\_node | The default maximum number of pods per node in this cluster. | `number` | `null` | no |
| deletion\_protection | "Determines if the cluster can be deleted by gcluster commands or not".<br>To delete a cluster provisioned with deletion\_protection set to true, you must first set it to false and apply the changes.<br>Then proceed with deletion as usual. | `bool` | `false` | no |
| deployment\_name | Name of the HPC deployment. Used in the GKE cluster name by default and can be configured with `prefix_with_deployment_name`. | `string` | n/a | yes |
| enable\_dataplane\_v2 | Enables [Dataplane v2](https://cloud.google.com/kubernetes-engine/docs/concepts/dataplane-v2). This setting is immutable on clusters. If null, will default to false unless using multi-networking, in which case it will default to true | `bool` | `null` | no |
| enable\_dcgm\_monitoring | Enable GKE to collect DCGM metrics | `bool` | `false` | no |
| enable\_filestore\_csi | The status of the Filestore Container Storage Interface (CSI) driver addon, which allows the usage of filestore instance as volumes. | `bool` | `false` | no |
| enable\_gcsfuse\_csi | The status of the GCSFuse Filestore Container Storage Interface (CSI) driver addon, which allows the usage of a gcs bucket as volumes. | `bool` | `false` | no |
| enable\_master\_global\_access | Whether the cluster master is accessible globally (from any region) or only within the same region as the private endpoint. | `bool` | `false` | no |
| enable\_multi\_networking | Enables [multi networking](https://cloud.google.com/kubernetes-engine/docs/how-to/setup-multinetwork-support-for-pods#create-a-gke-cluster) (Requires GKE Enterprise). This setting is immutable on clusters and enables [Dataplane V2](https://cloud.google.com/kubernetes-engine/docs/concepts/dataplane-v2?hl=en). If null, will determine state based on if additional\_networks are passed in. | `bool` | `null` | no |
| enable\_node\_local\_dns\_cache | Enable GKE NodeLocal DNSCache addon to improve DNS lookup latency | `bool` | `false` | no |
| enable\_parallelstore\_csi | The status of the Google Compute Engine Parallelstore Container Storage Interface (CSI) driver addon, which allows the usage of a parallelstore as volumes. | `bool` | `false` | no |
| enable\_persistent\_disk\_csi | The status of the Google Compute Engine Persistent Disk Container Storage Interface (CSI) driver addon, which allows the usage of a PD as volumes. | `bool` | `true` | no |
| enable\_private\_endpoint | (Beta) Whether the master's internal IP address is used as the cluster endpoint. | `bool` | `true` | no |
| enable\_private\_ipv6\_google\_access | The private IPv6 google access type for the VMs in this subnet. | `bool` | `true` | no |
| enable\_private\_nodes | (Beta) Whether nodes have internal IP addresses only. | `bool` | `true` | no |
| gcp\_public\_cidrs\_access\_enabled | Whether the cluster master is accessible via all the Google Compute Engine Public IPs. To view this list of IP addresses look here https://cloud.google.com/compute/docs/faq#find_ip_range | `bool` | `false` | no |
| labels | GCE resource labels to be applied to resources. Key-value pairs. | `map(string)` | n/a | yes |
| maintenance\_exclusions | List of maintenance exclusions. A cluster can have up to three. | <pre>list(object({<br>    name            = string<br>    start_time      = string<br>    end_time        = string<br>    exclusion_scope = string<br>  }))</pre> | `[]` | no |
| maintenance\_start\_time | Start time for daily maintenance operations. Specified in GMT with `HH:MM` format. | `string` | `"09:00"` | no |
| master\_authorized\_networks | External network that can access Kubernetes master through HTTPS. Must be specified in CIDR notation. | <pre>list(object({<br>    cidr_block   = string<br>    display_name = string<br>  }))</pre> | `[]` | no |
| master\_ipv4\_cidr\_block | (Beta) The IP range in CIDR notation to use for the hosted master network. | `string` | `"172.16.0.32/28"` | no |
| min\_master\_version | The minimum version of the master. If unset, the cluster's version will be set by GKE to the version of the most recent official release. | `string` | `null` | no |
| name\_suffix | Custom cluster name postpended to the `deployment_name`. See `prefix_with_deployment_name`. | `string` | `""` | no |
| network\_id | The ID of the GCE VPC network to host the cluster given in the format: `projects/<project_id>/global/networks/<network_name>`. | `string` | n/a | yes |
| networking\_mode | Determines whether alias IPs or routes will be used for pod IPs in the cluster. Options are VPC\_NATIVE or ROUTES. VPC\_NATIVE enables IP aliasing. The default is VPC\_NATIVE. | `string` | `"VPC_NATIVE"` | no |
| pods\_ip\_range\_name | The name of the secondary subnet ip range to use for pods. | `string` | `"pods"` | no |
| prefix\_with\_deployment\_name | If true, cluster name will be prefixed by `deployment_name` (ex: <deployment\_name>-<name\_suffix>). | `bool` | `true` | no |
| project\_id | The project ID to host the cluster in. | `string` | n/a | yes |
| region | The region to host the cluster in. | `string` | n/a | yes |
| release\_channel | The release channel of this cluster. Accepted values are `UNSPECIFIED`, `RAPID`, `REGULAR` and `STABLE`. | `string` | `"UNSPECIFIED"` | no |
| service\_account | DEPRECATED: use service\_account\_email and scopes. | <pre>object({<br>    email  = string,<br>    scopes = set(string)<br>  })</pre> | `null` | no |
| service\_account\_email | Service account e-mail address to use with the system node pool | `string` | `null` | no |
| service\_account\_scopes | Scopes to to use with the system node pool. | `set(string)` | <pre>[<br>  "https://www.googleapis.com/auth/cloud-platform"<br>]</pre> | no |
| services\_ip\_range\_name | The name of the secondary subnet range to use for services. | `string` | `"services"` | no |
| subnetwork\_self\_link | The self link of the subnetwork to host the cluster in. | `string` | n/a | yes |
| system\_node\_pool\_disk\_size\_gb | Size of disk for each node of the system node pool. | `number` | `100` | no |
| system\_node\_pool\_disk\_type | Disk type for each node of the system node pool. | `string` | `null` | no |
| system\_node\_pool\_enable\_secure\_boot | Enable secure boot for the nodes.  Keep enabled unless custom kernel modules need to be loaded. See [here](https://cloud.google.com/compute/shielded-vm/docs/shielded-vm#secure-boot) for more info. | `bool` | `true` | no |
| system\_node\_pool\_enabled | Create a system node pool. | `bool` | `true` | no |
| system\_node\_pool\_image\_type | The default image type used by NAP once a new node pool is being created. Use either COS\_CONTAINERD or UBUNTU\_CONTAINERD. | `string` | `"COS_CONTAINERD"` | no |
| system\_node\_pool\_kubernetes\_labels | Kubernetes labels to be applied to each node in the node group. Key-value pairs. <br>(The `kubernetes.io/` and `k8s.io/` prefixes are reserved by Kubernetes Core components and cannot be specified) | `map(string)` | `null` | no |
| system\_node\_pool\_machine\_type | Machine type for the system node pool. | `string` | `"e2-standard-4"` | no |
| system\_node\_pool\_name | Name of the system node pool. | `string` | `"system"` | no |
| system\_node\_pool\_node\_count | The total min and max nodes to be maintained in the system node pool. | <pre>object({<br>    total_min_nodes = number<br>    total_max_nodes = number<br>  })</pre> | <pre>{<br>  "total_max_nodes": 10,<br>  "total_min_nodes": 2<br>}</pre> | no |
| system\_node\_pool\_taints | Taints to be applied to the system node pool. | <pre>list(object({<br>    key    = string<br>    value  = any<br>    effect = string<br>  }))</pre> | <pre>[<br>  {<br>    "effect": "NO_SCHEDULE",<br>    "key": "components.gke.io/gke-managed-components",<br>    "value": true<br>  }<br>]</pre> | no |
| timeout\_create | Timeout for creating a node pool | `string` | `null` | no |
| timeout\_update | Timeout for updating a node pool | `string` | `null` | no |
| upgrade\_settings | Defines gke cluster upgrade settings. It is highly recommended that you define all max\_surge and max\_unavailable.<br>If max\_surge is not specified, it would be set to a default value of 0.<br>If max\_unavailable is not specified, it would be set to a default value of 1. | <pre>object({<br>    strategy        = string<br>    max_surge       = optional(number)<br>    max_unavailable = optional(number)<br>  })</pre> | <pre>{<br>  "max_surge": 0,<br>  "max_unavailable": 1,<br>  "strategy": "SURGE"<br>}</pre> | no |
| version\_prefix | If provided, Terraform will only return versions that match the string prefix. For example, `1.31.` will match all `1.31` series releases. Since this is just a string match, it's recommended that you append a `.` after minor versions to ensure that prefixes such as `1.3` don't match versions like `1.30.1-gke.10` accidentally. | `string` | `"1.31."` | no |
| zone | Zone for a zonal cluster. | `string` | `null` | no |

## Outputs

| Name | Description |
|------|-------------|
| cluster\_id | An identifier for the resource with format projects/{{project\_id}}/locations/{{region}}/clusters/{{name}}. |
| gke\_ca\_cert | GKe cluster's ca cert. |
| gke\_cluster\_exists | A static flag that signals to downstream modules that a cluster has been created. Needed by community/modules/scripts/kubernetes-operations. |
| gke\_endpoint | GKe cluster's endpoint. |
| gke\_version | GKE cluster's version. |
| instructions | Instructions on how to connect to the created cluster. |
| k8s\_service\_account\_name | Name of k8s service account. |

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
