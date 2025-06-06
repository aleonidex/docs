1. [Get your cloud ready](#before-begin).
1. [Create an infrastructure](#deploy).
1. [Configure Samba and NFS](#setup-samba-nfs).
1. [Test your file server](#test-file-server).

If you no longer need the resources you created, [delete them](#clear-out).

## Getting started {#before-you-begin}

{% include [before-you-begin](../_tutorials_includes/before-you-begin.md) %}

### Required paid resources {#paid-resources}

{% include [paid-resources](../_tutorials_includes/single-node-file-server/paid-resources.md) %}

## Create an infrastructure {#deploy}

{% include [terraform-definition](../_tutorials_includes/terraform-definition.md) %}

To create an infrastructure using {{ TF }}:
1. [Install {{ TF }}](../../tutorials/infrastructure-management/terraform-quickstart.md#install-terraform), [get the credentials](../../tutorials/infrastructure-management/terraform-quickstart.md#get-credentials), and specify the source for installing {{ yandex-cloud }} (see [{#T}](../../tutorials/infrastructure-management/terraform-quickstart.md#configure-provider), step 1).
1. Prepare the infrastructure description files:

   {% list tabs group=infrastructure_description %}

   - Ready-made archive {#ready}

     1. Create a directory for the files.
     1. Download the [archive](https://{{ s3-storage-host }}/doc-files/single-node-file-server.zip) (1 KB).
     1. Unpack the archive to the directory. This will add the `single-node-file-server.tf` configuration file to this directory.

   - Manually {#manual}

     1. Create a directory for the files.
     1. In the directory, create a configuration file named `single-node-file-server.tf`:

        {% cut "single-node-file-server.tf" %}

        {% include [single-node-file-server-tf-config](../../_includes/archive/single-node-file-server-tf-config.md) %}

        {% endcut %}

   {% endlist %}

   For more information about the properties of the {{ TF }} resources, see the Terraform official website:
   * [Network](../../vpc/concepts/network.md#network): [yandex_vpc_network]({{ tf-provider-resources-link }}/vpc_network)
   * [Subnets](../../vpc/concepts/network.md#subnet): [yandex_vpc_subnet]({{ tf-provider-resources-link }}/vpc_subnet)
   * [Security groups](../../vpc/concepts/security-groups.md): [yandex_vpc_security_group]({{ tf-provider-resources-link }}/vpc_security_group)
   * [VM image](../../compute/concepts/image.md): [yandex_compute_image]({{ tf-provider-resources-link }}/compute_image)
   * [Disk](../../compute/concepts/disk.md): [yandex_compute_disk]({{ tf-provider-resources-link }}/compute_disk)
   * [VM](../../compute/concepts/vm.md): [yandex_compute_instance]({{ tf-provider-resources-link }}/compute_instance)

1. Under `metadata`, enter your username and the SSH key contents. For more information, see [{#T}](../../compute/concepts/vm-metadata.md).
1. Create the resources:

   {% include [terraform-validate-plan-apply](../_tutorials_includes/terraform-validate-plan-apply.md) %}

After creating the infrastructure, [configure Samba and NFS](#setup-samba-nfs).

## Configure Samba and NFS {#setup-samba-nfs}

{% include [setup-samba-nfs](../_tutorials_includes/single-node-file-server/setup-samba-nfs.md) %}

## Test your file server {#test-file-server}

{% include [single-node-file-server](../_tutorials_includes/single-node-file-server/test-file-server.md) %}

## How to delete the resources you created {#clear-out}

To stop paying for the resources you created:

1. Open the `single-node-file-server.tf` configuration file and delete the new infrastructure description from it.
1. Apply the changes:

    {% include [terraform-validate-plan-apply](../_tutorials_includes/terraform-validate-plan-apply.md) %}