# Creating an external table from a {{ objstorage-full-name }} bucket table using a configuration file

To [create an external table](../../../managed-greenplum/operations/pxf/create-table.md) from a table in a {{ objstorage-full-name }} bucket, you need to provide a [static access key](../../../iam/concepts/authorization/access-key.md) for the service account in the query. You can do this using the [S3 protocol]({{ gp.docs.broadcom }}/7/greenplum-database/admin_guide-external-g-s3-protocol.html) and a configuration file stored on the HTTP server.

To create an external table using a configuration file:

1. [Set up your web server with a configuration file](#set-web-server).
1. [Create an external table](#create-ext-table).

If you no longer need the resources you created, [delete them](#clear-out).


## Required paid resources {#paid-resources}

The support cost includes:

* {{ mgp-name }} cluster fee: Using computing resources allocated to hosts and disk space (see [{{ GP }} pricing](../../../managed-greenplum/pricing/index.md)).
* Fee for a NAT gateway (see [{{ vpc-name }} pricing](../../../vpc/pricing.md)).
* Fee for an {{ objstorage-name }} bucket: Data storage and operations (see [{{ objstorage-name }} pricing](../../../storage/pricing.md)). 
* VM fee: Using its computing resources, storage, and, optionally, public IP address (see [{{ compute-name }} pricing](../../../compute/pricing.md)).
* Fee for using public IP addresses if public access is enabled for cluster hosts (see [{{ vpc-name }} pricing](../../../vpc/pricing.md)).


## Getting started {#before-you-begin}

Set up your infrastructure:

{% list tabs group=instructions %}

- Manually {#manual}

    1. [Create a {{ mgp-name }} cluster](../../../managed-greenplum/operations/cluster-create.md) with any suitable configuration.

    
    1. In the cluster subnet, [set up a NAT gateway](../../../vpc/operations/create-nat-gateway.md) and [create a security group](../../../vpc/operations/security-group-create.md) allowing all incoming and outgoing traffic from all addresses.


    
    1. [Create a virtual machine on Linux](../../../compute/operations/vm-create/create-linux-vm.md) in the same cloud network as the {{ mgp-name }} cluster.


    1. [Create a {{ objstorage-name }} bucket](../../../storage/operations/buckets/create.md) with restricted access. [Upload](../../../storage/operations/objects/upload.md) the `example.csv` file with a test table to it:

        ```csv
        10,2010
        ```

    
    1. [Create a static access key](../../../iam/operations/authentication/manage-access-keys.md#create-access-key) for the service account.


- {{ TF }} {#tf}

    1. {% include [terraform-install-without-setting](../../../_includes/mdb/terraform/install-without-setting.md) %}
    1. {% include [terraform-authentication](../../../_includes/mdb/terraform/authentication.md) %}
    1. {% include [terraform-setting](../../../_includes/mdb/terraform/setting.md) %}
    1. {% include [terraform-configure-provider](../../../_includes/mdb/terraform/configure-provider.md) %}

    1. In the working directory, create a file called `example.csv` with a test table:

        ```csv
        10,2010
        ```

    1. Download the [infrastructure configuration file](https://github.com/yandex-cloud-examples/yc-greenplum-config-server-for-s3/blob/main/greenplum-s3-vm.tf) to the same working directory.

        This file describes:

        * Network.
        * Subnet.
        * Service account with a static access key.
        * {{ mgp-name }} cluster.
        * Bucket to contain the `example.csv` file.
        * Virtual machine on [Ubuntu 20.04](/marketplace/products/yc/ubuntu-20-04-lts).

    1. Specify the following in the `greenplum-s3-vm.tf` file:

        * Password for the user named `user` for access to the {{ mgp-name }} cluster.
        * Virtual machine image ID.
        * Username and path to the SSH key for virtual machine access.
        * Folder ID for the same service account as specified in the provider settings.
        * Bucket name that must be unique throughout {{ objstorage-name }}.

    1. In the terminal window, go to the directory containing the infrastructure plan.

    1. To verify that the config files are correct, run the command below:

       ```bash
       terraform validate
       ```

       If there are any errors in the configuration files, {{ TF }} will point them out.

    1. Create the infrastructure required to follow the steps provided in this tutorial:

       {% include [terraform-apply](../../../_includes/mdb/terraform/apply.md) %}

       {% include [explore-resources](../../../_includes/mdb/terraform/explore-resources.md) %}

    1. To retrieve the static key parameters, run the command below in the working directory:

        ```bash
        terraform output -raw access_key > static-key.txt && \
        echo \n >> static-key.txt && \
        terraform output -raw secret_key >> static-key.txt
        ```

        The command saves to a file named `static-key.txt` the static key ID and the static key you are going to need later.

    
    1. Go to the [management console]({{ link-console-main }}) and [set up a NAT gateway](../../../vpc/operations/create-nat-gateway.md) for the subnet hosting your cluster.


{% endlist %}

## Set up your web server with a configuration file {#set-web-server}

Set up a web server on your virtual machine and create an `s3` configuration file on the web server:


1. [Connect](../../../compute/operations/vm-connect/ssh.md) to the virtual machine over SSH.


1. Create a folder named `/opt/gp_http_server` and put into it the `s3.config` file with the static key parameters and the other configuration settings for the `s3` protocol:

    ```bash
    sudo mkdir /opt/gp_http_server
    ```

    `s3.config` file contents:

    ```config
    [default]
    accessid = "<static_key_ID>"
    secret = "<static_key>"
    threadnum = 4
    chunksize = 67108864
    low_speed_limit = 10240
    low_speed_time = 60
    encryption = true
    version = 2
    proxy = ""
    autocompress = true
    verifycert = true
    server_side_encryption = ""
    # gpcheckcloud config
    gpcheckcloud_newline = "\n"
    ```

1. In the `/etc/systemd/system/` folder, put a web server config file named `gp_s3_config_http.service`:

    ```config
    [Unit]
    Description=HTTP server config to connect Greenplum with Object Storage
    After=network.target

    [Service]
    KillMode=process
    Restart=on-failure
    Type=simple
    User=root
    ExecStart=python3 -m http.server 8553 --directory /opt/gp_http_server

    [Install]
    WantedBy=multi-user.target
    ```

1. Start your web server:

    ```bash
    sudo systemctl daemon-reload && \
    sudo systemctl enable gp_s3_config_http && \
    sudo systemctl start gp_s3_config_http
    ```

1. To check the state of the web server, run the command below:

    ```bash
    sudo systemctl status gp_s3_config_http
    ```

## Create an external table {#create-ext-table}

1. [Connect to the {{ mgp-name }} cluster](../../../managed-greenplum/operations/connect.md).
1. Run a query to create an external table referencing the `example.csv` table in the bucket:

    ```sql
    CREATE READABLE EXTERNAL TABLE s3_ext_table(id int, year int)
    LOCATION('s3://{{ s3-storage-host }}/<bucket_name>/example.csv config_server=http://<VM_internal_IP_address>:8553/s3.config region={{ region-id }}-a')
    FORMAT 'csv';
    ```

    A successful query will return the following response:

    ```sql
    CREATE EXTERNAL TABLE
    ```

1. Query the external table for data:

    ```sql
    SELECT * FROM s3_ext_table;
    ```

    Expected result:

    ```text
     id | year
    ----+------
     10 | 2010
    (1 row)
    ```

## Delete the resources you created {#clear-out}

Some resources are not free of charge. To avoid paying for them, delete the resources you no longer need:

{% list tabs group=instructions %}

- Manually {#manual}

    
    1. [Delete the VM](../../../compute/operations/vm-control/vm-delete.md).
    1. If you reserved a public static IP address for the VM, [delete it](../../../vpc/operations/address-delete.md).
    1. [Delete the bucket in {{ objstorage-name }}](../../../storage/operations/buckets/delete.md).
    1. [Delete the {{ mgp-name }} cluster](../../../managed-greenplum/operations/cluster-delete.md).
    1. [Delete the service account](../../../iam/operations/sa/delete.md).
    1. [Delete the subnet](../../../vpc/operations/subnet-delete.md).
    1. [Delete the route table](../../../vpc/operations/delete-route-table.md).
    1. [Delete the NAT gateway](../../../vpc/operations/delete-nat-gateway.md).
    1. [Delete the cloud network](../../../vpc/operations/network-delete.md).


- {{ TF }} {#tf}

    {% include [terraform-clear-out](../../../_includes/mdb/terraform/clear-out.md) %}

{% endlist %}
