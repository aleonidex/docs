# Импорт данных из {{ mpg-full-name }} в {{ dataproc-full-name }} с помощью Sqoop


{% include [What is the Sqoop](./header.md) %}


## Необходимые платные ресурсы {#paid-resources}

В стоимость поддержки описываемого решения входят:

* Плата за кластер {{ mpg-name }}: использование вычислительных ресурсов, выделенных хостам, и дискового пространства (см. [тарифы {{ mpg-name }}](../../../managed-postgresql/pricing.md)).
* Плата за использование публичных IP-адресов, если для хостов кластера включен публичный доступ (см. [тарифы {{ vpc-name }}](../../../vpc/pricing.md)).
* Плата за кластер {{ dataproc-name }}: использование вычислительных ресурсов ВМ и сетевых дисков {{ compute-name }}, а также сервиса {{ cloud-logging-name }} для работы с логами (см. [тарифы {{ dataproc-name }}](../../../data-proc/pricing.md)).
* Плата за NAT-шлюз (см. [тарифы {{ vpc-name }}](../../../vpc/pricing.md#nat-gateways)).
* Плата за бакет {{ objstorage-name }}: хранение данных и выполнение операций с ними (см. [тарифы {{ objstorage-name }}](../../../storage/pricing.md)). 
* Плата за ВМ: использование вычислительных ресурсов, операционной системы и хранилища (см. [тарифы {{ compute-name }}](../../../compute/pricing.md)).
* Плата за публичный IP-адрес для ВМ (см. [тарифы {{ vpc-name }}](../../../vpc/pricing.md)).


## Перед началом работы {#before-you-begin}

{% include [Same Network](../../_tutorials_includes/note-same-network.md) %}

1. [Создайте облачную сеть](../../../vpc/operations/network-create.md).
1. [Создайте подсеть](../../../vpc/operations/subnet-create.md) в зоне доступности `{{ zone-id }}`.
1. [Настройте NAT-шлюз](../../../vpc/operations/create-nat-gateway.md) для созданной подсети — это обязательное условие для работы кластера {{ dataproc-name }}.

Остальные ресурсы вы можете создать вручную или с помощью {{ TF }}.

### Вручную {#create-manual}

1. [Создайте кластер {{ mpg-name }}](../../../managed-postgresql/operations/cluster-create.md) любой подходящей вам [конфигурации](../../../managed-postgresql/concepts/instance-types.md) со следующими настройками:

    * **{{ ui-key.yacloud.mdb.forms.database_field_name }}** — `db1`;
    * **{{ ui-key.yacloud.mdb.forms.database_field_user-login }}** — `user1`.

1. Чтобы импортировать данные в бакет {{ objstorage-name }}:

    1. [Создайте бакет](../../../storage/operations/buckets/create.md) с ограниченным доступом.
    1. [Создайте сервисный аккаунт](../../../iam/operations/sa/create.md) с ролями:

        * [dataproc.agent](../../../data-proc/security/index.md#dataproc-agent);
        * [dataproc.provisioner](../../../data-proc/security/index.md#dataproc-provisioner);
        * [{{ roles-monitoring-viewer }}](../../../monitoring/security/index.md#monitoring-viewer);
        * [storage.viewer](../../../storage/security/index.md#storage-viewer);
        * [storage.uploader](../../../storage/security/index.md#storage-uploader).

    1. [Выдайте этому сервисному аккаунту](../../../storage/operations/buckets/edit-acl.md) разрешения на чтение и запись в бакет.

1. [Создайте кластер {{ dataproc-name }}](../../../data-proc/operations/cluster-create.md) любой подходящей вам [конфигурации](../../../data-proc/concepts/instance-types.md).

    {% include [Settings for DataProc cluster](./data-processing-cluster-settings.md) %}

1. [Создайте виртуальную машину](../../../compute/operations/vm-create/create-linux-vm.md) для подключения к кластерам {{ mpg-name }} и {{ dataproc-name }}.

1. Если вы используете группы безопасности для кластеров и виртуальной машины, настройте их так, чтобы разрешить подключение:

    * [к виртуальной машине и кластеру {{ dataproc-name }}](../../../data-proc/operations/connect.md);
    * [к кластеру {{ mpg-name }}](../../../managed-postgresql/operations/connect.md#configuring-security-groups).

### С помощью {{ TF }} {#create-terraform}

1. {% include [terraform-install-without-setting](../../../_includes/mdb/terraform/install-without-setting.md) %}
1. {% include [terraform-authentication](../../../_includes/mdb/terraform/authentication.md) %}
1. {% include [terraform-setting](../../../_includes/mdb/terraform/setting.md) %}
1. {% include [terraform-configure-provider](../../../_includes/mdb/terraform/configure-provider.md) %}

1. Скачайте в ту же рабочую директорию файл конфигурации [clusters-postgresql-data-proc-and-vm.tf](https://github.com/yandex-cloud-examples/yc-data-proc-postgresql-data-import/blob/main/clusters-postgresql-data-proc-and-vm.tf).

    В этом файле описаны:

    * [группы безопасности](../../../vpc/concepts/security-groups.md) для кластеров и виртуальной машины;
    * [сервисный аккаунт](../../../iam/concepts/users/service-accounts.md) для кластера {{ dataproc-name }};
    * [бакет {{ objstorage-name }}](../../../storage/concepts/bucket.md);
    * кластер {{ mpg-name }};
    * кластер {{ dataproc-name }};
    * виртуальная машина с публичным доступом из интернета.

1. Укажите параметры инфраструктуры в файле конфигурации `clusters-postgresql-data-proc-and-vm.tf` в блоке `locals`:

    * `folder_id` — [идентификатор каталога](../../../resource-manager/operations/folder/get-id.md), в котором будут созданы ресурсы.
    * `network_id` — идентификатор созданной ранее облачной сети.
    * `subnet_id` — идентификатор созданной ранее подсети.
    * `storage_sa_id` — идентификатор сервисного аккаунта, с помощью которого будет создан бакет в {{ objstorage-name }}.
    * `data_proc_sa` — имя сервисного аккаунта для кластера {{ dataproc-name }}. Оно должны быть уникальным в каталоге.
    * `pg_cluster_version` — версия {{ PG }} кластера {{ mpg-name }}.
    * `pg_cluster_password` — пароль для пользователя `user1` базы данных `db1` {{ mpg-name }}.
    * `vm_image_id` — идентификатор публичного [образа](../../../compute/operations/images-with-pre-installed-software/get-list) с Ubuntu без [GPU](../../../glossary/gpu.md). Например, для [Ubuntu 20.04 LTS](/marketplace/products/yc/ubuntu-20-04-lts).
    * `vm_username` и `vm_public_key` — логин и абсолютный путь к [публичному SSH-ключу](../../../compute/operations/vm-connect/ssh.md#creating-ssh-keys), которые будут использоваться для доступа к виртуальной машине. По умолчанию в образе [Ubuntu 20.04 LTS](/marketplace/products/yc/ubuntu-20-04-lts) указанный логин игнорируется, вместо него создается пользователь с логином `ubuntu`. Используйте его для подключения к виртуальной машине.
    * `bucket_name` — имя бакета в {{ objstorage-name }}. Оно должны быть уникальным для всего {{ objstorage-name }}.
    * `dp_public_key` — абсолютный путь к [публичному SSH-ключу](../../../data-proc/operations/connect.md#data-proc-ssh) для кластера {{ dataproc-name }}.

        Для [SSH-подключения](../../../glossary/ssh-keygen.md) к хостам кластера {{ dataproc-name }} версии 1.х используйте имя пользователя `root`.

1. Проверьте корректность файлов конфигурации {{ TF }} с помощью команды:

    ```bash
    terraform validate
    ```

    Если в файлах конфигурации есть ошибки, {{ TF }} на них укажет.

1. Создайте необходимую инфраструктуру:

    {% include [terraform-apply](../../../_includes/mdb/terraform/apply.md) %}

    {% include [explore-resources](../../../_includes/mdb/terraform/explore-resources.md) %}

{% endlist %}

## Подготовка кластера-источника {#prepare}

1. [Подключитесь к базе данных](../../../managed-postgresql/operations/connect.md) `db1` кластера {{ mpg-name }} от имени пользователя `user1`.
1. Наполните базу тестовыми данными. В качестве примера используется простая таблица с именами и возрастом людей:

    1. Создайте таблицу:

        ```sql
        CREATE TABLE persons (
            Name VARCHAR(30) NOT NULL,
            Age INTEGER DEFAULT 0,
            PRIMARY KEY (Name)
        );
        ```

    1. Наполните таблицу данными:

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

## Импорт базы данных {#import}

Чтобы обеспечить [параллелизм базы данных](https://sqoop.apache.org/docs/1.4.2/SqoopUserGuide.html#_controlling_parallelism), Sqoop позволяет разделять импортируемые данные не только по первичному ключу, но и по другим столбцам таблицы. В примере разделение данных выполняется по столбцу `age`.

Пусть:

{% include [Shared settings](./shared-properties.md) %}
* Идентификатор кластера {{ mpg-name }}: `c9qgcd6lplrs********`.

{% list tabs group=storage_system %}

- {{ objstorage-name }} {#storage}

    1. [Выполните все необходимые подготовительные шаги](../../../data-proc/operations/sqoop-usage.md#object-storage).
    1. Выполните команду:

        ```bash
        sqoop import "-Dorg.apache.sqoop.splitter.allow_text_splitter=true" \
            --connect "jdbc:postgresql://c-c9qgcd6lplrs********.rw.{{ dns-zone }}:{{ port-mpg }}/db1" \
            --username "user1" \
            --P \
            --table "persons" \
            --target-dir "s3a://<имя_бакета>/import-directory" \
            --split-by "age"
        ```

- Директория HDFS {#hdfs}

    1. [Выполните все необходимые подготовительные шаги](../../../data-proc/operations/sqoop-usage.md#hdfs).
    1. Выполните команду:

        ```bash
        sqoop import "-Dorg.apache.sqoop.splitter.allow_text_splitter=true" \
            --connect "jdbc:postgresql://c-c9qgcd6lplrs********.rw.{{ dns-zone }}:{{ port-mpg }}/db1" \
            --username "user1" \
            --table "persons" \
            --target-dir "import-directory" \
            --P \
            --split-by "age"
        ```

- Apache Hive {#hive}

    1. [Выполните все необходимые подготовительные шаги](../../../data-proc/operations/sqoop-usage.md#apache-hive).
    1. Выполните команду:

        ```bash
        sqoop import "-Dorg.apache.sqoop.splitter.allow_text_splitter=true" \
            --connect "jdbc:postgresql://c-c9qgcd6lplrs********.rw.{{ dns-zone }}:{{ port-mpg }}/db1" \
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

    1. [Выполните все необходимые подготовительные шаги](../../../data-proc/operations/sqoop-usage.md#apache-hbase).
    1. Выполните команду:

        ```bash
        sqoop import "-Dorg.apache.sqoop.splitter.allow_text_splitter=true" \
            --connect "jdbc:postgresql://c-c9qgcd6lplrs********.rw.{{ dns-zone }}:{{ port-mpg }}/db1" \
            --username "user1" \
            --P \
            --table "persons" \
            --hbase-create-table \
            --column-family "family1" \
            --hbase-table "import-table" \
            --split-by "age"
        ```

{% endlist %}

## Проверка корректности импорта {#check}

{% include [Check import](./check-import.md) %}

## Удаление созданных ресурсов {#clear-out}

Некоторые ресурсы платные. Чтобы за них не списывалась плата, удалите ресурсы, которые вы больше не будете использовать:

{% list tabs group=instructions %}

- Вручную {#manual}

    1. [Удалите виртуальную машину](../../../compute/operations/vm-control/vm-delete.md).
    1. Если вы зарезервировали для виртуальной машины публичный статический IP-адрес, освободите и [удалите его](../../../vpc/operations/address-delete.md).
    1. Удалите кластеры:

        * [{{ mpg-name }}](../../../managed-postgresql/operations/cluster-delete.md);
        * [{{ dataproc-name }}](../../../data-proc/operations/cluster-delete.md).

    1. Если вы создавали бакет {{ objstorage-name }}, [удалите его](../../../storage/operations/buckets/delete.md).
    1. [Удалите подсеть](../../../vpc/operations/subnet-delete.md).
    1. [Удалите таблицу маршрутизации](../../../vpc/operations/delete-route-table.md).
    1. [Удалите NAT-шлюз](../../../vpc/operations/delete-nat-gateway.md).
    1. [Удалите облачную сеть](../../../vpc/operations/network-delete.md).
    1. [Удалите сервисный аккаунт](../../../iam/operations/sa/delete.md).

- {{ TF }} {#tf}

    Чтобы удалить инфраструктуру, созданную с помощью {{ TF }}:

    {% include [terraform-clear-out](../../../_includes/mdb/terraform/clear-out.md) %}

    Удалите созданные вручную:

    1. [Подсеть](../../../vpc/operations/subnet-delete.md).
    1. [Таблицу маршрутизации](../../../vpc/operations/delete-route-table.md).
    1. [NAT-шлюз](../../../vpc/operations/delete-nat-gateway.md).
    1. [Облачную сеть](../../../vpc/operations/network-delete.md).

{% endlist %}
