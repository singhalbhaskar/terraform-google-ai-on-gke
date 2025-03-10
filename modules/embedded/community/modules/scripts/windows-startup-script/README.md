# terraform-google-ai-on-gke

## Description

This module contains a set of scripts to be used in customizing Windows VMs at
boot or during image building. Please note that the installation of NVIDIA GPU
drivers takes, at minimum, 30-60 minutes. It is therefore recommended to build
a custom image and reuse it as shown below, rather than install GPU drivers at
boot time.

> NOTE: the output `windows_startup_ps1` must be passed explicitly as shown
> below when used with Packer modules. This is due to a limitation in the `use`
> keyword and inputs of type `list` in Packer modules; this does not impact
> Terraform modules

### NVIDIA Drivers and CUDA Toolkit

Many Google Cloud VM families include or can have NVIDIA GPUs attached to them.
This module supports GPU applications by enabling you to easily install
a compatible release of NVIDIA drivers and of the CUDA Toolkit. The script is
the [solution recommended by our documentation][docs] and is [directly sourced
from GitHub][script-src].

[docs]: https://cloud.google.com/compute/docs/gpus/install-drivers-gpu#windows
[script-src]: https://github.com/GoogleCloudPlatform/compute-gpu-installation/blob/24dac3004360e0696c49560f2da2cd60fcb80107/windows/install_gpu_driver.ps1

```yaml
- group: primary
  modules:
  - id: network1
    source: modules/network/vpc
    settings:
      enable_iap_rdp_ingress: true
      enable_iap_winrm_ingress: true

  - id: windows_startup
    source: community/modules/scripts/windows-startup-script
    settings:
      install_nvidia_driver: true

- group: packer
  modules:
  - id: image
    source: modules/packer/custom-image
    kind: packer
    use:
    - network1
    - windows_startup
    settings:
      source_image_family: windows-2016
      machine_type: n1-standard-8
      accelerator_count: 1
      accelerator_type: nvidia-tesla-t4
      disk_size: 75
      disk_type: pd-ssd
      omit_external_ip: false
      state_timeout: 15m
```

## License

<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| http\_proxy | Set http and https proxy for use by Invoke-WebRequest commands | `string` | `""` | no |
| http\_proxy\_set\_environment | Set system default environment variables http\_proxy and https\_proxy for all commands | `bool` | `false` | no |
| install\_nvidia\_driver | Install NVIDIA GPU drivers and the CUDA Toolkit using script specified by var.install\_nvidia\_driver\_script | `bool` | `false` | no |
| install\_nvidia\_driver\_args | Arguments to supply to NVIDIA driver install script | `string` | `"/s /n"` | no |
| install\_nvidia\_driver\_script | Install script for NVIDIA drivers specified by http/https URL | `string` | `"https://developer.download.nvidia.com/compute/cuda/12.1.1/local_installers/cuda_12.1.1_531.14_windows.exe"` | no |
| no\_proxy | Environment variables no\_proxy (only used if var.http\_proxy\_set\_environment is enabled) | `string` | `"169.254.169.254,metadata,metadata.google.internal,.googleapis.com"` | no |

## Outputs

| Name | Description |
|------|-------------|
| windows\_startup\_ps1 | A string list of scripts selected by this module |

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
