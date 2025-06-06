

С помощью сервиса {{ data-transfer-name }} вы можете перенести данные из [потока {{ yds-name }}](../../data-streams/concepts/glossary.md#stream-concepts) в бакет {{ objstorage-full-name }}:

1. [Подготовьте поток данных {{ yds-name }}](#prepare-source).
1. [Подготовьте и активируйте трансфер](#prepare-transfer).
1. [Проверьте работоспособность трансфера](#verify-transfer).

Если созданные ресурсы вам больше не нужны, [удалите их](#clear-out).


## Необходимые платные ресурсы {#paid-resources}

В стоимость поддержки описываемого решения входят:

* Плата за базу данных {{ ydb-name }}. Тарификация зависит от режима использования:

	* Для бессерверного режима — оплачиваются операции с данными и объем хранимых данных.
	* Для режима с выделенными инстансами — оплачивается использование вычислительных ресурсов, выделенных БД, и дискового пространства.
	
    [Подробнее о тарифах {{ ydb-name }}](../../ydb/pricing/index.md).

* Плата за сервис {{ yds-name }}. Она зависит от режима тарификации:

	* По выделенным ресурсам — оплачивается количество единиц записываемых данных и ресурсы, выделенные для обслуживания потоков данных.
	* По фактическому использованию:
		* Если БД работает в бессерверном режиме, поток данных тарифицируется по [правилам тарификации для бессерверного режима {{ ydb-short-name }}](../../ydb/pricing/serverless.md).

		* Если БД работает в режиме с выделенными инстансами, поток данных отдельно не тарифицируется (оплачивается только БД, см. [правила тарификации](../../ydb/pricing/dedicated)).

    [Подробнее о тарифах {{ yds-name }}](../../data-streams/pricing.md).

* Плата за бакет {{ objstorage-name }}: хранение данных и выполнение операций с ними (см. [тарифы {{ objstorage-name }}](../../storage/pricing.md)).


## Перед началом работы {#before-you-begin}

Подготовьте инфраструктуру:

{% list tabs group=instructions %}

- Вручную {#manual}

    1. [Создайте базу данных {{ ydb-name }}](../../ydb/operations/manage-databases.md) любой подходящей конфигурации.

    1. [Создайте бакет в {{ objstorage-full-name }}](../../storage/operations/buckets/create.md).

- {{ TF }} {#tf}

    1. {% include [terraform-install-without-setting](../../_includes/mdb/terraform/install-without-setting.md) %}
    1. {% include [terraform-authentication](../../_includes/mdb/terraform/authentication.md) %}
    1. {% include [terraform-setting](../../_includes/mdb/terraform/setting.md) %}
    1. {% include [terraform-configure-provider](../../_includes/mdb/terraform/configure-provider.md) %}

    1. Скачайте в ту же рабочую директорию файл конфигурации [data-transfer-yds-obj.tf](https://github.com/yandex-cloud-examples/yc-data-transfer-from-yds-to-object-storage/blob/main/data-transfer-yds-obj.tf).

        В этом файле описаны:

        * база данных {{ ydb-name }};
        * сервисный аккаунт с ролями `yds.editor`, `storage.editor` и `storage.uploader`;
        * бакет в {{ objstorage-name }};
        * трансфер.

    1. Укажите в файле `data-transfer-yds-obj.tf`:

        * `folder_id` — [идентификатор каталога](../../resource-manager/operations/folder/get-id.md), в котором будут созданы ресурсы.
        * `sa_name` — имя сервисного аккаунта для создания бакета и использования в эндпоинтах.
        * `source_db_name` — имя базы данных {{ ydb-name }}.
        * `bucket_name` — имя бакета в {{ objstorage-name }}.

    1. Проверьте корректность файлов конфигурации {{ TF }} с помощью команды:

        ```bash
        terraform validate
        ```

        Если в файлах конфигурации есть ошибки, {{ TF }} на них укажет.

    1. Создайте необходимую инфраструктуру:

        {% include [terraform-apply](../../_includes/mdb/terraform/apply.md) %}

        {% include [explore-resources](../../_includes/mdb/terraform/explore-resources.md) %}

{% endlist %}

## Подготовьте поток данных {{ yds-name }} {#prepare-source}

1. [Создайте поток данных {{ yds-name }}](../../data-streams/operations/aws-cli/create.md).

1. [Отправьте в поток](../../data-streams/operations/aws-cli/send.md) тестовые данные. В качестве сообщения используйте данные от сенсоров автомобиля в формате JSON:

```json
{
    "device_id":"iv9a94th6rzt********",
    "datetime":"2020-06-05T17:27:00",
    "latitude":"55.70329032",
    "longitude":"37.65472196",
    "altitude":"427.5",
    "speed":"0",
    "battery_voltage":"23.5",
    "cabin_temperature":"17",
    "fuel_level":null
}
```

## Подготовьте и активируйте трансфер {#prepare-transfer}

1. [Создайте эндпоинт для источника](../../data-transfer/operations/endpoint/index.md#create):

    * **{{ ui-key.yacloud.data-transfer.forms.label-database_type }}** — `{{ yds-full-name }}`.
    * **Параметры эндпоинта**:

        * **{{ ui-key.yc-data-transfer.data-transfer.console.form.yds.console.form.yds.YDSSource.connection.title }}**:

            * **{{ ui-key.yc-data-transfer.data-transfer.console.form.yds.console.form.yds.YDSConnection.database.title }}** — выберите базу данных {{ ydb-name }} из списка.
            * **{{ ui-key.yc-data-transfer.data-transfer.console.form.yds.console.form.yds.YDSConnection.stream.title }}** — укажите имя потока {{ yds-name }}.
            * **{{ ui-key.yc-data-transfer.data-transfer.console.form.yds.console.form.yds.YDSConnection.service_account_id.title }}** — выберите или создайте сервисный аккаунт с ролью `yds.editor`.

        * **{{ ui-key.yc-data-transfer.data-transfer.console.form.yds.console.form.yds.YDSSource.advanced_settings.title }}**:

            * **{{ ui-key.yc-data-transfer.data-transfer.console.form.yds.console.form.yds.YDSSourceAdvancedSettings.converter.title }}** — `JSON`.
            * **{{ ui-key.yc-data-transfer.data-transfer.console.form.common.console.form.common.ConvertRecordOptions.data_schema.title }}** — `{{ ui-key.yc-data-transfer.data-transfer.console.form.common.console.form.common.DataSchema.json_fields.title }}`:

                Создайте и загрузите файл схемы данных в формате JSON `json_schema.json`:

                {% cut "json_schema.json" %}

                ```json
                [
                    {
                        "name": "device_id",
                        "type": "string"
                    },
                    {
                        "name": "datetime",
                        "type": "datetime"
                    },
                    {
                        "name": "latitude",
                        "type": "double"
                    },
                    {
                        "name": "longitude",
                        "type": "double"
                    },
                    {
                        "name": "altitude",
                        "type": "double"
                    },
                    {
                        "name": "speed",
                        "type": "double"
                    },
                    {
                        "name": "battery_voltage",
                        "type": "any"
                    },
                    {
                        "name": "cabin_temperature",
                        "type": "double"
                    },
                    {
                        "name": "fuel_level",
                        "type": "any"
                    }
                ]
                ```

                {% endcut %}

1. [Создайте эндпоинт для приемника](../../data-transfer/operations/endpoint/index.md#create):

    * **{{ ui-key.yacloud.data-transfer.forms.label-database_type }}** — `{{ objstorage-name }}`.
    * **{{ ui-key.yc-data-transfer.data-transfer.console.form.object_storage.console.form.object_storage.ObjectStorageEndpoint.title }}**:

        * **{{ ui-key.yc-data-transfer.data-transfer.console.form.object_storage.console.form.object_storage.ConnectionSettings.title }}**:

            * **{{ ui-key.yc-data-transfer.data-transfer.console.form.object_storage.console.form.object_storage.ConnectionSettings.bucket.title }}** — укажите имя бакета в {{ objstorage-name }}.
            * **{{ ui-key.yc-data-transfer.data-transfer.console.form.object_storage.console.form.object_storage.ObjectStorageConnectionSettings.service_account_id.title }}** — выберите или создайте сервисный аккаунт с ролью `storage.uploader`.

        * **{{ ui-key.yc-data-transfer.data-transfer.console.form.object_storage.console.form.object_storage.ObjectStorageTarget.output_format.title }}** — выберите `JSON` или `CSV`, если в расширенных настройках эндпоинта источника включили опцию **{{ ui-key.yc-data-transfer.data-transfer.console.form.yds.console.form.yds.YDSSourceAdvancedSettings.converter.title }}**.

1. Создайте трансфер:

    {% list tabs group=instructions %}

    - Вручную {#manual}

        1. [Создайте трансфер](../../data-transfer/operations/transfer.md#create) типа **{{ ui-key.yc-data-transfer.data-transfer.console.form.transfer.console.form.transfer.TransferType.increment.title }}**, использующий созданные эндпоинты.
        1. [Активируйте](../../data-transfer/operations/transfer.md#activate) его.

    - {{ TF }} {#tf}

        1. Раскомментируйте в файле `data-transfer-yds-obj.tf`:

            * переменные:

                * `source_endpoint_id` и задайте ей значение идентификатора эндпоинта для источника;
                * `target_endpoint_id` и задайте ей значение идентификатора эндпоинта для приемника;

            * ресурс `yandex_datatransfer_transfer`.

        1. Проверьте корректность файлов конфигурации {{ TF }} с помощью команды:

            ```bash
            terraform validate
            ```

            Если в файлах конфигурации есть ошибки, {{ TF }} на них укажет.

        1. Создайте необходимую инфраструктуру:

            {% include [terraform-apply](../../_includes/mdb/terraform/apply.md) %}

            Трансфер активируется автоматически после создания.

    {% endlist %}

## Проверьте работоспособность трансфера {#verify-transfer}

1. Дождитесь перехода трансфера в статус **{{ ui-key.yacloud.data-transfer.label_connector-status-RUNNING }}**.

1. Убедитесь, что данные из потока {{ yds-name }} перенеслись в бакет {{ objstorage-name }}:

    1. В [консоли управления]({{ link-console-main }}) выберите каталог, в котором находится бакет.
    1. В списке сервисов выберите **{{ ui-key.yacloud.iam.folder.dashboard.label_storage }}**.
    1. Выберите бакет из списка.
    1. Проверьте, что бакет содержит файл `<имя_потока>_0.raw` (`.json` или `.csv`, в зависимости от выбранного выходного формата) с тестовыми данными.

1. [Отправьте в поток {{ yds-name }}](../../data-streams/operations/aws-cli/send.md) новое сообщение:

    ```json
    {
        "device_id": "rhibbh3y08qm********",
        "datetime": "2020-06-06 09:49:54",
        "latitude": 55.71294467,
        "longitude": 37.66542005,
        "altitude": 429.13,
        "speed": 55.5,
        "battery_voltage": null,
        "cabin_temperature": 18,
        "fuel_level": 32
    }
    ```

1. Убедитесь, что в бакете {{ objstorage-name }} появились добавленные данные:

    1. В [консоли управления]({{ link-console-main }}) выберите каталог, в котором находится бакет.
    1. В списке сервисов выберите **{{ ui-key.yacloud.iam.folder.dashboard.label_storage }}**.
    1. Выберите бакет из списка.
    1. Проверьте, что в бакет добавился файл `<имя_потока>_0-1_1.raw` (`.json` или `.csv`, в зависимости от выбранного выходного формата) с новыми данными.

## Удалите созданные ресурсы {#clear-out}

{% note info %}

Перед тем как удалить созданные ресурсы, [деактивируйте трансфер](../../data-transfer/operations/transfer.md#deactivate).

{% endnote %}

Некоторые ресурсы платные. Чтобы за них не списывалась плата, удалите ресурсы, которые вы больше не будете использовать:

1. [Удалите трансфер](../../data-transfer/operations/transfer.md#delete).
1. [Удалите эндпоинты](../../data-transfer/operations/endpoint/index.md#delete) для источника и приемника.
1. [Удалите объекты](../../storage/operations/objects/delete.md) из бакета {{ objstorage-name }}.

Остальные ресурсы удалите в зависимости от способа их создания:

{% list tabs group=instructions %}

- Вручную {#manual}

    * [Удалите базу данных {{ ydb-name }}](../../ydb/operations/manage-databases.md#delete-db).
    * [Удалите бакет в {{ objstorage-name }}](../../storage/operations/buckets/delete.md).
    * Если при создании эндпоинтов вы создавали сервисные аккаунты, [удалите их](../../iam/operations/sa/delete.md).

- {{ TF }} {#tf}

    {% include [terraform-clear-out](../../_includes/mdb/terraform/clear-out.md) %}

{% endlist %}
