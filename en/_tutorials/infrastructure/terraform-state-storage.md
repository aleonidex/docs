# Uploading {{ TF }} states to {{ objstorage-full-name }}


{% include [terraform-ref-intro](../../_includes/terraform-ref-intro.md) %}

This tutorial describes the steps for uploading a {{ TF }} state to [{{ objstorage-name }}](../../storage/).

A {{ TF }} state describes the current deployed infrastructure and is stored in `.tfstate` files. The state file is created after the infrastructure is deployed and can be immediately uploaded to {{ objstorage-name }}. The uploaded state file will be updated as the infrastructure you created changes.

In this example, the saved state allows other users to get the ID of one of the created [subnets](../../vpc/concepts/network.md#subnet) to connect a new [VM](../../compute/concepts/vm.md) to it.

To configure {{ TF }} state storage in {{ objstorage-name }} and use it to create new resources:
1. [Get your cloud ready](#before-you-begin).
1. [Required paid resources](#paid-resources).
1. [Install and configure {{ TF }}](#install-terraform).
1. [Configure your backend](#set-up-backend).
1. [Deploy your configuration](#deploy).
1. [Check the saved state](#check-condition).
1. [Retrieve the state from the backend](#retrieve-state).

If you no longer need the resources you created, [delete them](#clear-out).

## Get your cloud ready {#before-you-begin}

{% include [before-you-begin](../_tutorials_includes/before-you-begin.md) %}

### Required paid resources {#paid-resources}

The cost of infrastructure support for uploading {{ TF }} states to {{ objstorage-full-name }} includes data storage fees (see [{{ objstorage-name }} pricing](../../storage/pricing.md#prices-storage)).

Follow this tutorial to create three [VMs](../../compute/concepts/vm.md) with [public IP addresses](../../vpc/concepts/address.md#public-addresses), a virtual [network](../../vpc/concepts/network.md#network), and two [subnets](../../vpc/concepts/network.md#subnet) as a sample infrastructure deployed through {{ TF }}. The cost of supporting this infrastructure includes:
* Fee for [disks](../../compute/concepts/disk.md) and continuously running VMs (see [{{ compute-full-name }} pricing](../../compute/pricing.md)).
* Fee for a dynamic public IP address (see [{{ vpc-full-name }} pricing](../../vpc/pricing.md)).

## Create a service account and static access key {#create-service-account}

1. [Create a service account](../../iam/operations/sa/create.md) with the [editor](../../iam/roles-reference.md#editor) [role](../../iam/concepts/access-control/roles.md) for the [folder](../../resource-manager/concepts/resources-hierarchy.md#folder) specified in the provider settings.
1. [Get a static access key](../../iam/operations/authentication/manage-access-keys.md#create-access-key).Save the key ID and secret key: you will need them later as you follow this tutorial.

### Create a bucket {#create-service-account}

[Create a bucket](../../storage/operations/buckets/create.md) with restricted access. This bucket will store the {{ TF }} state file.

## Install and configure {{ TF }} {#prepare-terraform}

{% include notitle [terraform-prepare.md](../../_tutorials/infrastructure/terraform-prepare.md) %}

## Configure your backend {#set-up-backend}

{% note info %}

The following backend settings apply in {{ TF }} `1.6.3` and higher.

{% endnote %}

1. Add the key ID and secret key [you got earlier](#create-service-account) to environment variables:

   {% list tabs group=programming_language %}

   - Bash {#bash}

     ```bash
     export ACCESS_KEY="<key_ID>"
     export SECRET_KEY="<secret_key>"
     ```

   - PowerShell {#powershell}

     ```powershell
     $ACCESS_KEY="<key_ID>"
     $SECRET_KEY="<secret_key>"
     ```

   {% endlist %}

1. Add the provider and backend settings to the configuration file:

   ```hcl
   terraform {
     required_providers {
       yandex = {
         source = "yandex-cloud/yandex"
       }
     }

     backend "s3" {
       endpoints = {
         s3 = "https://{{ s3-storage-host }}"
       }
       bucket = "<bucket_name>"
       region = "{{ region-id }}"
       key    = "<path_to_state_file_in_bucket>/<state_file_name>.tfstate"

       skip_region_validation      = true
       skip_credentials_validation = true
       skip_requesting_account_id  = true # This option is required for {{ TF }} 1.6.1 or higher.
       skip_s3_checksum            = true # This option is required to describe a backend for {{ TF }} 1.6.3 or higher.

     }
   }

   provider "yandex" {
     zone      = "<default_availability_zone>"
   }
   ```

   To read more about the state storage backend, see the [{{ TF }} website](https://www.terraform.io/docs/backends/types/s3.html).
1. Run the following command in the folder with the configuration file:

   ```bash
   terraform init -backend-config="access_key=$ACCESS_KEY" -backend-config="secret_key=$SECRET_KEY"
   ```

{% include [deploy-infrastructure-step](../_tutorials_includes/deploy-infrastructure-step.md) %}

{% include [check-condition-step](../_tutorials_includes/check-condition-step.md) %}

## Retrieve the state from the backend {#retrieve-state}

You can request the {{ objstorage-name }} state saved in {{ TF }} from another configuration to expand the infrastructure you created.

Create another configuration and use the saved state to create another VM in one of the existing subnets:
1. Create a directory named `remote-state`.
1. Navigate to that directory and create the `remote-state.tf` configuration file:

   ```hcl
   terraform {
     required_providers {
       yandex = {
         source = "yandex-cloud/yandex"
       }
     }
   }

   provider "yandex" {
     zone      = "{{ region-id }}-d"
   }

   data "terraform_remote_state" "vpc" {
     backend = "s3"
     config  = {
       endpoints = {
         s3 = "https://{{ s3-storage-host }}"
       }
       bucket = "<bucket_name>"
       region = "{{ region-id }}"
       key    = "<path_to_state_file_in_bucket>/<state_file_name>.tfstate"

       skip_region_validation      = true
       skip_credentials_validation = true
       skip_requesting_account_id  = true # This option is required to describe a backend for Terraform versions higher than 1.6.1.

       access_key = "<key_ID>"
       secret_key = "<secret_key>"

      }
    }

   resource "yandex_compute_image" "ubuntu_2004" {
     source_family = "ubuntu-2004-lts"
   }

   resource "yandex_compute_disk" "boot-disk-vm3" {
     name     = "boot-disk-3"
     type     = "network-hdd"
     zone     = "{{ region-id }}-d"
     size     = "20"
     image_id = yandex_compute_image.ubuntu_2004.id
   }

   resource "yandex_compute_instance" "vm-3" {
     name = "terraform3"

     resources {
       cores  = 2
       memory = 2
     }

     boot_disk {
       disk_id = yandex_compute_disk.boot-disk-vm3.id
     }

     network_interface {
       subnet_id = data.terraform_remote_state.vpc.outputs.subnet-1
       nat       = true
     }

     metadata = {
       ssh-keys = "ubuntu:${file("<path_to_public_SSH_key>")}"
     }
   }
   ```

   Where:
   * `bucket`: Bucket name.
   * `key`: Object key in the bucket (name and path to the {{ TF }} state file in the bucket).
   * `access_key`: [Secret key ID](#create-service-account) of the [service account](../../iam/concepts/users/service-accounts.md) to access the bucket.
   * `secret_key`: Service account’s secret key value.
1. Run the `terraform init` command.
1. Run the `terraform plan` command. The terminal will display the plan for creating the VM.
1. Run the `terraform apply` command.
1. Navigate to the management console and make sure you can see the `terraform3` VM in the {{ compute-name }} section.

## Delete the resources you created {#clear-out}

To delete the resources you created, run the `terraform destroy` command; start with the second configuration, and then proceed to the first.

## See also {#see-also}

* [Getting started with {{ TF }}](../../tutorials/infrastructure-management/terraform-quickstart.md).
* [Locking {{ TF }} states using {{ ydb-name }}](../../tutorials/infrastructure-management/terraform-state-lock.md).
* [Using {{ yandex-cloud }} modules in {{ TF }}](../../tutorials/infrastructure-management/terraform-modules.md).
* [{{ TF }} data sources](../../tutorials/infrastructure-management/terraform-data-sources.md).
