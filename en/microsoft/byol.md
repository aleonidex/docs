# Using personal licenses for Microsoft products

{{ yandex-cloud }} allows you to use your own Microsoft product licenses on [dedicated hosts](../compute/concepts/dedicated-host.md). The licensing relationship exists solely between you as a client and Microsoft as a license vendor. You are fully responsible for complying with the Microsoft license [terms and conditions](https://www.microsoft.com/en-us/useterms). 

When using Microsoft license products in {{ yandex-cloud }}, the cost of license is deducted from the cost of VM usage. You need to settle all license-related matters with your supplier.

{% note info %}

Information in this article isn't legally binding and is provided for reference only.

{% endnote %}

## What licenses can I use {#which-licenses}

The possibility of using a license in {{ yandex-cloud }} depends on the license terms and conditions of a given Microsoft product. BYOL (Bring Your Own License) is possible if the license was purchased under one of the following agreements:

* Microsoft Enterprise Agreement (EA) or Microsoft Enterprise Subscription Agreement.
* Microsoft Server and Cloud Enrollment (SCE) Agreement.
* Microsoft Enterprise Subscription Agreement (EAS).
* Microsoft Products and Services Agreement (MPSA).
* Microsoft Open, Open Value, and Open Value Subscription.
* Microsoft Enrollment for Education Solutions (EES).

{% note info %}

Licenses purchased under Microsoft Cloud Solution Provider (CSP) don't fit for BYOL.

{% endnote %}

Note that the list of licenses may change and not be exhaustive. Make sure to contact Microsoft for the up-to-date list of license programs suitable for BYOL in cloud environments.

## How do I transfer licenses {#how-to-bring-licenses}

1. [Import the necessary image](#how-to-import).
1. [Create a group of dedicated hosts](#create-host-group).
1. [Create a virtual machine on the dedicated host from the imported image](#create-vm).
1. [Make sure the VM is up and running and activate your license](#test-vm).

### Import the necessary image {#how-to-import}

You can import the image using the [{{ compute-name }} REST API](../compute/api-ref/Image/create).

1. Execute the following query:

   {% list tabs group=programming_language %}

   - Bash {#bash}

     ```bash
     curl -H "Authorization: Bearer `yc iam create-token`" -H  "accept: application/json" -X POST https://compute.{{ api-host }}/compute/v1/images -d '{"folderId": "<ID of your folder>", "name": "<image name>", "description": "<image description>", "os": {"type": "WINDOWS"}, "pooled": false, "uri": "<link to image in Object Storage>"}'
     ```

   - PowerShell {#powershell}

     ```powershell
     function Create-YCImage {
       param(
         [ValidateNotNullOrEmpty()]
         [string]$folderId = "",

         [ValidateNotNullOrEmpty()]
         [string]$name = "",

         [string]$description = "",

         [ValidateNotNullOrEmpty()]
         [string]$os_type = "WINDOWS",

         [int64]$minDiskSizeGb = 50GB,

         [ValidateNotNullOrEmpty()]
         [string]$uri = ""
       )

       $body = @"
     {
       "folderId": "$folderId",
       "name": "$name",
       "description": "$description",
       "os.type": "$os_type",
       "minDiskSize": "$minDiskSizeGb",
       "os": {
         "type": "$os_type"
       },
       "uri": "$uri"
     }
     "@

       Invoke-WebRequest `
         -Method POST `
         -URI https://compute.{{ api-host }}/compute/v1/images `
         -header @{ "Authorization" = "Bearer $(& yc iam create-token)" } `
         -ContentType 'Application/json' `
         -body $body
     }


     $folderId = "<ID of your folder>"

     Create-YCImage `
       -folderId $folderId `
       -name "<image name>" `
       -uri "<link to image in Object Storage>"

     ```

   {% endlist %}

1. Open the [management console]({{ link-console-cloud }}) and select the folder whose ID you specified in the `folderId` parameter in the first step.
1. Go to {{ compute-name }} and choose the **Images** tab.
1. Find the image you are importing; it will have the `Creating` status. Wait until its status changes from `Creating` to `Ready`.

### Create a group of dedicated hosts {#create-host-group}

You can use your own licenses only on VMs created on dedicated hosts. Contact your account manager for a quota for a dedicated host group. Note that the minimum number of vCPU cores on dedicated hosts is 128, and they are all paid.

[Follow the guide](../compute/operations/dedicated-host/create-host-group.md) to create a group of dedicated hosts.

### Create a VM from your image on the dedicated host {#create-vm}

On the dedicated host, create a VM with the boot disk from the imported image. Specify the ID of the dedicated host in the `--host-id` parameter. You can only create a VM using the CLI, API, or Terraform. Run this command:

   {% list tabs group=instructions %}

   - CLI {#cli}

     ```bash
     yc compute instance create \
     --host-id <dedicated host ID> \
     --name 'win-test' \
     --folder-id <folder ID> \
     --cores <number of vCPUs> \
     --core-fraction 100 \
     --memory <amount of RAM in GB> \
     --network-interface subnet-id=<subnet ID>,nat-ip-version=ipv4 \
     --create-boot-disk image-id=<ID of imported image> \
     --zone <availability zone> \
     ```

   {% endlist %}

## Reset the admin password {#reset-password}

To access the VM, [reset the default administrator password](../compute/operations/vm-guest-agent/reset-password.md) and generate a new one.

## Test the VM {#test-vm}

Connect to the created VM [using RDP](../compute/operations/vm-connect/rdp.md) with the password generated after you reset the default one.

After that, you can activate your license.
