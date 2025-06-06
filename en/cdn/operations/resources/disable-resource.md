---
title: How to disable a resource in {{ cdn-full-name }}
description: Follow this guide to disable a resource.
---

# Disabling a resource

You can stop a resource by disabling end-user access to the content. To do this:

{% list tabs group=instructions %}

- Management console {#console}

  1. In the [management console]({{ link-console-main }}), select the folder where your resource is located.

  1. Select **{{ ui-key.yacloud.iam.folder.dashboard.label_cdn }}**.

  1. Click the resource name.

  1. In the top-right corner, click ![image](../../../_assets/console-icons/pencil.svg) **{{ ui-key.yacloud.common.edit }}**.

  1. Under **{{ ui-key.yacloud.cdn.label_section-additional }}**, disable **{{ ui-key.yacloud.cdn.label_access }}**.

  1. Click **{{ ui-key.yacloud.common.save }}**.

- CLI {#cli}

  {% include [include](../../../_includes/cli-install.md) %}

  {% include [default-catalogue](../../../_includes/default-catalogue.md) %}

  1. View the description of the CLI command to update a resource:
 
      ```bash
      yc cdn resource update --help
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
      created_at: "2022-01-19T09:23:57.921365Z"
      updated_at: "2022-01-19T10:55:30.305141Z"
      active: true
      options:
        edge_cache_settings:
          enabled: true
          default value: "345600"
        cache_http_headers:
          enabled: true
          value:
          - content-type
          - content-length
          - connection
          - server
          - date
          - test
        stale:
          enabled: true
          value:
          - error
          - updating
        allowed_http_methods:
          value:
          - GET
          - POST
          - HEAD
          - OPTIONS
      origin_group_id: "89783"
      origin_group_name: My origins group
      origin_protocol: HTTP
      ssl_certificate:
        type: DONT_USE
        status: READY
      ```

  1. Disable end-user access to content by setting the `--active` flag to `false`:

      ```bash
      yc cdn resource update <resource_ID> --active false
      ```

      For more information about the `yc cdn resource update` command, see the [CLI reference](../../../cli/cli-ref/cdn/cli-ref/resource/update.md).

- {{ TF }} {#tf}

  {% include [terraform-install](../../../_includes/terraform-install.md) %}

  1. Open the {{ TF }} configuration file and add or edit the `yandex_cdn_resource` section by setting `active = false`.

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

      For more information about the `yandex_cdn_resource` properties in {{ TF }}, see the [provider documentation]({{ tf-provider-resources-link }}/cdn_resource).

  1. In the command line, go to the directory with the {{ TF }} configuration file.

  1. Check the configuration using this command:
     ```
     terraform validate
     ```
     
     If the configuration is correct, you will get this message:

     ```
     Success! The configuration is valid.
     ```

  1. Run this command:
     ```
     terraform plan
     ```

     You will see a detailed list of resources. No changes will be made at this step. If the configuration contains any errors, {{ TF }} will point them out.

  1. Apply the changes:
     ```
     terraform apply
     ```

  1. Type `yes` and press **Enter** to confirm the changes.

     You can check the CDN resource update in the [management console]({{ link-console-main }}) or using this [CLI](../../../cli/quickstart.md) command:

     ```
     yc cdn resource list
     ```

- API {#api}

  Use the [update](../../api-ref/Resource/update.md) REST API method for the [Resource](../../api-ref/Resource/index.md) resource or the [ResourceService/Update](../../api-ref/grpc/Resource/update.md) gRPC API call.

{% endlist %}