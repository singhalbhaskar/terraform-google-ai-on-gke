## Description

This module simplifies the following functionality:

* Applying Kubernetes manifests to GKE clusters: It provides flexible options for specifying manifests, allowing you to either directly embed them as strings content or reference them from URLs, files, templates, or entire .yaml and .tftpl files in directories.
* Deploying commonly used infrastructure like [Kueue](https://kueue.sigs.k8s.io/docs/) or [Jobset](https://jobset.sigs.k8s.io/docs/).

> Note: Kueue can work with a variety of frameworks out of the box, find them [here](https://kueue.sigs.k8s.io/docs/tasks/run/)

### Explanation

* **Manifest:**
  * **Raw String:** Specify manifests directly within the module configuration using the `content: manifest_body` format.
  * **File/Template/Directory Reference:** Set `source` to the path to:
    * A single URL to a manifest file. Ex.: `https://github.com/.../myrepo/manifest.yaml`.
    * A single local YAML manifest file (`.yaml`). Ex.: `./manifest.yaml`.
    * A template file (`.tftpl`) to generate a manifest. Ex.: `./template.yaml.tftpl`. You can pass the variables to format the template file in `template_vars`.
    * A directory containing multiple YAML or template files. Ex: `./manifests/`. You can pass the variables to format the template files in `template_vars`.

#### Manifest Example

```yaml
- id: existing-gke-cluster
  source: modules/scheduler/pre-existing-gke-cluster
  settings:
    project_id: $(vars.project_id)
    cluster_name: my-gke-cluster
    region: us-central1

- id: kubectl-apply
  source: modules/management/kubectl-apply
  use: [existing-gke-cluster]
  settings:
    - content: |
        apiVersion: v1
        kind: Namespace
        metadata:
          name: my-namespace
    - source: "https://github.com/kubernetes-sigs/jobset/releases/download/v0.6.0/manifests.yaml"
    - source: $(ghpc_stage("manifests/configmap1.yaml"))
    - source: $(ghpc_stage("manifests/configmap2.yaml.tftpl"))
      template_vars: {name: "dev-config", public: "false"}
    - source: $(ghpc_stage("manifests"))/
      template_vars: {name: "dev-config", public: "false"}
```

#### Pre-build infrastructure Example

```yaml
  - id: workload_component_install
    source: modules/management/kubectl-apply
    use: [gke_cluster]
    settings:
      kueue:
        install: true
        config_path: $(ghpc_stage("manifests/user-provided-kueue-config.yaml"))
      jobset:
        install: true
```

The `config_path` field in `kueue` installation accepts a template file, too. You will need to provide variables for the template using `config_template_vars` field.

```yaml
  - id: workload_component_install
    source: modules/management/kubectl-apply
    use: [gke_cluster]
    settings:
      kueue:
        install: true
        config_path: $(ghpc_stage("manifests/user-provided-kueue-config.yaml.tftpl"))
        config_template_vars: {name: "dev-config", public: "false"}
      jobset:
        install: true
```

> **_NOTE:_**
>
> The `project_id` and `region` settings would be inferred from the deployment variables of the same name, but they are included here for clarity.
>
> Terraform may apply resources in parallel, leading to potential dependency issues. If a resource's dependencies aren't ready, it will be applied again up to 15 times.

## License

<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| apply\_manifests | A list of manifests to apply to GKE cluster using kubectl. For more details see [kubectl module's inputs](kubectl/README.md). | <pre>list(object({<br>    content           = optional(string, null)<br>    source            = optional(string, null)<br>    template_vars     = optional(map(any), null)<br>    server_side_apply = optional(bool, false)<br>    wait_for_rollout  = optional(bool, true)<br>  }))</pre> | `[]` | no |
| cluster\_id | An identifier for the gke cluster resource with format projects/<project\_id>/locations/<region>/clusters/<name>. | `string` | n/a | yes |
| jobset | Install [Jobset](https://github.com/kubernetes-sigs/jobset) which manages a group of K8s [jobs](https://kubernetes.io/docs/concepts/workloads/controllers/job/) as a unit. | <pre>object({<br>    install = optional(bool, false)<br>    version = optional(string, "v0.5.2")<br>  })</pre> | `{}` | no |
| kueue | Install and configure [Kueue](https://kueue.sigs.k8s.io/docs/overview/) workload scheduler. A configuration yaml/template file can be provided with config\_path to be applied right after kueue installation. If a template file provided, its variables can be set to config\_template\_vars. | <pre>object({<br>    install              = optional(bool, false)<br>    version              = optional(string, "v0.8.1")<br>    config_path          = optional(string, null)<br>    config_template_vars = optional(map(any), null)<br>  })</pre> | `{}` | no |
| project\_id | The project ID that hosts the gke cluster. | `string` | n/a | yes |

## Outputs

No outputs.

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
