# terraform-google-ai-on-gke

## Description

This module is used to create a Kubernetes job template file.

The job template file can be submitted as is or used as a template for further
customization. Add the `instructions` output to a blueprint (as shown below) to
get instructions on how to use `kubectl` to submit the job.

This module is designed to `use` one or more `gke-node-pool` modules. The job
will be configured to run on any of the specified node pools.

> **_NOTE:_** This is an experimental module and the functionality and
> documentation will likely be updated in the near future. This module has only
> been tested in limited capacity.

### Example

The following example creates a GKE job template file.

```yaml
  - id: job-template
    source: modules/compute/gke-job-template
    use: [compute_pool]
    settings:
      node_count: 3
    outputs: [instructions]
```

Also see a full [GKE example blueprint](../../../examples/hpc-gke.yaml).

### Storage Options

This module natively supports:

* Filestore as a shared file system between pods/nodes.
* Pod level ephemeral storage options:
  * memory backed emptyDir
  * local SSD backed emptyDir
  * SSD persistent disk backed ephemeral volume
  * balanced persistent disk backed ephemeral volume

See the [storage-gke.yaml blueprint](../../../examples/storage-gke.yaml) and the
associated [documentation](../../../../examples/README.md#storage-gkeyaml--) for
examples of how to use Filestore and ephemeral storage with this module.

### Requested Resources

When one or more `gke-node-pool` modules are referenced with the `use` field.
The requested resources will be populated to achieve a 1 pod per node packing
while still leaving some headroom for required system pods.

This functionality can be overridden by specifying the desired cpu requirement
using the `requested_cpu_per_pod` setting.

## License

<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| allocatable\_cpu\_per\_node | The allocatable cpu per node. Used to claim whole nodes. Generally populated from gke-node-pool via `use` field. | `list(number)` | <pre>[<br>  -1<br>]</pre> | no |
| allocatable\_gpu\_per\_node | The allocatable gpu per node. Used to claim whole nodes. Generally populated from gke-node-pool via `use` field. | `list(number)` | <pre>[<br>  -1<br>]</pre> | no |
| backoff\_limit | Controls the number of retries before considering a Job as failed. Set to zero for shared fate. | `number` | `0` | no |
| command | The command and arguments for the container that run in the Pod. The command field corresponds to entrypoint in some container runtimes. | `list(string)` | <pre>[<br>  "hostname"<br>]</pre> | no |
| completion\_mode | Sets value of `completionMode` on the job. Default uses indexed jobs. See [documentation](https://kubernetes.io/blog/2021/04/19/introducing-indexed-jobs/) for more information | `string` | `"Indexed"` | no |
| ephemeral\_volumes | Will create an emptyDir or ephemeral volume that is backed by the specified type: `memory`, `local-ssd`, `pd-balanced`, `pd-ssd`. `size_gb` is provided in GiB. | <pre>list(object({<br>    type       = string<br>    mount_path = string<br>    size_gb    = number<br>  }))</pre> | `[]` | no |
| has\_gpu | Indicates that the job should request nodes with GPUs. Typically supplied by a gke-node-pool module. | `list(bool)` | <pre>[<br>  false<br>]</pre> | no |
| image | The container image the job should use. | `string` | `"debian"` | no |
| k8s\_service\_account\_name | Kubernetes service account to run the job as. If null then no service account is specified. | `string` | `null` | no |
| labels | Labels to add to the GKE job template. Key-value pairs. | `map(string)` | n/a | yes |
| machine\_family | The machine family to use in the node selector (example: `n2`). If null then machine family will not be used as selector criteria. | `string` | `null` | no |
| name | The name of the job. | `string` | `"my-job"` | no |
| node\_count | How many nodes the job should run in parallel. | `number` | `1` | no |
| node\_pool\_name | A list of node pool names on which to run the job. Can be populated via `use` field. | `list(string)` | `[]` | no |
| node\_selectors | A list of node selectors to use to place the job. | <pre>list(object({<br>    key   = string<br>    value = string<br>  }))</pre> | `[]` | no |
| persistent\_volume\_claims | A list of objects that describes a k8s PVC that is to be used and mounted on the job. Generally supplied by the gke-persistent-volume module. | <pre>list(object({<br>    name          = string<br>    mount_path    = string<br>    mount_options = string<br>    is_gcs        = bool<br>  }))</pre> | `[]` | no |
| random\_name\_sufix | Appends a random suffix to the job name to avoid clashes. | `bool` | `true` | no |
| requested\_cpu\_per\_pod | The requested cpu per pod. If null, allocatable\_cpu\_per\_node will be used to claim whole nodes. If provided will override allocatable\_cpu\_per\_node. | `number` | `-1` | no |
| requested\_gpu\_per\_pod | The requested gpu per pod. If null, allocatable\_gpu\_per\_node will be used to claim whole nodes. If provided will override allocatable\_gpu\_per\_node. | `number` | `-1` | no |
| restart\_policy | Job restart policy. Only a RestartPolicy equal to `Never` or `OnFailure` is allowed. | `string` | `"Never"` | no |
| security\_context | The security options the container should be run with. More info: https://kubernetes.io/docs/tasks/configure-pod-container/security-context/ | <pre>list(object({<br>    key   = string<br>    value = string<br>  }))</pre> | `[]` | no |
| tolerations | Tolerations allow the scheduler to schedule pods with matching taints. Generally populated from gke-node-pool via `use` field. | <pre>list(object({<br>    key      = string<br>    operator = string<br>    value    = string<br>    effect   = string<br>  }))</pre> | <pre>[<br>  {<br>    "effect": "NoSchedule",<br>    "key": "user-workload",<br>    "operator": "Equal",<br>    "value": "true"<br>  }<br>]</pre> | no |

## Outputs

| Name | Description |
|------|-------------|
| instructions | Instructions for submitting the GKE job. |

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
