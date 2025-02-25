## Description

This module creates [parallelstore](https://cloud.google.com/parallelstore)
instance. Parallelstore is Google Cloud's first party parallel file system
service based on [Intel DAOS](https://docs.daos.io/v2.2/)

### Supported Operating Systems

A parallelstore instance can be used with Slurm cluster or compute
VM running Ubuntu 22.04, debian 12 or HPC Rocky Linux 8.

### Parallelstore Quota

To get access to a private preview of Parallelstore APIs, your project needs to
be allowlisted. To set this up, please work with your account representative.

### Parallelstore mount options

After parallelstore instance is created, you can specify mount options depending
upon your workload. DAOS is configured to deliver the best user experience for
interactive workloads with aggressive caching. If you are running parallel
workloads concurrently accessing the sane files from multiple client nodes, it
is recommended to disable the writeback cache to avoid cross-client consistency
issues. You can specify different mount options as follows,

```yaml
  - id: parallelstore
    source: modules/file-system/parallelstore
    use: [network, ps_connect]
    settings:
      mount_options: "disable-wb-cache,thread-count=20,eq-count=8"
```

### Example - New VPC

For parallelstore instance, Below snippet creates new VPC and configures private-service-access
for this newly created network.

```yaml
 - id: network
    source: modules/network/vpc

  - id: private_service_access
    source: community/modules/network/private-service-access
    use: [network]
    settings:
      prefix_length: 24

  - id: parallelstore
    source: modules/file-system/parallelstore
    use: [network, private_service_access]
```

### Example - Existing VPC

If you want to use existing network with private-service-access configured, you need
to manually provide `private_vpc_connection_peering` to the parallelstore module.
You can get this details from the Google Cloud Console UI in `VPC network peering`
section. Below is the example of using existing network and creating parallelstore.
If existing network is not configured with private-service-access, you can follow
[Configure private service access](https://cloud.google.com/vpc/docs/configure-private-services-access)
to set it up.

```yaml
  - id: network
    source: modules/network/pre-existing-vpc
    settings:
      network_name: <network_name> // Add network name
      subnetwork_name: <subnetwork_name> // Add subnetwork name

  - id: parallelstore
    source: modules/file-system/parallelstore
    use: [network]
    settings:
      private_vpc_connection_peering: <private_vpc_connection_peering> # will look like "servicenetworking.googleapis.com"
```

### Import data from GCS bucket

You can import data from your GCS bucket to parallelstore instance. Important to
note that data may not be available to the instance immediately. This depends on
latency and size of data. Below is the example of importing data from  bucket.

```yaml
  - id: parallelstore
    source: modules/file-system/parallelstore
    use: [network]
    settings:
      import_gcs_bucket_uri: gs://gcs-bucket/folder-path
      import_destination_path: /gcs/import/
```

Here you can replace `import_gcs_bucket_uri` with the uri of sub folder within GCS
bucket and `import_destination_path` with local directory within parallelstore
instance.

### Additional configuration for DAOS agent and dfuse
Use `daos_agent_config` to provide additional configuration for `daos_agent`, for example:

```yaml
- id: parallelstorefs
  source: modules/file-system/pre-existing-network-storage
  settings:
    daos_agent_config: |
      credential_config:
        cache_expiration: 1m
```

Use `dfuse_environment` to provide additional environment variables for `dfuse` process, for example:

```yaml
- id: parallelstorefs
  source: modules/file-system/parallelstore
  settings:
    dfuse_environment:
      D_LOG_FILE: /tmp/client.log
      D_APPEND_PID_TO_LOG: 1
      D_LOG_MASK: debug
```

<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| daos\_agent\_config | Additional configuration to be added to daos\_config.yml | `string` | `""` | no |
| deployment\_name | Name of the HPC deployment. | `string` | n/a | yes |
| dfuse\_environment | Additional environment variables for DFuse process | `map(string)` | `{}` | no |
| directory\_stripe | The parallelstore stripe level for directories. | `string` | `null` | no |
| file\_stripe | The parallelstore stripe level for files. | `string` | `null` | no |
| import\_destination\_path | The name of local path to import data on parallelstore instance from GCS bucket. | `string` | `null` | no |
| import\_gcs\_bucket\_uri | The name of the GCS bucket to import data from to parallelstore. | `string` | `null` | no |
| labels | Labels to add to parallel store instance. | `map(string)` | `{}` | no |
| local\_mount | The mount point where the contents of the device may be accessed after mounting. | `string` | `"/parallelstore"` | no |
| mount\_options | Options describing various aspects of the parallelstore instance. | `string` | `"disable-wb-cache,thread-count=16,eq-count=8"` | no |
| name | Name of parallelstore instance. | `string` | `null` | no |
| network\_id | The ID of the GCE VPC network to which the instance is connected given in the format:<br>`projects/<project_id>/global/networks/<network_name>`" | `string` | n/a | yes |
| private\_vpc\_connection\_peering | The name of the VPC Network peering connection.<br>If using new VPC, please use community/modules/network/private-service-access to create private-service-access and<br>If using existing VPC with private-service-access enabled, set this manually." | `string` | n/a | yes |
| project\_id | Project in which the HPC deployment will be created. | `string` | n/a | yes |
| size\_gb | Storage size of the parallelstore instance in GB. | `number` | `12000` | no |
| zone | Location for parallelstore instance. | `string` | n/a | yes |

## Outputs

| Name | Description |
|------|-------------|
| instructions | Instructions to monitor import-data operation from GCS bucket to parallelstore. |
| network\_storage | Describes a parallelstore instance. |

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
