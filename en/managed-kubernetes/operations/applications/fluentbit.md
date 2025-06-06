---
title: Installing Fluent Bit with a plugin for {{ cloud-logging-full-name }}
description: Follow this tutorial to install Fluent Bit with a plugin for {{ cloud-logging-name }}.
---

# Installing Fluent Bit with a plugin for {{ cloud-logging-full-name }}


[Fluent Bit](https://www.fluentbit.io/) is an open-source multi-platform tool for collecting and processing logs. Fluent Bit collects, parses, and filters messages from a variety of inputs and saves them to storage. Messages then go to the router, which defines the output to send them to. Plugins are used to work with various inputs and outputs.

[The plugin for {{ cloud-logging-name }}](/marketplace/products/yc/fluent-bit) enables you to export logs generated in a {{ managed-k8s-name }} cluster, including at the [pod](../../concepts/index.md#pod) level, to [{{ cloud-logging-full-name }}](../../../logging/).

## Getting started {#before-you-begin}

1. {% include [cli-install](../../../_includes/cli-install.md) %}

   {% include [default-catalogue](../../../_includes/default-catalogue.md) %}

1. [Create a service account](../../../iam/operations/sa/create.md), which is required for Fluent Bit.
1. Assign the `logging.writer` and `monitoring.editor` [roles](../../../iam/operations/sa/assign-role-for-sa.md) to the service account.
1. [Create an authorized key](../../../iam/operations/authentication/manage-access-keys.md#create-access-key) for the service account and save it to the `sa-key.json` file.

1. {% include [check-sg-prerequsites](../../../_includes/managed-kubernetes/security-groups/check-sg-prerequsites-lvl3.md) %}

    {% include [sg-common-warning](../../../_includes/managed-kubernetes/security-groups/sg-common-warning.md) %}

## Installation using {{ marketplace-full-name }} {#marketplace-install}

1. Navigate to the [folder dashboard]({{ link-console-main }}) and select **{{ ui-key.yacloud.iam.folder.dashboard.label_managed-kubernetes }}**.
1. Click the name of the {{ managed-k8s-name }} cluster you need and select the ![image](../../../_assets/console-icons/shopping-cart.svg) **{{ ui-key.yacloud.k8s.cluster.switch_marketplace }}** tab.
1. Under **{{ ui-key.yacloud.marketplace-v2.label_available-products }}**, select [Fluent Bit with a plugin for {{ cloud-logging-name }}](/marketplace/products/yc/fluent-bit) and click **{{ ui-key.yacloud.marketplace-v2.button_k8s-product-use }}**.
1. Configure the application:
   * **Namespace**: Create a new [namespace](../../concepts/index.md#namespace), e.g., `fluent-bit-space`. If you leave the default namespace, Fluent Bit may work incorrectly.
   * **Application name**: Specify the application name.
   * **Target log group ID**: Specify the [ID of the log group](../../../logging/operations/list.md) to which Fluent Bit logs will be saved.
   * **Cluster ID**: Specify the [{{ managed-k8s-name }} cluster ID](../kubernetes-cluster/kubernetes-cluster-list.md).
   * **Service account key for {{ cloud-logging-name }}**: Paste the contents of the `sa-key.json` file.
1. Click **{{ ui-key.yacloud.k8s.cluster.marketplace.button_install }}**.
1. Wait for the application to change its status to `Deployed`.

## Installation using a Helm chart {#helm-install}

1. {% include [Install Helm](../../../_includes/managed-kubernetes/helm-install.md) %}
1. {% include [Install and configure kubectl](../../../_includes/managed-kubernetes/kubectl-install.md) %}
1. To install a [Helm chart](https://helm.sh/docs/topics/charts/) with Fluent Bit, run this command:

   ```bash
   cat sa-key.json | helm registry login {{ registry }} --username 'json_key' --password-stdin && \
   helm pull oci://{{ mkt-k8s-key.yc_fluent-bit.helmChart.name }} \
     --version {{ mkt-k8s-key.yc_fluent-bit.helmChart.tag }} \
     --untar && \
   helm install \
     --namespace <namespace> \
     --create-namespace \
     --set loggingGroupId=<log_group_ID> \
     --set loggingFilter=<cluster_ID> \
     --set-file auth.json=sa-key.json \
     fluentbit ./fluent-bit/
   ```

   This command also creates a new namespace required for Fluent Bit.

   If you set `namespace` to the default namespace, Fluent Bit may work incorrectly. We recommend you to specify a value different from all existing namespaces (e.g., `fluent-bit-space`).

   {% include [Support OCI](../../../_includes/managed-kubernetes/note-helm-experimental-oci.md) %}

## Use cases {#examples}

* [{#T}](../../tutorials/fluent-bit-logging.md).

## See also {#see-also}

* [Fluent Bit documentation](https://docs.fluentbit.io/manual)
* [{{ cloud-logging-name }} documentation](../../../logging/)
