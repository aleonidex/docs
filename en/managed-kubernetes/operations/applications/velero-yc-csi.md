---
title: Installing Velero
description: Follow this guide to install Velero.
---

# Installing Velero


[Velero](https://velero.io/) is a backup, recovery, and migration tool for [{{ managed-k8s-name }} cluster](../../concepts/index.md#kubernetes-cluster) objects, including [persistent volumes](../../concepts/volume.md#persistent-volume). With Velero, you can:
* Protect your data from loss using a flexible backup system.
* Recover a {{ managed-k8s-name }} cluster faster if it goes down.
* Move your data from one {{ managed-k8s-name }} cluster to another.

Velero uses the {{ CSI }} driver to [create backups](../../tutorials/kubernetes-backup.md) and restore persistent volumes from {{ yandex-cloud }} [disk snapshots](../../../compute/concepts/snapshot.md).

## Getting started {#before-you-begin}

1. {% include [cli-install](../../../_includes/cli-install.md) %}

   {% include [default-catalogue](../../../_includes/default-catalogue.md) %}

1. [Create a service account](../../../iam/operations/sa/create.md) needed to access [{{ objstorage-full-name }}](../../../storage/).

   ```bash
   yc iam service-account create --name <service_account_name>
   ```

1. [Assign](../../../iam/operations/sa/assign-role-for-sa.md) the `storage.editor` role to the service account:

   ```bash
   yc resource-manager folder add-access-binding <folder_ID> \
     --role storage.editor \
     --subject serviceAccount:<service_account_ID>
   ```

1. [Create a static access key](../../../iam/operations/authentication/manage-access-keys.md#create-access-key) for your [service account](../../../iam/concepts/users/service-accounts.md).

   * If installing Velero [in the management console using {{ marketplace-full-name }}](#marketplace-install), create a static key in JSON format and save it to the `sa-key.json` file:

     ```bash
     yc iam access-key create \
       --service-account-name=<service_account_name> \
       --format=json > sa-key.json
     ```

   * If installing Velero using a [Helm chart](#helm-install), run the following command and save the `key_id` and `secret` key you get:

     ```bash
     yc iam access-key create \
       --service-account-name=<service_account_name>
     ```

1. [Create an {{ objstorage-name }} bucket](../../../storage/operations/buckets/create.md).

1. {% include [check-sg-prerequsites](../../../_includes/managed-kubernetes/security-groups/check-sg-prerequsites-lvl3.md) %}

    {% include [sg-common-warning](../../../_includes/managed-kubernetes/security-groups/sg-common-warning.md) %}

1. Make sure you have enough disk snapshot and disk size quotas to create a backup. To do this, you can use the [service for viewing quotas](../../../quota-manager/operations/read-quotas.md).

## Installation using {{ marketplace-full-name }} {#marketplace-install}

1. Navigate to the [folder dashboard]({{ link-console-main }}) and select **{{ ui-key.yacloud.iam.folder.dashboard.label_managed-kubernetes }}**.
1. Click the name of the {{ managed-k8s-name }} cluster you need and select the ![image](../../../_assets/console-icons/shopping-cart.svg) **{{ ui-key.yacloud.k8s.cluster.switch_marketplace }}** tab.
1. Under **{{ ui-key.yacloud.marketplace-v2.label_available-products }}**, select [Velero](/marketplace/products/yc/velero-yc-csi) and click **{{ ui-key.yacloud.marketplace-v2.button_k8s-product-use }}**.
1. Configure the application:
   * **Namespace**: Create a [namespace](../../concepts/index.md#namespace) named `velero`. The application uses it by default. If you leave the default namespace, Velero may work incorrectly.

     {% note info %}

     If you create a namespace with a different name, you will need to specify it in every command using the `--namespace <Velero_application_namespace>` parameter.

     {% endnote %}

   * **Application name**: Specify the application name.
   * **{{ objstorage-name }} static access key**: Copy the contents of the `sa-key.json` file or create a new [access key](../../../iam/concepts/authorization/access-key.md) for the service account. The service account must have the `storage.editor` role.
   * **{{ objstorage-name }} bucket name**: Specify the name of the {{ objstorage-name }} bucket.
1. Click **{{ ui-key.yacloud.k8s.cluster.marketplace.button_install }}**.
1. Wait for the application to change its status to `Deployed`.

## Installation using a Helm chart {#helm-install}

1. {% include [Helm install](../../../_includes/managed-kubernetes/helm-install.md) %}
1. {% include [kubectl setup](../../../_includes/managed-kubernetes/kubectl-install.md) %}
1. To install a [Helm chart](https://helm.sh/docs/topics/charts/) with Velero, run the following command, specifying the parameters of the resources you created [earlier](#before-you-begin):

   ```bash
   helm pull oci://{{ mkt-k8s-key.yc_velero-yc-csi.helmChart.name }} \
        --version {{ mkt-k8s-key.yc_velero-yc-csi.helmChart.tag }} \
        --untar && \
   helm install \
        --namespace velero \
        --create-namespace \
        --set configuration.backupStorageLocation.bucket=<bucket_name> \
        --set serviceaccountawskeyvalue_generated.accessKeyID=<key_ID> \
        --set serviceaccountawskeyvalue_generated.secretAccessKey=<secret_key> \
        velero ./velero/
   ```

   {% include [Support OCI](../../../_includes/managed-kubernetes/note-helm-experimental-oci.md) %}

## See also {#see-also}

* [Velero documentation](https://velero.io/docs/v1.11/examples/)
* [{{ managed-k8s-name }} cluster backups in {{ objstorage-name }}](../../tutorials/kubernetes-backup.md)
