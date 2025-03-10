# terraform-google-ai-on-gke

# Description

This module creates:

- **A local file:** A Google Cloud Batch job template file is created. See the
  `instructions` output for the location of the file and instructions on how to
  submit it to Batch.
- **An [instance template]:** This instance template defines the compute settings to
  be used for the Batch job such as network, machine type, image, and startup
  script. This instance template is automatically referenced from the Batch job
  template described above.

[instance template]: https://cloud.google.com/compute/docs/instance-templates

When this module is used with the `batch-login-node` module, the generated
job template will be placed on the login node.

In some cases the job template can be submitted to the Google Cloud Batch API
without modification, but for more complex workloads it is expected that the
user will modify the template after running the Cluster Toolkit.

## Example

```yaml
- id: batch-job
  source: modules/scheduler/batch-job-template
  use: [network1]
  settings:
    runnable: "echo 'hello world'"
    machine_type: n2-standard-4
  outputs: [instructions]
```

See the
[Google Cloud Batch Example](../../../../examples/README.md#serverless-batchyaml--)
for how to use the `batch-job-template` module with other Cluster Toolkit modules such
as `filestore` and `startup-script`.

## Shared VPC

This module supports using a [shared VPC] with a Batch job. To accomplish this,
include a [`pre-existing-vpc`] module that references an existing shared VPC and
then have the `batch-job-template` module `use` the `pre-existing-vpc`.

[shared VPC]: https://cloud.google.com/vpc/docs/shared-vpc
[`pre-existing-vpc`]: https://github.com/GoogleCloudPlatform/hpc-toolkit/tree/main/modules/network/pre-existing-vpc

## Instance Templates

Many of the settings for a Google Cloud Batch job are set using an instance
template, `machine_type` for example. The `batch-job-template` module accomplishes
this by creating an instance template within the module, which is supplied to
the Google Cloud Batch job.

Alternatively, one can supply an instance template to the `batch-job-template`
module using the `instance_template` setting. This supplied instance template
could be generated outside of the Cluster Toolkit (via the Cloud Console UI for
example) or using a separate module within the blueprint. To define an instance
template within a blueprint, one can use the Cloud Foundation Toolkit instance
template module as shown in the following example. This can be useful when
trying to set a property not natively supported in the `batch-job-template` module.

### Example generating instance template using Cloud Foundation Toolkit module

```yaml
deployment_groups:
- group: primary
  modules:
  - id: network1
    source: modules/network/pre-existing-vpc

  - id: appfs
    source: modules/file-system/filestore
    use: [network1]

  - id: batch-startup-script
    source: modules/scripts/startup-script
    settings:
      runners: ...

  - id: batch-compute-template
    source: github.com/terraform-google-modules/terraform-google-vm//modules/instance_template?ref=v7.8.0
    use: [batch-startup-script]
    settings:
      # Boiler plate to work with Cloud Foundation Toolkit
      network: $(network1.network_self_link)
      service_account: {email: null, scopes: ["https://www.googleapis.com/auth/cloud-platform"]}
      access_config: [{nat_ip: null, network_tier: null}]
      # Batch customization
      machine_type: n2-standard-4
      metadata:
        network_storage: ((jsonencode([module.appfs.network_storage])))
      source_image_family: hpc-rocky-linux-8
      source_image_project: cloud-hpc-image-public

  - id: batch-job
    source: modules/scheduler/batch-job-template
    settings:
      instance_template: $(batch-compute-template.self_link)
    outputs: [instructions]
```

## License

<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| allow\_automatic\_updates | If false, disables automatic system package updates on the created instances.  This feature is<br>only available on supported images (or images derived from them).  For more details, see<br>https://cloud.google.com/compute/docs/instances/create-hpc-vm#disable_automatic_updates | `bool` | `true` | no |
| deployment\_name | Name of the deployment, used for the job\_id | `string` | n/a | yes |
| enable\_public\_ips | If set to true, instances will have public IPs | `bool` | `true` | no |
| gcloud\_version | The version of the gcloud cli being used. Used for output instructions. Valid inputs are `"alpha"`, `"beta"` and "" (empty string for default version) | `string` | `""` | no |
| image | DEPRECATED: Google Cloud Batch compute node image. Ignored if `instance_template` is provided. | `any` | `null` | no |
| instance\_image | Google Cloud Batch compute node image. Ignored if `instance_template` is provided.<br><br>Expected Fields:<br>name: The name of the image. Mutually exclusive with family.<br>family: The image family to use. Mutually exclusive with name.<br>project: The project where the image is hosted. | `map(string)` | <pre>{<br>  "family": "hpc-rocky-linux-8",<br>  "project": "cloud-hpc-image-public"<br>}</pre> | no |
| instance\_template | Compute VM instance template self-link to be used for Google Cloud Batch compute node. If provided, a number of other variables will be ignored as noted by `Ignored if instance_template is provided` in descriptions. | `string` | `null` | no |
| job\_filename | The filename of the generated job template file. Will default to `cloud-batch-<job_id>.json` if not specified | `string` | `null` | no |
| job\_id | An id for the Google Cloud Batch job. Used for output instructions and file naming. Automatically populated by the module id if not set. If setting manually, ensure a unique value across all jobs. | `string` | n/a | yes |
| labels | Labels to add to the Google Cloud Batch compute nodes. Key-value pairs. Ignored if `instance_template` is provided. | `map(string)` | n/a | yes |
| log\_policy | Create a block to define log policy.<br>When set to `CLOUD_LOGGING`, logs will be sent to Cloud Logging.<br>When set to `PATH`, path must be added to generated template.<br>When set to `DESTINATION_UNSPECIFIED`, logs will not be preserved. | `string` | `"CLOUD_LOGGING"` | no |
| machine\_type | Machine type to use for Google Cloud Batch compute nodes. Ignored if `instance_template` is provided. | `string` | `"n2-standard-4"` | no |
| mpi\_mode | Sets up barriers before and after each runnable. In addition, sets `permissiveSsh=true`, `requireHostsFile=true`, and `taskCountPerNode=1`. `taskCountPerNode` can be overridden by `task_count_per_node`. | `bool` | `false` | no |
| native\_batch\_mounting | Batch can mount some fs\_type nativly using the 'volumes' block in the job file. If set to false, all mounting will happen through Cluster Toolkit startup scripts. | `bool` | `true` | no |
| network\_storage | An array of network attached storage mounts to be configured. Ignored if `instance_template` is provided. | <pre>list(object({<br>    server_ip             = string<br>    remote_mount          = string<br>    local_mount           = string<br>    fs_type               = string<br>    mount_options         = string<br>    client_install_runner = map(string)<br>    mount_runner          = map(string)<br>  }))</pre> | `[]` | no |
| on\_host\_maintenance | Describes maintenance behavior for the instance. If left blank this will default to `MIGRATE` except the use of GPUs requires it to be `TERMINATE` | `string` | `null` | no |
| project\_id | Project in which the HPC deployment will be created | `string` | n/a | yes |
| region | The region in which to run the Google Cloud Batch job | `string` | n/a | yes |
| runnable | A simplified form of `var.runnables` that only takes a single script. Use either `runnables` or `runnable`. | `string` | `null` | no |
| runnables | A list of shell scripts to be executed in sequence as the main workload of the Google Batch job. These will be used to populate the generated template. | <pre>list(object({<br>    script = string<br>  }))</pre> | `null` | no |
| service\_account | Service account to attach to the Google Cloud Batch compute node. Ignored if `instance_template` is provided. | <pre>object({<br>    email  = string,<br>    scopes = set(string)<br>  })</pre> | <pre>{<br>  "email": null,<br>  "scopes": [<br>    "https://www.googleapis.com/auth/devstorage.read_only",<br>    "https://www.googleapis.com/auth/logging.write",<br>    "https://www.googleapis.com/auth/monitoring.write",<br>    "https://www.googleapis.com/auth/servicecontrol",<br>    "https://www.googleapis.com/auth/service.management.readonly",<br>    "https://www.googleapis.com/auth/trace.append"<br>  ]<br>}</pre> | no |
| startup\_script | Startup script run before Google Cloud Batch job starts. Ignored if `instance_template` is provided. | `string` | `null` | no |
| submit | When set to true, the generated job file will be submitted automatically to Google Cloud as part of terraform apply. | `bool` | `false` | no |
| subnetwork | The subnetwork that the Batch job should run on. Defaults to 'default' subnet. Ignored if `instance_template` is provided. | `any` | `null` | no |
| task\_count | Number of parallel tasks | `number` | `1` | no |
| task\_count\_per\_node | Max number of tasks that can be run on a VM at the same time. If not specified, Batch will decide a value. | `number` | `null` | no |

## Outputs

| Name | Description |
|------|-------------|
| gcloud\_version | The version of gcloud to be used. |
| instance\_template | Instance template used by the Batch job. |
| instructions | Instructions for submitting the Batch job. |
| job\_data | All data associated with the defined job, typically provided as input to clout-batch-login-node. |
| network\_storage | An array of network attached storage mounts used by the Batch job. |
| startup\_script | Startup script run before Google Cloud Batch job starts. |

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
