# terraform-google-ai-on-gke

## Description

This module creates a Toolkit runner that will install HTCondor on RedHat 7 or
8 and its derivative operating systems. These include the CentOS 7 and Rocky
Linux 8 releases of the [HPC VM Image][hpcvmimage]. It may also function on
RedHat 9 and derivatives, however it is not yet supported. Please report any
[issues] on these 3 distributions or open a [discussion] to request support on
Debian or Ubuntu distributions.

[issues]: https://github.com/GoogleCloudPlatform/hpc-toolkit/issues
[discussion]: https://github.com/GoogleCloudPlatform/hpc-toolkit/discussions

It also exports a list of Google Cloud APIs which must be enabled prior to
provisioning an HTCondor Pool.

It is expected to be used with the [htcondor-setup] and
[htcondor-execute-point] modules.

[hpcvmimage]: https://cloud.google.com/compute/docs/instances/create-hpc-vm
[htcondor-setup]: ../../scheduler/htcondor-setup/README.md
[htcondor-execute-point]: ../../compute/htcondor-execute-point/README.md

### Example

The following code snippet uses this module to create startup scripts that
install the HTCondor software into a custom VM image.

```yaml
deployment_groups:
- group: primary
  modules:
  - id: network1
    source: modules/network/vpc
    outputs:
    - network_name

  - id: htcondor_install
    source: community/modules/scripts/htcondor-install

  - id: htcondor_install_script
    source: modules/scripts/startup-script
    use:
    - htcondor_install

- group: packer
  modules:
  - id: custom-image
    source: modules/packer/custom-image
    kind: packer
    use:
    - network1
    - htcondor_install_script
    settings:
      disk_size: 50
      source_image_family: hpc-rocky-linux-8
      image_family: "htcondor-10x"
```

A full example can be found in the [examples README][htc-example].

[htc-example]: ../../../../examples/README.md#htc-htcondoryaml--

## Important note

All POSIX users and HTCondor jobs can act as the service account attached to
VMs within the pool. This enables the use of IAM restrictions via service
accounts but also allows users to access services to which system daemons need
access (e.g. to create Cloud Logging entries). If this is undesirable, one can
restrict access to the instance metadata server to the `root` and `condor`
users. This will allow system services to use the service account, but not
other POSIX users or HTCondor jobs. The firewall example below is appropriate
for CentOS 7.

```shell
firewall-cmd --direct --permanent --add-rule ipv4 filter OUTPUT_direct 1 \
    -m owner --uid-owner root -p tcp -d metadata.google.internal --dport 80 -j ACCEPT
firewall-cmd --direct --permanent --add-rule ipv4 filter OUTPUT_direct 2 \
    -m owner --uid-owner condor -p tcp -d metadata.google.internal --dport 80 -j ACCEPT
firewall-cmd --direct --permanent --add-rule ipv4 filter OUTPUT_direct 3 \
    -p tcp -d metadata.google.internal --dport 80 -j DROP
firewall-cmd --direct --permanent --add-rule ipv4 filter OUTPUT_direct 4 \
    -p tcp -d metadata.google.internal --dport 8080 -j DROP
firewall-cmd --permanent --zone=public --add-port=9618/tcp
firewall-cmd --reload
```

## Support

HTCondor is maintained by the [Center for High Throughput Computing][chtc] at
the University of Wisconsin-Madison. Support for HTCondor is available via:

- [Discussion lists](https://htcondor.org/mail-lists/)
- [HTCondor on GitHub](https://github.com/htcondor/htcondor/)
- [HTCondor manual](https://htcondor.readthedocs.io/en/latest/)

[chtc]: https://chtc.cs.wisc.edu/

## License

<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| condor\_version | Yum/DNF-compatible version string; leave unset to use latest 23.0 LTS release (examples: "23.0.0","23.*")) | `string` | `"23.*"` | no |
| enable\_docker | Install and enable docker daemon alongside HTCondor | `bool` | `true` | no |
| http\_proxy | Set system default web (http and https) proxy for Windows HTCondor installation | `string` | `""` | no |
| python\_windows\_installer\_url | URL of Python installer for Windows | `string` | `"https://www.python.org/ftp/python/3.11.9/python-3.11.9-amd64.exe"` | no |

## Outputs

| Name | Description |
|------|-------------|
| gcp\_service\_list | Google Cloud APIs required by HTCondor |
| runners | Runner to install HTCondor using startup-scripts |
| windows\_startup\_ps1 | Windows PowerShell script to install HTCondor |

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
