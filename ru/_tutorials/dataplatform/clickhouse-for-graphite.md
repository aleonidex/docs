# Настройка {{ mch-name }} для Graphite

{{ mch-full-name }} можно использовать как хранилище данных для [Graphite](https://graphite.readthedocs.io/en/latest/index.html).

Движок таблиц [GraphiteMergeTree]({{ ch.docs }}/engines/table-engines/mergetree-family/graphitemergetree/) позволяет прореживать и агрегировать или усреднять содержимое БД специально для Graphite. Движок уменьшает объем хранения данных и повышает эффективность запросов от Graphite.

{% note info %}

Если прореживание и агрегирование или усреднение не требуется, то для хранения данных Graphite можно использовать любой [движок таблиц]({{ ch.docs }}/engines/table-engines/) {{ CH }}.

{% endnote %}

Чтобы настроить БД для работы с Graphite:
1. [Подготовьте окружение](#start).
1. [Создайте кластер](#cluster-create).
1. [Зарегистрируйте конфигурацию rollup в кластере](#rollup-config).
1. [Создайте и настройте виртуальную машину](#VM-setup).
1. [Подключите виртуальную машину к базе данных](#cluster-connect).
1. [Создайте таблицу на основе GraphiteMergeTree](#table-create).
1. [Настройте Graphite](#graphite-setup).

Если созданные ресурсы вам больше не нужны, [удалите их](#clear-out).

## Подготовьте окружение {#start}

{% include [before-you-begin](../_tutorials_includes/before-you-begin.md) %}


### Необходимые платные ресурсы {#paid-resources}

В стоимость поддержки БД {{ CH }} для Graphite входит:
* плата за вычислительные ресурсы кластера, объем хранилища и резервных копий (см. [тарифы {{ mch-full-name }}](../../managed-clickhouse/pricing.md));
* плата за запущенную ВМ для управления БД (см. [тарифы {{ compute-full-name }}](../../compute/pricing.md)).


## Создайте кластер {#cluster-create}

1. [Создайте кластер {{ mch-name }}](../../managed-clickhouse/operations/cluster-create.md) любой подходящей вам конфигурации с БД `db1` и публичным доступом ко всем его хостам. Сохраните имя БД, имя пользователя БД и пароль.
1. В [консоли управления]({{ link-console-main }}) выберите сервис {{ mch-name }}, перейдите в созданный кластер. На вкладке **{{ ui-key.yacloud.common.overview }}** сохраните идентификатор кластера. 
1. В правом верхнем углу нажмите **{{ ui-key.yacloud.mdb.clusters.button_action-connect }}**, на вкладке **Shell** из поля **Пример строки подключения** сохраните параметр `--host`, например `rc1a-2sqal8f0********.{{ dns-zone }}`, это FQDN хоста кластера, он потребуется в дальнейшем.

## Зарегистрируйте конфигурацию rollup в кластере {#rollup-config}

Зарегистрируйте в кластере конфигурацию `rollup` для прореживания и агрегирования или усреднения содержимого БД для Graphite:

{% list tabs group=instructions %}

- CLI {#cli}
  
  {% include [include](../../_includes/cli-install.md) %}

  {% include [default-catalogue](../../_includes/default-catalogue.md) %}

  1. Подготовьте yaml-файл `graphite-rollup.yaml` с описанием параметров `rollup`, например:
  
      ```yaml
      name: test_rollup
      patterns:
        - regexp: click_cost
          function: max
          retention:
            - age: 86400
              precision: 60
      ```

  1. Выполните команду:

      ```bash
      yc managed-clickhouse cluster add-graphite-rollup <идентификатор_кластера> --rollup-file-name <путь_к_yaml-файлу>
      ```

      Где:

      * `<идентификатор_кластера>` — идентификатор кластера.
      * `--rollup-file-name` — путь к `graphite-rollup.yaml`.

      Подробнее о команде `managed-clickhouse cluster add-graphite-rollup` см. в [справочнике CLI](../../cli/cli-ref/managed-clickhouse/cli-ref/cluster/add-graphite-rollup.md).

- API {#api}
  
  Используйте метод REST API [update](../../managed-clickhouse/api-ref/Cluster/update.md), передав в теле запроса требуемые параметры `rollup`:

    ```json
    "graphiteRollup": [
              {
                "name": "test_rollup",
                "patterns": [
                  {
                    "regexp": "click_cost",
                    "function": "max",
                    "retention": [
                      {
                        "age": "86400",
                        "precision": "60"
                      }
                    ]
                  }
                ]
              }
            ]
    ```

{% endlist %}

## Создайте и настройте виртуальную машину {#VM-setup}

1. В той же [облачной сети](../../vpc/concepts/network.md), где расположен кластер, [создайте](../../compute/operations/vm-create/create-linux-vm.md) ВМ на основе Linux. 
1. [Подключитесь](../../compute/operations/vm-connect/ssh.md) к ВМ по [SSH](../../glossary/ssh-keygen.md). 
1. Подключите [DEB-репозиторий]({{ ch.docs }}/getting-started/install/#install-from-deb-packages) {{ CH }}:

    ```bash
    sudo apt update && sudo apt install -y apt-transport-https ca-certificates dirmngr && \
    sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv E0C56BD4 && \
    echo "deb https://packages.{{ ch-domain }}/deb stable main" | sudo tee \
    /etc/apt/sources.list.d/clickhouse.list
    ```

1. Установите зависимости и клиентское приложение `clickhouse-client`:

    ```bash
    sudo apt update && sudo apt install -y clickhouse-client
    ```

1. Загрузите файл конфигурации для `clickhouse-client`:

    {% include [ClickHouse client config](../../_includes/mdb/mch/client-config.md) %}

1. Получите SSL-сертификат:

    ```bash
    sudo mkdir --parents {{ crt-local-dir }} && \
    sudo wget "{{ crt-web-path-root }}" \
         --output-document {{ crt-local-dir }}{{ crt-local-file-root }} && \
    sudo chmod 655 {{ crt-local-dir }}{{ crt-local-file-root }} && \
    sudo update-ca-certificates
    ```

## Подключите виртуальную машину к базе данных {#cluster-connect}

1. Если вы используете группы безопасности для облачной сети, [настройте их](../../managed-clickhouse/operations/connect/index.md#configuring-security-groups) так, чтобы был разрешен весь необходимый трафик между кластером и ВМ.

1. [Подключитесь](../../compute/operations/vm-connect/ssh.md). 
1. Запустите {{ CH }} CLI со следующими параметрами: вместо `<FQDN_хоста>`, `<имя_БД>`, `<имя_пользователя_БД>` и `<пароль_пользователя_БД>` укажите ранее сохраненные параметры.

    ```bash
    clickhouse-client --host <FQDN_хоста> \
                      --secure \
                      --user <имя_пользователя_БД> \
                      --database <имя_БД> \
                      --password <пароль_пользователя_БД> \
                      --port 9440
    ```

## Создайте таблицу на основе GraphiteMergeTree {#table-create}

{% list tabs group=instructions %}

- {{ CH }} CLI {#cli}

  В интерфейсе {{ CH }} CLI выполните запрос на создание таблицы на основе [GraphiteMergeTree]({{ ch.docs }}/engines/table-engines/mergetree-family/graphitemergetree/). В качестве параметра передайте имя секции `rollup`, описанной ранее:

    ```sql
    CREATE TABLE GraphiteTable
    (        
        Path String, 
        Time DateTime, 
        Value Int64, 
        Version UInt64
    )
    ENGINE = GraphiteMergeTree('test_rollup')
    PARTITION BY Time
    ORDER BY cityHash64(Version, Path)
    ```

{% endlist %}

## Настройте Graphite {#graphite-setup}

Настройте Graphite для сохранения значений метрик в кластере {{ CH }}. При этом прореживание данных будет проводиться автоматически средствами сервера {{ CH }} в соответствии с параметрами, которые вы указали.

1. В системе с Graphite установите утилиту `carbon-clickhouse`:

    ```bash
    wget https://github.com/go-graphite/carbon-clickhouse/releases/download/v0.11.2/carbon-clickhouse_0.11.2_amd64.deb && \
    sudo apt-get install $(pwd)/carbon-clickhouse_0.11.2_amd64.deb
    ```

1. [Настройте](https://github.com/go-graphite/carbon-clickhouse) `carbon-clickhouse`.
1. Запустите `carbon-clickhouse`:

    ```bash
    sudo systemctl enable carbon-clickhouse && \
    sudo systemctl start carbon-clickhouse
    ```

1. Установите утилиту `graphite-clickhouse`:

    ```bash
    wget https://github.com/go-graphite/graphite-clickhouse/releases/download/v0.13.2/graphite-clickhouse_0.13.2_amd64.deb && \
    sudo apt-get install $(pwd)/graphite-clickhouse_0.13.2_amd64.deb
    ```

1. [Настройте](https://github.com/go-graphite/graphite-clickhouse) `graphite-clickhouse`.
1. Запустите `graphite-clickhouse`:

    ```bash
    sudo systemctl enable graphite-clickhouse && \
    sudo systemctl start graphite-clickhouse
    ```

    Подробнее о настройке Graphite см. в [документации](https://graphite.readthedocs.io/en/latest/index.html). 

## Как удалить созданные ресурсы {#clear-out}

Чтобы удалить из кластера конфигурацию `rollup`:

1. Удалите все таблицы, которые используют эту конфигурацию.
1. Используйте команду `yc managed-clickhouse cluster remove-graphite-rollup`.

Подробнее о команде см. в [справочнике CLI](../../cli/cli-ref/managed-clickhouse/cli-ref/cluster/remove-graphite-rollup.md).

{% note alert %}

Удаление конфигурации `rollup` без предварительного удаления таблиц, которые ее используют, может привести к отказу кластера.

{% endnote %}

Удалите ресурсы, которые вы больше не будете использовать, чтобы за них не списывалась плата:

* [Удалите кластер {{ CH }}](../../managed-clickhouse/operations/cluster-delete.md).
* [Удалите ВМ](../../compute/operations/vm-control/vm-delete.md).
