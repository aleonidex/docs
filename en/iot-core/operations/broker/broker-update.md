---
title: How to update a broker
description: Follow this guide to update a broker.
---

# Updating a broker

{% include [note-pp](../../../_includes/iot-core/note-pp.md) %}

You can change the [name](broker-update.md#update-name) and [description](broker-update.md#update-description) of a broker as well as [manage broker labels](broker-update.md#manage-label).

To access a broker, use its unique ID or name. For info on how to get the unique broker ID or name, see [{#T}](broker-list.md).

## Updating the name of a broker {#update-name}

{% list tabs group=instructions %}

- Management console {#console}

   1. In the [management console]({{ link-console-main }}), select the folder to update the broker name in.
   1. Select **{{ ui-key.yacloud.iam.folder.dashboard.label_iot-core }}**.
   1. In the left-hand panel, select **{{ ui-key.yacloud.iot.label_brokers }}**.
   1. To the right of the name of the broker you need, click ![image](../../../_assets/console-icons/ellipsis.svg) and select **{{ ui-key.yacloud.common.edit }}** from the drop-down list.
   1. Edit the **{{ ui-key.yacloud.common.name }}** field.
   1. Click **{{ ui-key.yacloud.common.save }}**.

- CLI {#cli}

  {% include [cli-install](../../../_includes/cli-install.md) %}
    
  {% include [default-catalogue](../../../_includes/default-catalogue.md) %}

  Update the name of a broker:

  ```bash
  yc iot broker update my-broker --new-name test-broker
  ```

  Result:
  ```text
  id: b91ki3851h**********
  folder_id: aoek49ghmk**********
  created_at: "2019-05-28T11:29:42.420Z"
  name: test-broker
  ```

- {{ TF }} {#tf}

  {% include [terraform-definition](../../../_tutorials/_tutorials_includes/terraform-definition.md) %}
  
  {% include [terraform-install](../../../_includes/terraform-install.md) %}

  To update the name of a broker created using {{ TF }}:

  1. Open the {{ TF }} configuration file and edit the `name` parameter value in the broker description fragment.

      Example broker description in the {{ TF }} configuration:

      ```hcl
      resource "yandex_iot_core_broker" "my_broker" {
        name        = "test-broker"
        description = "test broker for terraform provider documentation"
        }
      ...
      }
      ```

      For more information about the `yandex_iot_core_broker` resource parameters in {{ TF }}, see the [relevant provider documentation]({{ tf-provider-resources-link }}/iot_core_broker).
  1. In the command line, change to the folder where you edited the configuration file.
  1. Make sure the configuration file is correct using this command:

      ```bash
      terraform validate
      ```

      If the configuration is correct, you will get this message:
     
      ```bash
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

  1. Confirm the changes: type `yes` into the terminal and press **Enter**.

      You can check the updated broker name in the [management console]({{ link-console-main }}) or using this [CLI](../../../cli/quickstart.md) command:

      ```bash
      yc iot broker list
      ```

- API {#api}

  To rename a broker, use the [update](../../broker/api-ref/Broker/update.md) REST API method for the [Broker](../../broker/api-ref/Broker/index.md) resource or the [BrokerService/Update](../../broker/api-ref/grpc/Broker/update.md) gRPC API call.

{% endlist %}

## Updating the description of a broker {#update-description}

{% list tabs group=instructions %}

- Management console {#console}

   To update the broker description:

   1. In the [management console]({{ link-console-main }}), select the folder to update the broker description in.
   1. Select **{{ ui-key.yacloud.iam.folder.dashboard.label_iot-core }}**.
   1. In the left-hand panel, select **{{ ui-key.yacloud.iot.label_brokers }}**.
   1. To the right of the name of the broker you need, click ![image](../../../_assets/console-icons/ellipsis.svg) and select **{{ ui-key.yacloud.common.edit }}** from the drop-down list.
   1. Edit the **{{ ui-key.yacloud.common.description }}** field.
   1. Click **{{ ui-key.yacloud.common.save }}**.

- CLI {#cli}

  {% include [cli-install](../../../_includes/cli-install.md) %}

  Update the broker description:

  ```bash
  yc iot broker update my-broker --description "My test broker."
  ```

  Result:
  ```text
  id: b91ki3851h**********
  folder_id: aoek49ghmk**********
  created_at: "2019-05-28T11:29:42.420Z"
  name: my-broker
  description: My test broker.
  labels:
    test_label: my_broker_label
  ```

- {{ TF }} {#tf}

  {% include [terraform-definition](../../../_tutorials/_tutorials_includes/terraform-definition.md) %}
  
  {% include [terraform-install](../../../_includes/terraform-install.md) %}

  To update the description of a broker created using {{ TF }}:

  1. Open the {{ TF }} configuration file and edit the `description` parameter value in the broker description fragment.

      Example broker description in the {{ TF }} configuration:

      ```hcl
      resource "yandex_iot_core_broker" "my_broker" {
        name        = "test-broker"
        description = "test broker for terraform provider documentation"
        }
      ...
      }
      ```

      For more information about the `yandex_iot_core_broker` resource parameters in {{ TF }}, see the [relevant provider documentation]({{ tf-provider-resources-link }}/iot_core_broker).
  1. In the command line, change to the folder where you edited the configuration file.
  1. Make sure the configuration file is correct using this command:

      ```bash
      terraform validate
      ```

      If the configuration is correct, you will get this message:
     
      ```bash
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

      You can check the updated broker description in the [management console]({{ link-console-main }}) or using this [CLI](../../../cli/quickstart.md) command:

      ```bash
      yc iot broker get <broker_name>
      ```

- API {#api}

  To update a broker description, use the [update](../../broker/api-ref/Broker/update.md) REST API method for the [Broker](../../broker/api-ref/Broker/index.md) resource or the [BrokerService/Update](../../broker/api-ref/grpc/Broker/update.md) gRPC API call.

{% endlist %}

## Managing broker labels {#manage-label}

You can perform the following actions with broker labels:

* [Add](broker-update.md#add-label)
* [Edit](broker-update.md#update-label)
* [Delete](broker-update.md#remove-label)

### Adding a label {#add-label}

{% list tabs group=instructions %}

- Management console {#console}

   1. In the [management console]({{ link-console-main }}), select the folder to add the broker label to.
   1. Select **{{ ui-key.yacloud.iam.folder.dashboard.label_iot-core }}**.
   1. In the left-hand panel, select **{{ ui-key.yacloud.iot.label_brokers }}**.
   1. To the right of the name of the broker you need, click ![image](../../../_assets/console-icons/ellipsis.svg) and select **{{ ui-key.yacloud.common.edit }}** from the drop-down list.
   1. Fill in the **{{ ui-key.yacloud.component.key-values-input.label_key }}** and **{{ ui-key.yacloud.component.key-values-input.label_value }}** fields and click **{{ ui-key.yacloud.component.label-set.button_add-label }}**.
   1. Click **{{ ui-key.yacloud.common.save }}**.

- CLI {#cli}

  {% include [cli-install](../../../_includes/cli-install.md) %}
    
  {% include [default-catalogue](../../../_includes/default-catalogue.md) %}

  Add a label to a broker:

  ```bash
  yc iot broker add-labels my-broker --labels new_label=test_label
  ```

  Result:
  ```text
  id: b91ki3851h**********
  folder_id: aoek49ghmk**********
  created_at: "2019-05-28T11:29:42.420Z"
  name: my-broker
  labels:
    new_label: test_label
  ```

- {{ TF }} {#tf}

  {% include [terraform-definition](../../../_tutorials/_tutorials_includes/terraform-definition.md) %}
  
  {% include [terraform-install](../../../_includes/terraform-install.md) %}

  To add a label to a broker created using {{ TF }}:

  1. In the configuration file, describe the parameters of the resource to create:

     * `yandex_iot_core_broker`: Broker parameters:
       * `name`: Broker name.
       * `description`: Broker description.
       * `labels`: Broker labels in `key:value` format.

      Here is an example of the resource structure in the configuration file:

      ```hcl
      resource "yandex_iot_core_broker" "my_broker" {
        name        = "test-broker"
        description = "test broker for terraform provider documentation"
        labels = {
          new-label = "test-label"
        }
      ...
      }
      ```

      For more information about the `yandex_iot_core_broker` resource parameters in {{ TF }}, see the [relevant provider documentation]({{ tf-provider-resources-link }}/iot_core_broker).
  1. In the command line, change to the folder where you edited the configuration file.
  1. Make sure the configuration file is correct using this command:

      ```bash
      terraform validate
      ```

      If the configuration is correct, you will get this message:
     
      ```bash
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

      You can verify broker labels using the following [CLI](../../../cli/quickstart.md) command:

      ```bash
      yc iot broker get <broker_name>
      ```

- API {#api}

  To add a label to a broker, use the [update](../../broker/api-ref/Broker/update.md) REST API method for the [Broker](../../broker/api-ref/Broker/index.md) resource or the [BrokerService/Update](../../broker/api-ref/grpc/Broker/update.md) gRPC API call.

{% endlist %}

### Updating a label {#update-label}

{% list tabs group=instructions %}

- Management console {#console}

   1. In the [management console]({{ link-console-main }}), select the folder to update the broker label in.
   1. Select **{{ ui-key.yacloud.iam.folder.dashboard.label_iot-core }}**.
   1. In the left-hand panel, select **{{ ui-key.yacloud.iot.label_brokers }}**.
   1. To the right of the name of the broker you need, click ![image](../../../_assets/console-icons/ellipsis.svg) and select **{{ ui-key.yacloud.common.edit }}** from the drop-down list.
   1. Edit the **{{ ui-key.yacloud.component.key-values-input.label_key }}** and **{{ ui-key.yacloud.component.key-values-input.label_value }}** fields.
   1. Click **{{ ui-key.yacloud.common.save }}**.

- CLI {#cli}

  {% include [cli-install](../../../_includes/cli-install.md) %}
    
  {% include [default-catalogue](../../../_includes/default-catalogue.md) %}

  Update a broker label:

  {% include [labels-rewrite-warning](../../../_includes/labels-rewrite-warning.md) %}

  ```bash
  yc iot broker update my-broker --labels test_label=my_broker_label
  ```

  Result:
  ```text
  id: b91ki3851h**********
  folder_id: aoek49ghmk**********
  created_at: "2019-05-28T11:29:42.420Z"
  name: my-broker
  labels:
    test_label: my_broker_label
  ```

- {{ TF }} {#tf}

  {% include [terraform-definition](../../../_tutorials/_tutorials_includes/terraform-definition.md) %}
  
  {% include [terraform-install](../../../_includes/terraform-install.md) %}

  To update the label of a broker created using {{ TF }}:

  1. Open the {{ TF }} configuration file and edit the label value in the `labels` section, in the broker description fragment.

      Example broker description in the {{ TF }} configuration:

      ```hcl
      resource "yandex_iot_core_broker" "my_broker" {
        name        = "test-broker"
        description = "test broker for terraform provider documentation"
        labels = {
          test-label = "my-broker-label"
        }
      ...
      }
      ```

      For more information about the `yandex_iot_core_broker` resource parameters in {{ TF }}, see the [relevant provider documentation]({{ tf-provider-resources-link }}/iot_core_broker).
  1. In the command line, change to the folder where you edited the configuration file.
  1. Make sure the configuration file is correct using this command:

      ```bash
      terraform validate
      ```

      If the configuration is correct, you will get this message:
     
      ```bash
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

      You can verify broker labels using the following [CLI](../../../cli/quickstart.md) command:

      ```bash
      yc iot broker get <broker_name>
      ```

- API {#api}

  To update a broker label, use the [update](../../broker/api-ref/Broker/update.md) REST API method for the [Broker](../../broker/api-ref/Broker/index.md) resource or the [BrokerService/Update](../../broker/api-ref/grpc/Broker/update.md) gRPC API call.

{% endlist %}

### Deleting a label {#remove-label}

{% list tabs group=instructions %}

- Management console {#console}

   1. In the [management console]({{ link-console-main }}), select the folder to delete the broker label from.
   1. Select **{{ ui-key.yacloud.iam.folder.dashboard.label_iot-core }}**.
   1. In the left-hand panel, select **{{ ui-key.yacloud.iot.label_brokers }}**.
   1. To the right of the name of the broker you need, click ![image](../../../_assets/console-icons/ellipsis.svg) and select **{{ ui-key.yacloud.common.edit }}** from the drop-down list.
   1. To the right of the label to delete, click ![image](../../../_assets/console-icons/xmark.svg).
   1. Click **{{ ui-key.yacloud.common.save }}**.

- CLI {#cli}

  {% include [cli-install](../../../_includes/cli-install.md) %}
    
  {% include [default-catalogue](../../../_includes/default-catalogue.md) %}

  Delete a broker label:

  ```bash
  yc iot broker remove-labels my-broker --labels new_label
  ```

  Result:
  ```text
  id: b91ki3851h**********
  folder_id: aoek49ghmk**********
  created_at: "2019-05-28T11:29:42.420Z"
  name: my-broker
  ```

- {{ TF }} {#tf}

  {% include [terraform-definition](../../../_tutorials/_tutorials_includes/terraform-definition.md) %}
  
  {% include [terraform-install](../../../_includes/terraform-install.md) %}

  To delete the label of a broker created using {{ TF }}:

  1. Open the {{ TF }} configuration file and delete the label value in the `labels` section, in the broker description fragment. To delete all labels, delete the entire `labels` section.

      Example broker description in the {{ TF }} configuration:

      ```hcl
      resource "yandex_iot_core_broker" "my_broker" {
        name        = "test-broker"
        description = "test broker for terraform provider documentation"
        labels = {
          test-label = "my-broker-label"
        }
      ...
      }
      ```

      For more information about the `yandex_iot_core_broker` resource parameters in {{ TF }}, see the [relevant provider documentation]({{ tf-provider-resources-link }}/iot_core_broker).
  1. In the command line, change to the folder where you edited the configuration file.
  1. Make sure the configuration file is correct using this command:

      ```bash
      terraform validate
      ```

      If the configuration is correct, you will get this message:
     
      ```bash
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

      You can verify broker labels using the following [CLI](../../../cli/quickstart.md) command:

      ```bash
      yc iot broker get <broker_name>
      ```

- API {#api}

  To delete a broker label, use the [update](../../broker/api-ref/Broker/update.md) REST API method for the [Broker](../../broker/api-ref/Broker/index.md) resource or the [BrokerService/Update](../../broker/api-ref/grpc/Broker/update.md) gRPC API call.

{% endlist %}