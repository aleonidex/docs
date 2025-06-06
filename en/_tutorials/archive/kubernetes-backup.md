# {{ managed-k8s-name }} cluster backups in {{ objstorage-name }}


The {{ yandex-cloud }} infrastructure provides secure storage and replication for data in [{{ managed-k8s-name }} clusters](../../managed-kubernetes/concepts/index.md#kubernetes-cluster). However, you can back up data from [{{ managed-k8s-name }} cluster node groups](../../managed-kubernetes/concepts/index.md#node-group) at any time and store them in [{{ objstorage-full-name }}](../../storage/) or other types of storage.

You can create backups of {{ managed-k8s-name }} cluster node group data using [Velero](https://velero.io/). It supports {{ yandex-cloud }} [disks](../../compute/concepts/disk.md) through the {{ k8s }} CSI driver and helps create [disk](../../compute/concepts/snapshot.md) and [volume snapshots](../../managed-kubernetes/concepts/volume.md).

If installed manually, Velero allows you to use [nfs](https://kubernetes.io/docs/concepts/storage/volumes/#nfs), [emptyDir](https://kubernetes.io/docs/concepts/storage/volumes/#emptydir), [local](https://kubernetes.io/docs/concepts/storage/volumes/#local), or any other volume type without built-in support for snapshots. To use one of these volume types, install Velero with the [restic plugin](https://velero.io/docs/v1.8/restic/). Velero installed from [{{ marketplace-name }}](/marketplace/products/yc/velero-yc-csi) does not include the `restic` plugin.

In this article, you will learn how to create a backup of a {{ managed-k8s-name }} cluster node group using Velero, save it in {{ objstorage-name }}, and restore it in a different cluster’s node group:
1. [Create a backup of your {{ managed-k8s-name }} node group](#backup).
1. [Restore a node group of another {{ managed-k8s-name }} cluster from a backup](#restore).

If you no longer need the resources you created, [delete them](#clear-out).


## Required paid resources {#paid-resources}

The support cost includes:

* Fee for each {{ managed-k8s-name }} cluster: Using the master and outgoing traffic (see [{{ managed-k8s-name }} pricing](../../managed-kubernetes/pricing.md)).
* VM fee (two node groups and a VM for managing a cluster with no public access): Using computing resources, OS, and storage (see [{{ compute-name }} pricing](../../compute/pricing.md)).
* Fee for public IP addresses for the cluster nodes (see [{{ vpc-name }} pricing](../../vpc/pricing.md#prices-public-ip)).
* {{ objstorage-name }} bucket fee: Storing data and performing operations with it (see [{{ objstorage-name }} pricing](../../storage/pricing.md)).


## Getting started {#before-you-begin}

### Set up your infrastructure {#deploy-infrastructure}

{% list tabs group=instructions %}

- Manually {#manual}

  1. {% include [configure-sg-manual](../../_includes/managed-kubernetes/security-groups/configure-sg-manual-lvl3.md) %}

        You must configure these security groups for each cluster and node group.

        {% include [sg-common-warning](../../_includes/managed-kubernetes/security-groups/sg-common-warning.md) %}

  1. [Create two {{ managed-k8s-name }} clusters](../../managed-kubernetes/operations/kubernetes-cluster/kubernetes-cluster-create.md). When creating them, specify the security groups prepared earlier.

     One {{ managed-k8s-name }} cluster is for creating a node group backup and the other one, for restoring data from the backup.

     If you intend to use your clusters within the {{ yandex-cloud }} network, there is no need to allocate a public IP address to them. To allow connections from outside the network, assign a public IP to the clusters.

  1. [Create a node group](../../managed-kubernetes/operations/node-group/node-group-create.md) in each cluster. When creating node groups, allocate each group a public IP address and specify the security groups prepared earlier.
  1. [Create a bucket in {{ objstorage-name }}](../../storage/operations/buckets/create.md).
  1. [Create a service account](../../iam/operations/sa/create.md) with the `compute.admin` [role](../../iam/concepts/access-control/roles.md) for the [folder](../../resource-manager/concepts/resources-hierarchy.md#folder) to work with Velero.
  1. Grant the [service account](../../iam/concepts/users/service-accounts.md) **READ and WRITE** permissions to a [bucket](../../storage/concepts/bucket.md) in {{ objstorage-name }}. To do this, [configure the bucket ACL](../../storage/operations/buckets/edit-acl.md).
  1. [Create a static access key](../../iam/operations/authentication/manage-access-keys.md#create-access-key) for the service account and save its ID and value. You will not be able to get the secret key again.

- {{ TF }} {#tf}

  1. {% include [terraform-install-without-setting](../../_includes/mdb/terraform/install-without-setting.md) %}
  1. {% include [terraform-authentication](../../_includes/mdb/terraform/authentication.md) %}
  1. {% include [terraform-setting](../../_includes/mdb/terraform/setting.md) %}
  1. {% include [terraform-configure-provider](../../_includes/mdb/terraform/configure-provider.md) %}

  1. Download the [velero-backup.tf](https://github.com/yandex-cloud-examples/yc-mk8s-velero-backup/blob/main/velero-backup.tf) configuration file to the same working directory.

     This file describes:
     * [Network](../../vpc/concepts/network.md#network).
     * [Subnet](../../vpc/concepts/network.md#subnet).
     * Two {{ managed-k8s-name }} clusters and their node groups.
     * [Service account](../../iam/concepts/users/service-accounts.md) required to use {{ managed-k8s-name }} clusters and node groups.
     * {% include [configure-sg-terraform](../../_includes/managed-kubernetes/security-groups/configure-sg-tf-with-audience-lvl3.md) %}

        Both clusters will use these security groups.

        {% include [sg-common-warning](../../_includes/managed-kubernetes/security-groups/sg-common-warning.md) %}

     * Service account with the `compute.admin` [role](../../iam/concepts/access-control/roles.md) to work with Velero.
     * Static access key for the service account used to work with Velero.
     * [Bucket](../../storage/concepts/bucket.md) in {{ objstorage-name }}.
  1. In the `velero-backup.tf` file, specify the following:
     * `folder_id`: [ID of the folder](../../resource-manager/operations/folder/get-id.md) to create resources in.
     * `k8s_version`: [{{ k8s }} version](../../managed-kubernetes/concepts/release-channels-and-updates.md) 1.22 or higher.
     * `zone_a_v4_cidr_blocks`: CIDR of the subnet to host the {{ managed-k8s-name }} clusters.
     * `sa_name_k8s`: Name of the {{ managed-k8s-name }} cluster service account.
     * `sa_name_velero`: Name of the service account to work with Velero.
     * `storage_sa_id`: ID of the [service account](../../iam/concepts/users/service-accounts.md) with the `storage.admin` role. It will be used to create a bucket in {{ objstorage-name }} with `READ and WRITE` permissions in the [ACL](../../storage/concepts/acl.md) for the `sa_name_velero` service account.
     * `bucket_name`: Bucket name in {{ objstorage-name }}.
  1. Run the `terraform init` command in the directory with the configuration file. This command initializes the provider specified in the configuration files and enables you to use the provider resources and data sources.
  1. Check that the {{ TF }} configuration files are correct using this command:

     ```bash
     terraform validate
     ```

     If there are any errors in the configuration files, {{ TF }} will point them out.
  1. Create the required infrastructure:

     {% include [terraform-apply](../../_includes/mdb/terraform/apply.md) %}

     {% include [explore-resources](../../_includes/mdb/terraform/explore-resources.md) %}

  1. Get and save the ID and value of the static access key for the service account used to work with Velero.
     * Key ID:

       ```bash
       terraform output -raw access_key
       ```

     * Key value:

       ```bash
       terraform output -raw secret_key
       ```

{% endlist %}

### Configure additional settings {#additional-settings}

1. Select a [Velero client](https://github.com/vmware-tanzu/velero/releases) for your platform based on the [compatibility table](https://github.com/vmware-tanzu/velero#velero-compatibility-matrix).
1. Download the Velero client, extract the archive contents, and install it. For more information about installation, see [this Velero guide](https://velero.io/docs/main/basic-install/#install-the-cli).
1. Make sure you have installed the Velero client. To do this, run the following command:

   ```bash
   velero version
   ```

   Result:

   ```text
   Client:
           Version: v1.10.3
           Git commit: 18ee078dffd9345df610e0ca9f61b31124e93f50
   ```

## Backups {#backup}

To back up {{ managed-k8s-name }} node group data:
1. [Install kubectl]({{ k8s-docs }}/tasks/tools/install-kubectl) and [configure it to work with the first {{ managed-k8s-name }} cluster](../../managed-kubernetes/operations/connect/index.md#kubectl-connect).

   {% include [kubectl info](../../_includes/managed-kubernetes/kubectl-info.md) %}

1. {% include [install-velero](../../_includes/managed-kubernetes/install-velero.md) %}
1. Make sure you have enough disk snapshot quota and disk size to create a backup. To do this, you can use the [service for viewing quotas](../../quota-manager/operations/read-quotas.md).
1. Back up data from the {{ managed-k8s-name }} cluster node group:

   ```bash
   velero backup create my-backup
   ```

   Result:

   ```text
   Backup request "my-backup" submitted successfully.
   Run `velero backup describe my-backup` or `velero backup logs my-backup` for more details.
   ```

1. Wait for the backup to complete. You will see `Completed` under `STATUS`.

   ```bash
   velero backup get
   ```

   Result:

   ```text
   NAME       STATUS     ERRORS  WARNINGS  CREATED                        EXPIRES  STORAGE LOCATION  SELECTOR
   my-backup  Completed  0       0         2020-10-19 17:13:25 +0300 MSK  29d      default           <none>
   ```

## Restoring data from a backup {#restore}

To restore data from a {{ managed-k8s-name }} cluster node group:
1. [Configure kubectl](../../managed-kubernetes/operations/connect/index.md#kubectl-connect) to work with the second {{ managed-k8s-name }} cluster.

   {% include [kubectl info](../../_includes/managed-kubernetes/kubectl-info.md) %}

1. {% include [install-velero](../../_includes/managed-kubernetes/install-velero.md) %}
1. Make sure the data backup is there in the new {{ managed-k8s-name }} cluster:

   ```bash
   velero backup get
   ```

   Result:

   ```text
   NAME       STATUS     ERRORS  WARNINGS  CREATED                        EXPIRES  STORAGE LOCATION  SELECTOR
   my-backup  Completed  0       0         2020-10-19 17:13:25 +0300 MSK  29d      default           <none>
   ```

1. Restore data from the backup:

   ```bash
   velero restore create my-restore \
     --exclude-namespaces velero \
     --from-backup my-backup
   ```

   Where:

   * `--exclude-namespaces`: List of namespaces to exclude from the recovery process.
   * `--from-backup`: Name of the backup for recovery.

   Result:

   ```text
   Restore request "my-restore" submitted successfully.
   Run `velero restore describe my-restore` or `velero restore logs my-restore` for more details.
   ```

1. Wait for the backup restoration to complete. You will see `Completed` under `STATUS`.

   ```bash
   velero get restore
   ```

   Result:

   ```text
   NAME        BACKUP     STATUS     STARTED                        COMPLETED                      ERRORS  WARNINGS  CREATED                        SELECTOR
   my-restore  my-backup  Completed  2020-10-20 14:04:55 +0300 MSK  2020-10-20 14:05:22 +0300 MSK  0       23        2020-10-20 14:04:55 +0300 MSK  <none>
   ```

## Delete the resources you created {#clear-out}

If you no longer need the resources you created, delete them:

{% list tabs group=instructions %}

- Manually {#manual}

  * [Delete the {{ managed-k8s-name }} clusters](../../managed-kubernetes/operations/kubernetes-cluster/kubernetes-cluster-delete.md).
  * If you reserved [public static IP addresses](../../vpc/concepts/address.md#public-addresses) for the {{ managed-k8s-name }} clusters, [delete them](../../vpc/operations/address-delete.md).
  * [Delete the {{ objstorage-name }} bucket](../../storage/operations/buckets/delete.md).
  * [Delete the service account](../../iam/operations/sa/delete.md) used to work with Velero.

- {{ TF }} {#tf}

  {% include [terraform-clear-out](../../_includes/mdb/terraform/clear-out.md) %}

{% endlist %}
