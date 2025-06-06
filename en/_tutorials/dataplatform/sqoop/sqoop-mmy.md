# Importing data from {{ mmy-full-name }} to {{ dataproc-full-name }} using Sqoop


{% include [What is the Sqoop](./header.md) %}


## Required paid resources {#paid-resources}

The support cost includes:

* {{ dataproc-name }} cluster fee (see [{{ dataproc-name }} pricing](../../../data-proc/pricing.md)).
* {{ mmy-name }} cluster fee: Using computing resources allocated to hosts and disk space (see [{{ MY }} pricing](../../../managed-mysql/pricing.md)).
* NAT gateway fee (see [{{ vpc-name }} pricing](../../../vpc/pricing.md)).
* {{ objstorage-name }} bucket fee: Storing data and performing operations with it (see [{{ objstorage-name }} pricing](../../../storage/pricing.md)).
* VM fee: Using computing resources, operating system, and storage (see [{{ compute-name }} pricing](../../../compute/pricing.md)).
* Fee for public IP addresses if public access is enabled for VMs or cluster hosts (see [{{ vpc-name }} pricing](../../../vpc/pricing.md)).


## Getting started {#before-you-begin}

{% include [Same Network](../../_tutorials_includes/note-same-network.md) %}

1. [Create a cloud network](../../../vpc/operations/network-create.md).
1. [Create a subnet](../../../vpc/operations/subnet-create.md) in the `{{ zone-id }}` availability zone.
1. [Set up a NAT gateway](../../../vpc/operations/create-nat-gateway.md) for the new subnet: this is a prerequisite for the {{ dataproc-name }} cluster.

You can create other resources manually or using {{ TF }}.

### Manually {#create-manual}

1. [Create a {{ mmy-name }} cluster](../../../managed-mysql/operations/cluster-create.md)of any suitable [configuration](../../../managed-mysql/concepts/instance-types.md) with the following settings:

    * **{{ ui-key.yacloud.mdb.forms.database_field_name }}**: `db1`
    * **{{ ui-key.yacloud.mdb.forms.database_field_user-login }}**: `user1`

1. To import data to an {{ objstorage-name }} bucket:

    1. [Create a bucket](../../../storage/operations/buckets/create.md) with restricted access.
    1. [Create a service account](../../../iam/operations/sa/create.md) with the following roles:

        * [dataproc.agent](../../../data-proc/security/index.md#dataproc-agent)
        * [dataproc.provisioner](../../../data-proc/security/index.md#dataproc-provisioner)
        * [{{ roles-monitoring-viewer }}](../../../monitoring/security/index.md#monitoring-viewer)
        * [storage.viewer](../../../storage/security/index.md#storage-viewer)
        * [storage.uploader](../../../storage/security/index.md#storage-uploader).

    1. [Grant this service account](../../../storage/operations/buckets/edit-acl.md) read and write permissions for this bucket.

1. [Create a {{ dataproc-name }} cluster](../../../data-proc/operations/cluster-create.md) in any suitable [configuration](../../../data-proc/concepts/instance-types.md).

    {% include [Settings for DataProc cluster](./data-processing-cluster-settings.md) %}

1. [Create a virtual machine](../../../compute/operations/vm-create/create-linux-vm.md) for connecting to {{ mmy-name }} and {{ dataproc-name }} clusters.

1. If you are using security groups for your clusters and VM instance, configure them to allow connecting:

    * [To the VM instance and {{ dataproc-name }} cluster](../../../data-proc/operations/connect.md).
    * [To the {{ mmy-name }} cluster](../../../managed-mysql/operations/connect.md#configure-security-groups).

### Using {{ TF }} {#create-terraform}

1. {% include [terraform-install-without-setting](../../../_includes/mdb/terraform/install-without-setting.md) %}
1. {% include [terraform-authentication](../../../_includes/mdb/terraform/authentication.md) %}
1. {% include [terraform-setting](../../../_includes/mdb/terraform/setting.md) %}
1. {% include [terraform-configure-provider](../../../_includes/mdb/terraform/configure-provider.md) %}

1. Download the [clusters-mysql-data-proc-and-vm.tf](https://github.com/yandex-cloud-examples/yc-data-proc-mysql-data-import/blob/main/clusters-mysql-data-proc-and-vm.tf) configuration file and save it to the same working directory.

    This file describes:

    * [Security groups](../../../vpc/concepts/security-groups.md) for the clusters and VM.
    * [Service account](../../../iam/concepts/users/service-accounts.md) for the {{ dataproc-name }} cluster.
    * [{{ objstorage-name }} bucket](../../../storage/concepts/bucket.md).
    * {{ mmy-name }} cluster.
    * {{ dataproc-name }} cluster.
    * Virtual machine with public internet access.

1. Specify the infrastructure settings in the `clusters-mysql-data-proc-and-vm.tf` configuration file under `locals`:

    * `folder_id`: [ID of the folder](../../../resource-manager/operations/folder/get-id.md) to create resources in.
    * `network_id`: ID of the cloud network you created earlier.
    * `subnet_id`: ID of the subnet you created earlier.
    * `storage_sa_id`: ID of the service account to use for creating a bucket in {{ objstorage-name }}.
    * `data_proc_sa`: Name of the {{ dataproc-name }} cluster service account. The name must be unique within the folder.
    * `my_cluster_version`: {{ MY }} version of the {{ mmy-name }} cluster.
    * `my_cluster_password`: Password for `user1` of the {{ mmy-name }} database named `db1`.
    * `vm_image_id`: ID of the public [image](../../../compute/operations/images-with-pre-installed-software/get-list) with Ubuntu without GPU, e.g., for [Ubuntu 20.04 LTS](/marketplace/products/yc/ubuntu-20-04-lts).
    * `vm_username` and `vm_public_key`: Username and absolute path to the [public SSH key](../../../compute/operations/vm-connect/ssh.md#creating-ssh-keys) to use for accessing the virtual machine. By default, the specified username is ignored in the [Ubuntu 20.04 LTS](/marketplace/products/yc/ubuntu-20-04-lts) image. A user with the `ubuntu` username is created instead. Use it to connect to the VM.
    * `bucket_name`: Bucket name in {{ objstorage-name }}. The name must be unique within the entire {{ objstorage-name }}.
    * `dp_public_key`: Absolute path to the [public SSH key](../../../data-proc/operations/connect.md#data-proc-ssh) for the {{ dataproc-name }} cluster.

        For an SSH connection to the hosts of a {{ dataproc-name }} cluster version 1.x , use the `root` username.

1. Make sure the {{ TF }} configuration files are correct using this command:

    ```bash
    terraform validate
    ```

    If there are any errors in the configuration files, {{ TF }} will point them out.

1. Create the required infrastructure:

    {% include [terraform-apply](../../../_includes/mdb/terraform/apply.md) %}

    {% include [explore-resources](../../../_includes/mdb/terraform/explore-resources.md) %}

{% endlist %}

## Preparing the source cluster {#prepare}

1. [Connect](../../../managed-mysql/operations/connect.md) to the `db1` database in the {{ mmy-full-name }} cluster as `user1`.
1. Add test data to the database. The example uses a simple table with people's names and ages:

    1. Create a table:

        ```sql
        CREATE TABLE persons (
            Name VARCHAR(30) NOT NULL,
            Age INTEGER DEFAULT 0,
            PRIMARY KEY (Name)
        );
        ```

    1. Populate the table with data:

        ```sql
        INSERT INTO persons (Name, Age) VALUES
            ('Anna', 19),
            ('Michael', 65),
            ('Fred', 28),
            ('Alsou', 50),
            ('Max', 27),
            ('John', 34),
            ('Dmitry', 42),
            ('Oleg', 19),
            ('Alina', 20),
            ('Maria', 28);
        ```

## Importing the database {#import}

To enable [database parallelism](https://sqoop.apache.org/docs/1.4.2/SqoopUserGuide.html#_controlling_parallelism), Sqoop allows you to split imported data not only by the primary key but also by other table columns. In this example, the data is split by the `age` column.

Let's assume that:

{% include [Shared settings](./shared-properties.md) %}
* {{ mmy-name }} cluster ID: `c9qgcd6lplrs********`

{% list tabs group=storage_system %}

- {{ objstorage-name }} {#storage}

    1. [Complete all prerequisite steps](../../../data-proc/operations/sqoop-usage.md#object-storage).
    1. Run this command:

        ```bash
        sqoop import "-Dorg.apache.sqoop.splitter.allow_text_splitter=true" \
            --connect "jdbc:mysql://c-c9qgcd6lplrs********.rw.{{ dns-zone }}:{{ port-mmy }}/db1" \
            --username "user1" \
            --P \
            --table "persons" \
            --target-dir "s3a://<bucket_name>/import-directory" \
            --split-by "age"
        ```

- HDFS directory {#hdfs}

    1. [Complete all prerequisite steps](../../../data-proc/operations/sqoop-usage.md#hdfs).
    1. Run this command:

        ```bash
        sqoop import "-Dorg.apache.sqoop.splitter.allow_text_splitter=true" \
            --connect "jdbc:mysql://c-c9qgcd6lplrs********.rw.{{ dns-zone }}:{{ port-mmy }}/db1" \
            --username "user1" \
            --table "persons" \
            --target-dir "import-directory" \
            --P \
            --split-by "age"
        ```

- Apache Hive {#hive}

    1. [Complete all prerequisite steps](../../../data-proc/operations/sqoop-usage.md#apache-hive).
    1. Run this command:

        ```bash
        sqoop import "-Dorg.apache.sqoop.splitter.allow_text_splitter=true" \
            --connect "jdbc:mysql://c-c9qgcd6lplrs********.rw.{{ dns-zone }}:{{ port-mmy }}/db1" \
            --username "user1" \
            --P \
            --table "persons" \
            --hive-import \
            --create-hive-table \
            --hive-database "db-hive" \
            --hive-table "import-table" \
            --split-by "age"
        ```

- Apache HBase {#hbase}

    1. [Complete all prerequisite steps](../../../data-proc/operations/sqoop-usage.md#apache-hbase).
    1. Run this command:

        ```bash
        sqoop import "-Dorg.apache.sqoop.splitter.allow_text_splitter=true" \
            --connect "jdbc:mysql://c-c9qgcd6lplrs********.rw.{{ dns-zone }}:{{ port-mmy }}/db1" \
            --username "user1" \
            --P \
            --table "persons" \
            --hbase-create-table \
            --column-family "family1" \
            --hbase-table "import-table" \
            --split-by "age"
        ```

{% endlist %}

## Verify the import {#check}

{% include [Check import](./check-import.md) %}

## Deleting the created resources {#clear-out}

Some resources are not free of charge. To avoid paying for them, delete the resources you no longer need:

{% list tabs group=instructions %}

- Manually {#manual}

    1. [Delete the VM](../../../compute/operations/vm-control/vm-delete.md).
    1. If you reserved a public static IP address for the virtual machine, release and [delete it](../../../vpc/operations/address-delete.md).
    1. Delete the clusters:

        * [{{ mmy-name }}](../../../managed-mysql/operations/cluster-delete.md).
        * [{{ dataproc-name }}](../../../data-proc/operations/cluster-delete.md).

    1. If you created a {{ objstorage-full-name }} bucket, [delete it](../../../storage/operations/buckets/delete.md).
    1. [Delete the subnet](../../../vpc/operations/subnet-delete.md).
    1. [Delete the route table](../../../vpc/operations/delete-route-table.md).
    1. [Delete the NAT gateway](../../../vpc/operations/delete-nat-gateway.md).
    1. [Delete the cloud network](../../../vpc/operations/network-delete.md).
    1. [Delete the service account](../../../iam/operations/sa/delete.md).

- {{ TF }} {#tf}

    To delete an infrastructure created with {{ TF }}:

    {% include [terraform-clear-out](../../../_includes/mdb/terraform/clear-out.md) %}

    Delete the resources you created manually:

    1. [Subnet](../../../vpc/operations/subnet-delete.md)
    1. [Route table](../../../vpc/operations/delete-route-table.md)
    1. [NAT gateway](../../../vpc/operations/delete-nat-gateway.md)
    1. [Cloud network](../../../vpc/operations/network-delete.md)

{% endlist %}
