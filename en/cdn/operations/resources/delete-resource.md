---
title: Deleting a resource
description: Follow this guide to delete a resource.
---

# Deleting a resource

{% list tabs group=instructions %}

- Management console {#console}

  1. In the [management console]({{ link-console-main }}), select the folder where your resource is located.
  1. Select **{{ ui-key.yacloud.iam.folder.dashboard.label_cdn }}**.
  1. Click the resource name.
  1. In the top-right corner, click ![image](../../../_assets/console-icons/trash-bin.svg) **{{ ui-key.yacloud.common.delete }}** and then click **{{ ui-key.yacloud.common.delete }}** in the window that opens.

- CLI {#cli}

  {% include [include](../../../_includes/cli-install.md) %}

  {% include [default-catalogue](../../../_includes/default-catalogue.md) %}

  1. View the description of the CLI command to delete a resource:

     ```bash
     yc cdn resource delete --help
     ```

  1. Get a list of all resources in the default folder:

     ```bash
     yc cdn resource list --format yaml
     ```

     Result:

     ```text
     id: s0me1dkfjq********
     folder_id: s0mef01der7p********
     cname: testexample.com
     ...
     ssl_certificate:
       type: DONT_USE
       status: READY
     ```

  1. Delete the resource:

     ```bash
     yc cdn resource delete <resource_ID>
     ```

     For more information about the `yc cdn resource delete` command, see the [CLI reference](../../../cli/cli-ref/cdn/cli-ref/resource/delete.md).
  1. Make sure that the delete operation was successful. To do this, view the folder's resource list again:

     ```bash
     yc cdn resource list
     ```

- {{ TF }} {#tf}

  {% include [terraform-install](../../../_includes/terraform-install.md) %}

  To delete a CDN resource created with {{ TF }}:
  1. Open the {{ TF }} configuration file and delete the resource description fragment.

     {% cut "An example of a CDN resource description in the {{ TF }} configuration" %}

     ```hcl
     resource "yandex_cdn_resource" "my_resource" {
         cname               = "cdn1.yandex-example.ru"
         active              = false
         origin_protocol     = "https"
         secondary_hostnames = ["cdn-example-1.yandex.ru", "cdn-example-2.yandex.ru"]
         origin_group_id     = yandex_cdn_origin_group.my_group.id
         ...
         options {
           edge_cache_settings = "345600"
           ignore_cookie       = true
           ...
         }
     }
     ```

     {% endcut %}

  1. In the command line, go to the directory with the {{ TF }} configuration file.
  1. Check the configuration using this command:

     ```bash
     terraform validate
     ```

     If the configuration is correct, you will get this message:

     ```text
     Success! The configuration is valid.
     ```

  1. Run this command:

     ```bash
     terraform plan
     ```

     You will see a detailed list of resources. No changes will be made at this step. If the configuration contains any errors, {{ TF }} will point them out.
  1. Apply the changes:

     ```bash
     terraform apply
     ```

  1. Type `yes` and press **Enter** to confirm the changes.

     You can check whether the CDN resource has been deleted using the [management console]({{ link-console-main }}) or this [CLI](../../../cli/quickstart.md) command:

     ```bash
     yc cdn resource list
     ```

- API {#api}

  To delete a resource, use the [delete](../../api-ref/Resource/delete.md) REST API method for the [Resource](../../api-ref/Resource/index.md) resource or the [ResourceService/Delete](../../api-ref/grpc/Resource/delete.md) gRPC API call.

{% endlist %}