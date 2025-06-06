---
title: S3cmd
description: Из статьи вы узнаете, что такое S3cmd, как его установить и настроить, а также ознакомитесь с особенностями и примерами операций.
---

# S3cmd


[S3cmd](https://s3tools.org/s3cmd) — это консольный клиент (Linux, Mac) для сервисов, поддерживающими HTTP API Amazon S3. Общий порядок вызова команд смотрите в [официальной документации](https://s3tools.org/usage) S3cmd.

{% note info %}

{% include [aws-tools-prepare](../../_includes/aws-tools/s3cmd-versioning-info.md) %}

{% endnote %}

## Подготовка к работе {#before-you-begin}

{% include [aws-tools-prepare](../../_includes/aws-tools/aws-tools-prepare.md) %}

{% include [access-bucket-sa](../../_includes/storage/access-bucket-sa.md) %}

## Установка {#installation}

Чтобы установить S3cmd, ознакомьтесь с [разделом установки](https://github.com/s3tools/s3cmd/blob/master/INSTALL.md) в репозитории S3cmd на GitHub.

## Настройка {#setup}

Для настройки S3cmd используйте команду `s3cmd --configure`. При запросе укажите значения для следующих параметров:

* `Access Key` — введите идентификатор ключа, [полученный ранее](#before-you-begin);
* `Secret Key` — содержимое статического ключа, [полученное ранее](#before-you-begin);
* `Default Region` — введите `{{ region-id }}`;

   Для работы с {{ objstorage-name }} всегда указывайте регион `{{ region-id }}`. Другие значения региона могут привести к ошибке авторизации.

* `S3 Endpoint` — введите `{{ s3-storage-host }}`;
* `DNS-style bucket+hostname:port template for accessing a bucket` — введите `%(bucket)s.{{ s3-storage-host }}`;
* Значения остальных параметров оставьте без изменений;

Программа попытается установить соединение с {{ objstorage-name }} и получить список бакетов. В случае успеха, программа выведет `Success. Your access key and secret key worked fine :-)`.

Команда `s3cmd --configure` сохранит настройки в файле `~/.s3cfg` в формате:

```text
[default]
access_key = id
secret_key = secretKey
bucket_location = {{ region-id }}
host_base = {{ s3-storage-host }}
host_bucket = %(bucket)s.{{ s3-storage-host }}
```

При необходимости эти настройки можно изменить напрямую в файле. Также можно указать настройки при запуске программы с помощью соответствующих параметров.

Для корректной работы команд, управляющих хостингом статических сайтов, в конфигурационный файл необходимо вручную добавить параметр:

```text
website_endpoint = http://%(bucket)s.{{ s3-web-host }}
```

## Особенности {#specifics}

- S3cmd работает с {{ objstorage-name }} как с иерархической файловой системой и ключи объектов имеют вид пути к файлу.
- {% include [aws-tools-prepare](../../_includes/aws-tools/s3cmd-versioning-info.md) %}
- По умолчанию S3cmd загружает объекты в стандартное хранилище. Чтобы указать [класс хранилища](../concepts/storage-class.md), при загрузке объекта используйте параметр `--storage-class`.
- По умолчанию при загрузке объекта S3cmd может отправлять дополнительный заголовок `X-Amz-Meta-S3cmd-Attrs` с атрибутами вашего файла (права доступа, владельцы файла, временные метки). Значение заголовка сохраняется в [метаданных](../concepts/object.md#metadata) объекта. Отключить отправку атрибутов можно с помощью параметра `preserve_attrs = False` в конфигурационном файле `~/.s3cfg` или флага `--no-preserve`.

## Примеры операций {#s3cmd-examples}

{% note info %}

Для включения отладочного вывода в консоли используйте флаг `--debug`.

{% endnote %}

### Получить список бакетов {#listing-buckets}

{% include [sa-acl-bucket-list-note](../../_includes/storage/sa-acl-bucket-list-note.md) %}

```bash
s3cmd ls
```

### Создать бакет {#creating-bucket}

```bash
s3cmd mb s3://bucket
```

При создании бакета помните об [ограничениях на имя](../concepts/bucket.md#naming).

### Загрузить объект в холодное хранилище {#uploading-object}

```bash
s3cmd --storage-class COLD put local_file s3://bucket/object
```

### Получить список объектов {#getting-object-list}

```bash
s3cmd ls s3://bucket
```

### Получить объект {#retrieving-object}

```bash
s3cmd get s3://bucket/object local_file
```

### Удалить объект {#deleting-object}

```bash
s3cmd del s3://bucket/object
```

### Загрузка объекта с указанием MIME-типов {#mime-types}

Для указания [MIME-типов](https://ru.wikipedia.org/wiki/Список_MIME-типов) при загрузке объектов используйте команду `put` со следующими флагами:

```bash
s3cmd put \
  --no-guess-mime-type \
  --no-mime-magic \
  --mime-type="application/javascript" \
  <путь_к_локальному_файлу> \
  s3://<имя_бакета>/
```

Где `--mime-type` — MIME-тип вашего объекта.