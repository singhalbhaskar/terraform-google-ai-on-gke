# terraform-google-ai-on-gke

## Description

This module creates a [Google Cloud Storage (GCS) bucket](https://cloud.google.com/storage).

For more information on this and other network storage options in the Cluster
Toolkit, see the extended [Network Storage documentation](../../../../docs/network_storage.md).

### Example

The following example will create a bucket named `simulation-results-xxxxxxxx`,
where `xxxxxxxx` is a randomly generated id.

```yaml
  - id: bucket
    source: community/modules/file-system/cloud-storage-bucket
    settings:
      name_prefix: simulation-results
      random_suffix: true
```

> **_NOTE:_** Use of `random_suffix` may cause the following error when used
> with other modules:
> `value depends on resource attributes that cannot be determined until apply`.
> To resolve this set `random_suffix` to `false` (default).

<!-- -->

> **_NOTE:_** Bucket namespace is shared by all users of Google Cloud so it is
> possible to have a bucket name clash with an existing bucket that is not in
> your project. To resolve this try to use a more unique name, or set the
> `random_suffix` variable to `true`.

## Naming of Bucket

There are potentially three parts to the bucket name. Each of these parts are
configurable in the blueprint.

1. A **custom prefix**, provided by the user in the blueprint \
Provide the custom prefix using the `name_prefix` setting.

1. The **deployment name**, included by default \
The deployment name can be excluded by setting `use_deployment_name_in_bucket_name: false`.

1. A **random id** suffix, excluded by default \
The random id can be included by setting `random_suffix: true`.

If none of these are provided (no `name_prefix`,
`use_deployment_name_in_bucket_name: false`, & `random_suffix: false`), then the
bucket name will default to `no-bucket-name-provided`.

Since bucket namespace is shared by all users of Google Cloud, it is more likely
to experience naming clashes than with other resources. In many cases, adding
the `random_suffix` will resolve the naming clash issue.

> **Warning**: If a bucket is created with a `random_suffix` and then used as
> the bucket for a startup script in the same deployment group this will cause a
> `not known at apply time` error in terraform. The solution is to either create
> the bucket in a separate deployment group or to remove the random suffix.

## Mounting

To mount the Cloud Storage bucket you must first ensure that the GCS Fuse client
has been installed and then call the proper `mount` command.

Both of these steps are automatically handled with the use of the `use` command
in a selection of Cluster Toolkit modules. See the [compatibility matrix][matrix] in
the network storage doc for a complete list of supported modules.

If mounting is not automatically handled as described above, the
`cloud-storage-bucket` module outputs runners that can be used with the
`startup-script` module to install the client and mount the file system. See the
following example:

```yaml
  - id: bucket
    source: community/modules/file-system/cloud-storage-bucket
    settings: {local_mount: /data}

  - id: mount-at-startup
    source: modules/scripts/startup-script
    settings:
      runners:
      - $(bucket.client_install_runner)
      - $(bucket.mount_runner)
```

[matrix]: ../../../../docs/network_storage.md#compatibility-matrix

## License

<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| deployment\_name | Name of the HPC deployment; used as part of name of the GCS bucket. | `string` | n/a | yes |
| enable\_hierarchical\_namespace | If true, enables hierarchical namespace for the bucket. This option must be configured during the initial creation of the bucket. | `bool` | `false` | no |
| force\_destroy | If true will destroy bucket with all objects stored within. | `bool` | `false` | no |
| labels | Labels to add to the GCS bucket. Key-value pairs. | `map(string)` | n/a | yes |
| local\_mount | The mount point where the contents of the device may be accessed after mounting. | `string` | `"/mnt"` | no |
| mount\_options | Mount options to be put in fstab. Note: `implicit_dirs` makes it easier to work with objects added by other tools, but there is a performance impact. See: [more information](https://github.com/GoogleCloudPlatform/gcsfuse/blob/master/docs/semantics.md#implicit-directories) | `string` | `"defaults,_netdev,implicit_dirs"` | no |
| name\_prefix | Name Prefix. | `string` | `null` | no |
| project\_id | ID of project in which GCS bucket will be created. | `string` | n/a | yes |
| random\_suffix | If true, a random id will be appended to the suffix of the bucket name. | `bool` | `false` | no |
| region | The region to deploy to | `string` | n/a | yes |
| use\_deployment\_name\_in\_bucket\_name | If true, the deployment name will be included as part of the bucket name. This helps prevent naming clashes across multiple deployments. | `bool` | `true` | no |
| viewers | A list of additional accounts that can read packages from this bucket | `set(string)` | `[]` | no |

## Outputs

| Name | Description |
|------|-------------|
| client\_install\_runner | Runner that performs client installation needed to use gcs fuse. |
| gcs\_bucket\_name | Bucket name. |
| gcs\_bucket\_path | The gsutil bucket path with format of `gs://<bucket-name>`. |
| mount\_runner | Runner that mounts the cloud storage bucket with gcs fuse. |
| network\_storage | Describes a remote network storage to be mounted by fs-tab. |

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
