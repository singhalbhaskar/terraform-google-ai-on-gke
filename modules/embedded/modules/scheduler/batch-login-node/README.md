# terraform-google-ai-on-gke

# Description

This module creates a VM that acts as a login node to test and submit Google
Cloud Batch jobs. It is intended to be used along with the `batch-job-template`
module.

This login node:

- Uses the same VM settings as the first provided `batch-job-template`, such as
  image, machine type, etc...
- Runs the same `startup-script` as the first provided `batch-job-template`.
- Has the same mounted file systems as the provided `batch-job-template`.
- Contains a folder with job templates generated by `batch-job-template` modules.

Since the login node has the same mounted storage and is a homogeneous machine
to the Google Cloud Batch compute VMs, it can be used to inspect shared file
systems and test installed software before submitting a Google Cloud Batch job.

## Example

```yaml
- id: batch-job
  source: modules/scheduler/batch-job-template
  ...

- id: batch-login
  source: modules/scheduler/batch-login-node
  use: [batch-job]
  outputs: [instructions]
```

## Authentication

To submit jobs from the login node, the service account attached to the VM needs
the `Batch Job Administrator` role. In most cases this service account will be
the Compute Engine default service account and will not be granted this role by
default.

You can grant this role either by adding the `Batch Job Administrator` role to
the service account in the IAM page in the Google Cloud Console, or by running
the following command line:

```bash
gcloud projects add-iam-policy-binding <project ID> \
  --member=serviceAccount:<service account email> \
  --role=roles/batch.jobsAdmin
```

## gcloud Batch Access

Until the Google Cloud Batch API is generally available (GA), it may not be
available in all versions of the `gcloud` cli. You can test if the Google Cloud
Batch commands are available by running `gcloud [alpha|beta|] batch -h`. If the
Google Cloud Batch cli is not available it can generally be mitigated by either
updating `gcloud` by running `gcloud components update`, or using an image that
contains a more recent version of `gcloud`.

## License

<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| batch\_job\_directory | The path of the directory on the login node in which to place the Google Cloud Batch job template | `string` | `"/home/batch-jobs"` | no |
| deployment\_name | Name of the deployment, also used for the job\_id | `string` | n/a | yes |
| enable\_oslogin | Enable or Disable OS Login with "ENABLE" or "DISABLE". Set to "INHERIT" to inherit project OS Login setting. | `string` | `"ENABLE"` | no |
| gcloud\_version | The version of the gcloud cli being used. Used for output instructions.<br>Valid inputs are `\"alpha\"`, `\"beta\"` and \"\" (empty string for default<br>version). Typically supplied by a batch-job-template module. If multiple<br>batch-job-template modules supply the gcloud\_version, only the first will be used. | `string` | `""` | no |
| instance\_template | Login VM instance template self-link. Typically supplied by a<br>batch-job-template module. If multiple batch-job-template modules supply the<br>instance\_template, the first will be used. | `string` | n/a | yes |
| job\_data | List of jobs and supporting data for each, typically provided via "use" from the batch-job-template module. | <pre>list(object({<br>    template_contents = string,<br>    filename          = string,<br>    id                = string<br>  }))</pre> | n/a | yes |
| job\_filename | Deprecated (use `job_data`): The filename of the generated job template file. Typically supplied by a batch-job-template module. | `string` | `null` | no |
| job\_id | Deprecated (use `job_data`): The ID for the Google Cloud Batch job. Typically supplied by a batch-job-template module for use in the output instructions. | `string` | `null` | no |
| job\_template\_contents | Deprecated (use `job_data`): The contents of the Google Cloud Batch job template. Typically supplied by a batch-job-template module. | `string` | `null` | no |
| labels | Labels to add to the login node. Key-value pairs | `map(string)` | n/a | yes |
| network\_storage | An array of network attached storage mounts to be configured. Typically supplied by a batch-job-template module. | <pre>list(object({<br>    server_ip             = string<br>    remote_mount          = string<br>    local_mount           = string<br>    fs_type               = string<br>    mount_options         = string<br>    client_install_runner = map(string)<br>    mount_runner          = map(string)<br>  }))</pre> | `[]` | no |
| project\_id | Project in which the HPC deployment will be created | `string` | n/a | yes |
| region | The region in which to create the login node | `string` | n/a | yes |
| startup\_script | Startup script run before Google Cloud Batch job starts. Typically supplied by a batch-job-template module. | `string` | `null` | no |
| zone | The zone in which to create the login node | `string` | n/a | yes |

## Outputs

| Name | Description |
|------|-------------|
| instructions | Instructions for accessing the login node and submitting Google Cloud Batch jobs |
| login\_node\_name | Name of the created VM |

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
