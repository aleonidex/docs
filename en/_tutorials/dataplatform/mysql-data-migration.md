# Migrating a database from a third-party {{ MY }} cluster to a {{ mmy-full-name }} cluster

There are two ways to migrate data from a third-party _source cluster_ to a {{ mmy-name }} _target cluster_:

* [Transferring data using {{ data-transfer-full-name }}](#data-transfer).

    This method is easy to configure, does not require the creation of an intermediate VM, and allows you to transfer the entire database without interrupting user service. To use it, allow connections to the source cluster from the internet.

    To learn more, see [{#T}](../../data-transfer/concepts/use-cases.md).

* [Transferring data by creating and restoring a logical dump](#logical-dump).

    A _logical dump_ is a file with a set of commands running which one by one you can restore the state of a database. To achieve a full logical dump, before you create it, switch the source cluster to <q>read-only</q>.

    Use this method only if, for some reason, it is not possible to migrate data using {{ data-transfer-name }}.


## Required paid resources {#paid-resources}

The cost of transferring data with {{ data-transfer-full-name }} includes:

* {{ mmy-name }} cluster fee: Using computing resources allocated to hosts and disk space (see [{{ MY }} pricing](../../managed-mysql/pricing.md)).
* Fee for using public IP addresses if public access is enabled for cluster hosts (see [{{ vpc-name }} pricing](../../vpc/pricing.md)).
* Transfer fee: Using computing resources and the number of transferred data rows (see [{{ data-transfer-name }} pricing](../../data-transfer/pricing.md)).

The cost of transferring data using a database dump includes:

* {{ mmy-name }} cluster fee: Using computing resources allocated to hosts and disk space (see [{{ MY }} pricing](../../managed-mysql/pricing.md)).
* Fee for using public IP addresses if public access is enabled for cluster hosts (see [{{ vpc-name }} pricing](../../vpc/pricing.md)).
* When creating a VM to download a dump: Fee for using computing resources, OS, and storage (see [{{ compute-name }} pricing](../../compute/pricing.md)).


## Transferring data using {{ data-transfer-name }} {#data-transfer}

{% include notitle [Migration with Data Transfer](../../_tutorials/dataplatform/datatransfer/managed-mysql.md) %}

## Transferring data by creating and restoring a logical dump {#logical-dump}

To move data to a {{ mmy-name }} cluster, create a logical dump of the desired database and restore it to the target cluster. There are two ways to do this:

* Use the `mydumper` and `myloader` [utilities](https://github.com/mydumper/mydumper). A database dump is created as a collection of files in a separate folder.
* Use `mysqldump` and `mysql`. A database dump is created as a single file.

Migration stages:

1. [Create a dump](#dump) of the database you want to migrate.
1. (Optional) [Upload a dump to an intermediate virtual machine](#vm-load) in {{ yandex-cloud }}.

    Transfer your data to an intermediate VM in {{ compute-full-name }} if:

    * Your {{ mmy-name }} cluster is not accessible from the internet.
    * Your hardware or connection to the cluster in {{ yandex-cloud }} is not very reliable.

    The larger the amount of data to be migrated and the required migration speed, the higher the virtual machine requirements: number of processor cores, RAM, and disk space.

1. [Restore data from the dump](#restore).

If you no longer need the resources you created, [delete them](#clear-out).

### Getting started {#before-you-begin}

Create the required resources:

{% list tabs group=instructions %}

- Manually {#manual}

    1. Create a [{{ mmy-name }} target cluster](../../managed-mysql/operations/cluster-create.md) in any suitable configuration. In this case, the following applies:

        * The {{ MY }} version must be the same or higher than the version in the source cluster.

            Transferring data with {{ MY }} major version upgrade is possible but not guaranteed. For more information, see the [{{ MY }} documentation](https://dev.mysql.com/doc/refman/8.0/en/faqs-migration.html).

            You [cannot](https://dev.mysql.com/doc/refman/8.0/en/downgrading.html) perform migration while downgrading {{ MY }} version.

        * [SQL mode](../../managed-mysql/concepts/settings-list.md#setting-sql-mode) must be the same as in the source cluster.

    1. (Optional step) [Create a VM](../../compute/operations/vm-create/create-linux-vm.md) based on [Ubuntu 20.04 LTS](/marketplace/products/yc/ubuntu-20-04-lts) with the following parameters:

        * **{{ ui-key.yacloud.compute.instances.create.section_storages }}** → **{{ ui-key.yacloud.compute.disk-form.field_size }}**: Sufficient to store both archived and unarchived dumps.

            The recommended size is two or three times the total dump and dump archive size.

        * **{{ ui-key.yacloud.compute.instances.create.section_network }}**:

            * **{{ ui-key.yacloud.component.compute.network-select.field_subnetwork }}**: Select a subnet on the cloud network hosting the target cluster.
            * **{{ ui-key.yacloud.component.compute.network-select.field_external }}**: Select `{{ ui-key.yacloud.component.compute.network-select.switch_auto }}` or one address from a list of reserved IPs.

    
    1. If you use security groups for the intermediate VM and the {{ mmy-name }} cluster, [configure them](../../managed-mysql/operations/connect.md#configure-security-groups).


- {{ TF }} {#tf}

    1. {% include [terraform-install-without-setting](../../_includes/mdb/terraform/install-without-setting.md) %}
    1. {% include [terraform-authentication](../../_includes/mdb/terraform/authentication.md) %}
    1. {% include [terraform-setting](../../_includes/mdb/terraform/setting.md) %}
    1. {% include [terraform-configure-provider](../../_includes/mdb/terraform/configure-provider.md) %}

    1. Download the [data-migration-mysql-mmy.tf](https://github.com/yandex-cloud-examples/yc-mysql-data-migration-from-on-premise/blob/main/data-migration-mysql-mmy.tf) configuration file to the same working directory.

        This file describes:

        * [Network](../../vpc/concepts/network.md#network).
        * [Subnet](../../vpc/concepts/network.md#subnet).
        * [Security group](../../vpc/concepts/security-groups.md) and the rule required to connect to a cluster.
        * {{ mmy-name }} cluster with public internet access.
        * (Optional) Virtual machine with public internet access.

    1. Specify the following in the `data-migration-mysql-mmy.tf` file:

        * Target cluster parameters:

            * `target_mysql_version`: {{ MY }} version. Must be the same or higher than in the source cluster.
            * `target_sql_mode`: [SQL mode](../../managed-mysql/concepts/settings-list.md#setting-sql-mode). It must be the same as in the source cluster.
            * `target_db_name`: Database name.
            * `target_user` and `target_password`: Name and user password of the database owner.

        * (Optional) Virtual machine parameters:

            * `vm_image_id`: ID of the public [image](../../compute/operations/images-with-pre-installed-software/get-list) with Ubuntu without GPU, e.g., for [Ubuntu 20.04 LTS](/marketplace/products/yc/ubuntu-20-04-lts).
            * `vm_username` and `vm_public_key`: Username and absolute path to the [public key](../../compute/operations/vm-connect/ssh.md#creating-ssh-keys), for access to the VM. By default, the specified username is ignored in the [Ubuntu 20.04 LTS](/marketplace/products/yc/ubuntu-20-04-lts) image. A user with the `ubuntu` username is created instead. Use it to connect to the VM.

    1. Make sure the {{ TF }} configuration files are correct using this command:

        ```bash
        terraform validate
        ```

        If there are any errors in the configuration files, {{ TF }} will point them out.

    1. Create the required infrastructure:

        {% include [terraform-apply](../../_includes/mdb/terraform/apply.md) %}

        {% include [explore-resources](../../_includes/mdb/terraform/explore-resources.md) %}

{% endlist %}

### Creating a dump {#dump}

{% list tabs %}

* Using the mysqldump utility

    1. Switch the database to <q>read-only</q> mode to avoid losing data that can appear while creating the dump.

    1. Install `mysqldump` in the source cluster, e.g., for Ubuntu:

        ```bash
        sudo apt update && sudo apt install mysql-client --yes
        ```

    1. Create a database dump:

        ```bash
        mysqldump \
            --host=<FQDN_or_IP_address> \
            --user=<username> \
            --password \
            --port=<port> \
            --set-gtid-purged=OFF \
            --quick \
            --single-transaction \
            <DB_name> > ~/db_dump.sql
        ```

        Where `--host` is the FQDN or IP address of the master host in the source cluster.

        If required, provide additional parameters in the create dump command:

        * `--events`, if there are recurring events in your database.
        * `--routines`, if your database stores procedures and functions.

        For InnoDB tables, use the `--single-transaction` option for data integrity.

    1. In the dump file, change the table engine names to `InnoDB`:

        ```bash
        sed -i -e 's/MyISAM/InnoDB/g' -e 's/MEMORY/InnoDB/g' db_dump.sql
        ```

    1. Archive the dump:

        ```bash
        tar -cvzf db_dump.tar.gz ~/db_dump.sql
        ```

* Using the mydumper utility

    1. Switch the database to <q>read-only</q> mode to avoid losing data that can appear while creating the dump.

    1. Create a directory for the dump files:

        ```bash
        mkdir db_dump
        ```

    1. Install `mydumper` in the source cluster, e.g., for Ubuntu:

        ```bash
        sudo apt update && sudo apt install mydumper --yes
        ```

    1. Create a database dump:

        ```bash
        mydumper \
            --triggers \
            --events \
            --routines \
            --outputdir=db_dump \
            --rows=10000000 \
            --threads=8 \
            --compress \
            --database=<DB_name> \
            --user=<username> \
            --ask-password \
            --host=<FQDN_or_IP_address>
        ```

        Where:

        * `--triggers`: Trigger dump.
        * `--events`: Event dump.
        * `--routines`: Stored procedure and function dump.
        * `--outputdir`: Dump file directory.
        * `--rows`: Number of rows in table fragments. The smaller the value, the more files in a dump.
        * `--threads`: Number of threads in use. The recommended value is equal to half the server's free cores.
        * `--compress`: Output file compression.
        * Where `--host` is the FQDN or IP address of the master host in the source cluster.

    1. In the dump file, change the table engine names to `InnoDB`:

        ```bash
        sed -i -e 's/MyISAM/InnoDB/g' -e 's/MEMORY/InnoDB/g' `find /db_dump -name '*-schema.sql'`
        ```

    1. Archive the dump:

        ```bash
        tar -cvzf db_dump.tar.gz ~/db_dump
        ```

{% endlist %}

### (Optional) Uploading a dump to a virtual machine in {{ yandex-cloud }} {#vm-load}

1. [Connect](../../compute/operations/vm-connect/ssh.md) to an intermediate virtual machine over SSH.

1. Copy the archive containing the database dump to the intermediate virtual machine, e.g., using `scp`:

    ```bash
    scp ~/db_dump.tar.gz <VM_username>@<VM_public_IP_address>:~/db_dump.tar.gz
    ```

1. Extract the dump from the archive:

    ```bash
    tar -xzf ~/db_dump.tar.gz
    ```

### Restoring data {#restore}

{% note alert %}

For {{ mmy-name }} clusters, [AUTOCOMMIT](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_autocommit) is enabled by default. **Do not disable** AUTOCOMMIT during the client session when restoring the database from the dump, otherwise the host storage may overflow and the cluster may not function properly.

{% endnote %}

{% list tabs %}

* Using the mysql utility

    This method is suitable if you used `mysqldump` to create the dump.

    1. Install the `mysql` utility to the host you are using to restore the dump, e.g., for Ubuntu:

        ```bash
        sudo apt update && sudo apt install mysql-client --yes
        ```

    1. Start the database restore from the dump:

        * If you restore a dump from the VM in {{ yandex-cloud }}:

            ```bash
            mysql \
                --host=c-<target_cluster_ID>.rw.{{ dns-zone }} \
                --user=<username> \
                --port={{ port-mmy }} \
                <DB_name> < ~/db_dump.sql
            ```

        * If you are restoring the dump from a host connecting to {{ yandex-cloud }} from the internet, [get an SSL certificate](../../managed-mysql/operations/connect.md#get-ssl-cert) and provide the `--ssl-ca` and the `--ssl-mode` parameters in the restore command:

            ```bash
            mysql \
                --host=c-<target_cluster_ID>.rw.{{ dns-zone }} \
                --user=<username> \
                --port={{ port-mmy }} \
                --ssl-ca=~/.mysql/root.crt \
                --ssl-mode=VERIFY_IDENTITY \
                <DB_name> < ~/db_dump.sql
            ```

* Using the myloader utility

    This method is suitable if you created the dump with `mydumper` and are using an intermediate virtual machine to restore it.

    1. Install the `myloader` utility to the host you are using to restore the dump, e.g., for Ubuntu:

        ```bash
        sudo apt update && sudo apt install mydumper --yes
        ```

    1. Start the database restore from the dump:

        ```bash
        myloader \
            --host=c-<target_cluster_ID>.rw.{{ dns-zone }} \
            --directory=db_dump/ \
            --overwrite-tables \
            --threads=8 \
            --compress-protocol \
            --user=<username> \
            --ask-password
        ```

{% endlist %}

You can get the cluster ID with the [list of clusters in the folder](../../managed-mysql/operations/cluster-list.md#list-clusters).

### Deleting the created resources {#clear-out}

Delete the resources you no longer need to avoid paying for them:

{% list tabs group=instructions %}

- Manually {#manual}

    * [Delete the {{ mmy-name }} cluster](../../managed-mysql/operations/cluster-delete.md).
    * If you created an intermediate virtual machine, [delete it](../../compute/operations/vm-control/vm-delete.md).
    * If you reserved public static IP addresses, release and [delete them](../../vpc/operations/address-delete.md).

- {{ TF }} {#tf}

    {% include [terraform-clear-out](../../_includes/mdb/terraform/clear-out.md) %}

{% endlist %}
