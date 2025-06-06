

{% note info %}

{{ mkf-name }} has built-in support for certain connectors and allows you to manage them. For a list of available connectors, see [{#T}](../../managed-kafka/concepts/connectors.md). If you need other connectors or want to manage Kafka Connect manually, refer to this tutorial.

{% endnote %}

{{ KFC }} is a tool for streaming data between {{ KF }} and other data storages.

Data in {{ KFC }} is handled using _processes_ called workers. You can deploy the tool either in distributed mode with multiple workers or standalone mode with a single worker.

Data is moved using _connectors_ that are run in separate worker threads.

To learn more about Kafka Connect, see the documentation [{{ KF }}](https://kafka.apache.org/documentation/#connect).

Next, we will configure {{ KFC }} to interact with a {{ mkf-name }} cluster. The tool will be deployed on a [{{ yandex-cloud }} VM](../../compute/concepts/vm.md) as a separate installation. SSL encryption will be used to protect the connection.


We will also set up a simple [FileStreamSource](https://docs.confluent.io/home/connect/filestream_connector.html) connector. {{ KFC }} will use it to read data from a test JSON file and provide this data to a cluster topic.

{% note info %}

You can use any other {{ KFC }} connector to interact with {{ mkf-name }} clusters.

{% endnote %}

To configure {{ KFC }} to work with a {{ mkf-name }} cluster:

1. [Configure the VM](#prepare-vm).
1. [Prepare the test data](#prepare-test-data).
1. [Configure {{ KFC }}](#configure-kafka-connect).
1. [Run {{ KFC }} and test it](#test-kafka-connect).

If you no longer need the resources you created, [delete them](#clear-out).


## Required paid resources {#paid-resources}

The support cost includes:

* {{ mkf-name }} cluster fee: Using computing resources allocated to hosts (including {{ ZK }} hosts) and disk space (see [{{ KF }} pricing](../../managed-kafka/pricing.md)).
* Fee for using public IP addresses if public access is enabled for cluster hosts (see [{{ vpc-name }} pricing](../../vpc/pricing.md)).
* VM fee: using computing resources, storage, and public IP address (see [{{ compute-name }} pricing](../../compute/pricing.md)).


## Getting started {#before-you-begin}

{% list tabs group=instructions %}

- Manually {#manual}

    1. [Create a {{ mkf-name }} cluster](../../managed-kafka/operations/cluster-create.md) with any suitable configuration.
    1. [Create a topic](../../managed-kafka/operations/cluster-topics.md#create-topic) named `messages` for exchanging messages between {{ KFC }} and the {{ mkf-name }} cluster.
    1. [Create a user](../../managed-kafka/operations/cluster-accounts.md#create-account) named `user` and [grant them permissions](../../managed-kafka/operations/cluster-accounts.md#grant-permission) for the `messages` topic:

        * `ACCESS_ROLE_CONSUMER`
        * `ACCESS_ROLE_PRODUCER`

        
    1. In the network hosting the {{ mkf-name }} cluster, [create a virtual machine](../../compute/operations/vm-create/create-linux-vm.md) with [Ubuntu 20.04](/marketplace/products/yc/ubuntu-20-04-lts) and a public IP address.
    1. If you are using security groups, [configure them](../../managed-kafka/operations/connect/index.md#configure-security-groups) to allow all required traffic between the {{ mkf-name }} cluster and the VM.


- {{ TF }} {#tf}

    1. {% include [terraform-install-without-setting](../../_includes/mdb/terraform/install-without-setting.md) %}
    1. {% include [terraform-authentication](../../_includes/mdb/terraform/authentication.md) %}
    1. {% include [terraform-setting](../../_includes/mdb/terraform/setting.md) %}
    1. {% include [terraform-configure-provider](../../_includes/mdb/terraform/configure-provider.md) %}

    1. Download the [kafka-connect.tf](https://github.com/yandex-cloud-examples/yc-kafka-connect/blob/main/kafka-connect.tf) configuration file to the same working directory.

        This file describes:

        * Network.
        * Subnet.

        
        * Default security group and rules required to connect to the cluster and VM from the internet.


        * Virtual machine with [Ubuntu 20.04](/marketplace/products/yc/ubuntu-20-04-lts).
        * Properly configured {{ mkf-name }} cluster.

    1. In the file, specify the password for the user named `user` you are going to use to access the {{ mkf-name }} cluster, as well as the username and the public part of the SSH key for the virtual machine. If the virtual machine will be running Ubuntu 20.04 from the recommended [image list](../../compute/operations/images-with-pre-installed-software/get-list.md), the username you put here will be ignored. In which case use `ubuntu` for username to establish the [connection](#prepare-vm).
    1. Make sure the {{ TF }} configuration files are correct using this command:

       ```bash
       terraform validate
       ```

       If there are any errors in the configuration files, {{ TF }} will point them out.
    1. Create the required infrastructure:

       {% include [terraform-apply](../../_includes/mdb/terraform/apply.md) %}

       {% include [explore-resources](../../_includes/mdb/terraform/explore-resources.md) %}

{% endlist %}

## Configure the VM {#prepare-vm}


1. [Connect to the virtual machine over SSH](../../compute/operations/vm-connect/ssh.md).


1. Install JDK and the [kcat](https://docs.confluent.io/platform/current/app-development/kafkacat-usage.html) utility:

    ```bash
    sudo apt update && \
    sudo apt install default-jdk --yes && \
    sudo apt install kafkacat
    ```
    
    Check that you can use it to [connect to the {{ mkf-name }} source cluster over SSL](../../managed-kafka/operations/connect/clients.md#bash-zsh).

1. [Download](https://downloads.apache.org/kafka/) and unpack the archive containing {{ KF }}:

    ```bash
    wget https://downloads.apache.org/kafka/3.1.0/kafka_2.12-3.1.0.tgz && tar -xvf kafka_2.12-3.1.0.tgz --strip 1 --directory /opt/kafka/
    ```

    This example uses {{ KF }} version `3.1.0`.

1. [Get an SSL certificate](../../managed-kafka/operations/connect#get-ssl-cert).

1. {% include [keytool-importcert](../../_includes/mdb/keytool-importcert.md) %}

1. Create a folder with worker settings and copy the store there:

    ```bash
    sudo mkdir --parents /etc/kafka-connect-worker && \
    sudo cp ssl /etc/kafka-connect-worker/client.truststore.jks
    ```

## Prepare the test data {#prepare-test-data}

Create a file named `/var/log/sample.json` with test data. This file contains data from car sensors in JSON format:

{% cut "sample.json" %}

```json
{"device_id":"iv9a94th6rzt********","datetime":"2020-06-05 17:27:00","latitude":55.70329032,"longitude":37.65472196,"altitude":427.5,"speed":0,"battery_voltage":23.5,"cabin_temperature":17,"fuel_level":null}
{"device_id":"rhibbh3y08qm********","datetime":"2020-06-06 09:49:54","latitude":55.71294467,"longitude":37.66542005,"altitude":429.13,"speed":55.5,"battery_voltage":null,"cabin_temperature":18,"fuel_level":32}
{"device_id":"iv9a94th6rzt********","datetime":"2020-06-07 15:00:10","latitude":55.70985913,"longitude":37.62141918,"altitude":417,"speed":15.7,"battery_voltage":10.3,"cabin_temperature":17,"fuel_level":null}
```

{% endcut %}

## Configure {{ KFC }} {#configure-kafka-connect}

1. Create a file named `/etc/kafka-connect-worker/worker.properties` with worker settings:

    ```ini
    # AdminAPI connect properties
    bootstrap.servers=<broker_host_FQDN>:9091
    sasl.mechanism=SCRAM-SHA-512
    security.protocol=SASL_SSL
    ssl.truststore.location=/etc/kafka-connect-worker/client.truststore.jks
    ssl.truststore.password=<certificate_storage_password>
    sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required username="user" password="<user_password>";

    # Producer connect properties
    producer.sasl.mechanism=SCRAM-SHA-512
    producer.security.protocol=SASL_SSL
    producer.ssl.truststore.location=/etc/kafka-connect-worker/client.truststore.jks
    producer.ssl.truststore.password=<certificate_storage_password>
    producer.sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required username="user" password="<user_password>";

    # Worker properties
    plugin.path=/etc/kafka-connect-worker/plugins
    key.converter=org.apache.kafka.connect.json.JsonConverter
    value.converter=org.apache.kafka.connect.json.JsonConverter
    key.converter.schemas.enable=true
    value.converter.schemas.enable=true
    offset.storage.file.filename=/etc/kafka-connect-worker/worker.offset
    ```

    {{ KFC }} will connect to the {{ mkf-name }} cluster as the user named `user` [created earlier](#before-you-begin).

    You can request the FQDNs of broker hosts with the [list of cluster hosts](../../managed-kafka/operations/cluster-hosts.md).

1. Create a file named `/etc/kafka-connect-worker/file-connector.properties` with connector settings:

    ```ini
    name=local-file-source
    connector.class=FileStreamSource
    tasks.max=1
    file=/var/log/sample.json
    topic=messages
    ```

    Where:

    * `file`: Name of the file the connector will read data from.
    * `topic`: Name of the {{ mkf-name }} cluster topic the connector will feed data to.

## Run {{ KFC }} and test it {#test-kafka-connect}

1. To send test data to the cluster, run the worker on the VM:

    ```bash
    cd ~/opt/kafka/bin/ && \
    sudo ./connect-standalone.sh \
         /etc/kafka-connect-worker/worker.properties \
         /etc/kafka-connect-worker/file-connector.properties
    ```

1. Connect to the cluster [using kcat](../../managed-kafka/operations/connect/clients.md#bash-zsh) and retrieve data from the cluster topic:

    ```bash
    kafkacat -C \
        -b <broker_host_FQDN>:9091 \
        -t messages \
        -X security.protocol=SASL_SSL \
        -X sasl.mechanisms=SCRAM-SHA-512 \
        -X sasl.username=user \
        -X sasl.password="<user_account_password>" \
        -X ssl.ca.location={{ crt-local-dir }}{{ crt-local-file }} -Z -K:
    ```

    You can request the FQDNs of broker hosts with the [list of cluster hosts](../../managed-kafka/operations/cluster-hosts.md).

    In the command output, you will see the contents of the `/var/log/sample.json` test file provided in the previous step.

## Delete the resources you created {#clear-out}

Delete the resources you no longer need to avoid paying for them:

{% list tabs group=instructions %}

- Manually {#manual}

    
    1. [Delete the VM](../../compute/operations/vm-control/vm-delete.md).
    1. If you reserved a public static IP address for the VM, [delete it](../../vpc/operations/address-delete.md).
    1. [Delete the {{ mkf-name }} cluster](../../managed-kafka/operations/cluster-delete.md).


- {{ TF }} {#tf}

    {% include [terraform-clear-out](../../_includes/mdb/terraform/clear-out.md) %}

{% endlist %}
