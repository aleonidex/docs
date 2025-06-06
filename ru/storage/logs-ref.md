---
title: Справочник логов {{ objstorage-full-name }}
description: В данном разделе приведены описания поля логов, поставляемых бакетом, и методы запроса к бакету.
---

# Справочник логов {{ objstorage-name }}


В этом разделе описаны поля логов, поставляемых бакетом.

Логи {{ objstorage-name }} сохраняются в формате JSON. Одна запись в логах соответствует одному запросу к бакету.

Подробнее см. в разделе [{#T}](concepts/server-logs.md).

## Поля логов {#log-fields}

Поле | Тип | Описание
--- | --- | ---
`bucket` | String | Имя бакета.
`bytes_received` | Int64 | Размер запроса в байтах.
`bytes_send` | Int64 | Размер ответа в байтах.
`handler` | String | [Метод запроса](#bucket-methods) в формате `REST.<HTTP-метод>.<субъект>`.
`http_referer` | String | URL-адрес источника запроса.
`ip` | String | IP-адрес пользователя.
`method` | String | Метод HTTP-запроса.
`object_key` | String | [Ключ](concepts/object.md#key) объекта, закодированный методом [URL-кодировки](https://en.wikipedia.org/wiki/Percent-encoding).
`protocol` | String | Версия протокола передачи данных.
`range` | String | HTTP-заголовок, который определяет диапазон байт для загрузки из объекта.
`requester` | String | Идентификатор пользователя.
`request_args` | String | Аргументы URL-запроса.
`request_id` | String | Идентификатор запроса.
`request_path` | String | Полный путь запроса.
`request_time` | Int64 | Время обработки запроса, в миллисекундах.
`scheme` | String | Тип протокола передачи данных.<br>Возможные значения:<br>- `http` — протокол прикладного уровня передачи данных.<br>- `https` — протокол прикладного уровня передачи данных с поддержкой шифрования.
`ssl_protocol` | String | Протокол обеспечения безопасности.
`status` | Int64 | HTTP-код [ответа](s3/api-ref/response-codes.md).
`storage_class` | String | [Класс хранилища](concepts/storage-class.md) объекта.
`timestamp` | String | Дата и время операции с бакетом, в формате `ГГГГ-ММ-ДДTЧЧ:ММ:ССZ`.
`user_agent` | String | Клиентское приложение (User Agent), которое выполнило запрос.
`version_id` | String | Версия объекта.
`vhost` | String | Виртуальный хост запроса.<br>Возможные значения:<br>– `{{ s3-storage-host }}`.<br>– `<имя_бакета>.{{ s3-storage-host }}`.<br>– `{{ s3-web-host }}`.<br> – `<имя_бакета>.{{ s3-web-host }}`.

## Методы запроса к бакету {#bucket-methods}

| HTTP-метод | Субъект | Описание |
|---|---|---|
| `GET` | `SERVICE` | Вывод списка доступных бакетов. |
| `GET` | `LOCATION` | Вывод региона, в котором размещен бакет. |
| `PUT` | `BUCKET` | Создание бакета. |
| `HEAD` | `BUCKET` | Вывод метаданных бакета. |
| `GET` | `BUCKET` | Вывод списка объектов в бакете (предыдущая версия метода). |
| `GET` | `BUCKET_V1EXT` | Вывод списка объектов в бакете (предыдущая расширенная версия метода). |
| `GET` | `BUCKET_V2` | Вывод списка объектов в бакете (актуальная версия метода). |
| `DELETE` | `BUCKET` | Удаление бакета. |
| `OPTIONS` | `BUCKET` | Проверка возможности [CORS](concepts/cors.md)-запроса к бакету. |
| `GET` | `BUCKETVERSIONS` | Вывод [метаданных](concepts/object.md#metadata) обо всех версиях объектов в бакете. |
| `GET` | `VERSIONING` | Вывод состояния опции [версионирования](concepts/versioning.md) бакета. |
| `PUT` | `VERSIONING` | Включение или приостановка версионирования бакета. |
| `PUT` | `BUCKET_ACL` | Загрузка [списка управления доступом](concepts/acl.md) для бакета. |
| `GET` | `BUCKET_ACL` | Вывод списка управления доступом для бакета. |
| `PUT` | `BUCKET_CORS` | Загрузка конфигурации CORS для бакета. |
| `GET` | `BUCKET_CORS` | Вывод конфигурации CORS для бакета. |
| `DELETE` | `BUCKET_CORS` | Удаление конфигурации CORS для бакета. |
| `PUT` | `BUCKET_LIFECYCLE` | Загрузка конфигурации [жизненных циклов](concepts/lifecycles.md) объектов в бакете. |
| `GET` | `BUCKET_LIFECYCLE` | Вывод конфигурации жизненных циклов объектов в бакете. |
| `DELETE` | `BUCKET_LIFECYCLE` | Удаление конфигурации жизненных циклов объектов в бакете. |
| `PUT` | `BUCKET_ENCRYPTION` | Включение [шифрования](concepts/encryption.md) бакета. |
| `GET` | `BUCKET_ENCRYPTION` | Вывод информации о шифровании бакета. |
| `DELETE` | `BUCKET_ENCRYPTION` | Отключение шифрования бакета. |
| `PUT` | `BUCKET_WEBSITES` | Загрузка конфигурации [хостинга статического сайта](concepts/hosting.md) для бакета. |
| `GET` | `BUCKET_WEBSITES` | Вывод конфигурации хостинга статического сайта для бакета. |
| `DELETE` | `BUCKET_WEBSITES` | Удаление конфигурации хостинга статических сайтов для бакета. |
| `GET` | `BUCKET_LOGGING` | Вывод настроек логирования действий с бакетом. |
| `PUT` | `BUCKET_LOGGING` | [Включение](operations/buckets/enable-logging.md#enable) и [выключение](operations/buckets/enable-logging.md#stop-logging) механизма логирования действий с бакетом. |
| `PUT` | `BUCKETPOLICY` | Применение [политики доступа](concepts/policy.md) к заданному бакету. |
| `GET` | `BUCKETPOLICY` | Вывод политики доступа для заданного бакета. |
| `DELETE` | `BUCKETPOLICY` | Удаление политики доступа заданного бакета. |
| `PUT` | `BUCKET_TAGGING` | Назначение [меток](concepts/tags.md) бакету. |
| `GET` | `BUCKET_TAGGING` | Вывод меток бакета. |
| `DELETE` | `BUCKET_TAGGING` | Удаление меток бакета. |
| `PUT` | `OBJECT` | Загрузка объекта и его метаданных в бакет. |
| `POST` | `OBJECT` | Загрузка объекта и его метаданных в бакет через HTML-форму. |
| `POST` | `OBJECT_RESTORE` | Восстановление объекта в бакете из архива. Не поддерживается {{ objstorage-name }}. |
| `PUT` | `OBJECT_COPY` | Создание копии объекта, хранящегося в бакете. |
| `DELETE` | `OBJECT` | Удаление объекта. |
| `DELETE` | `MULTIPLE_OBJECTS` | Удаление объектов по списку ключей, переданному в запросе. |
| `HEAD` | `OBJECT` | Вывод метаданных объекта. |
| `GET` | `OBJECT` | Получение объекта из бакета. |
| `OPTIONS` | `OBJECT` | Проверка возможности CORS-запроса к объекту. |
| `GET` | `OBJECT_ACL` | Вывод списка управления доступом для объекта. |
| `PUT` | `OBJECT_ACL` | Загрузка списка управления доступом для объекта. |
| `SELECT` | `OBJECT_CONTENT` | Фильтрация и вывод содержимого объекта на основе [запроса на языке S3 Select](concepts/s3-select-language.md). |
| `PUT` | `OBJECT_TAGGING` | Назначение меток объекту. |
| `GET` | `OBJECT_TAGGING` | Вывод меток объекта. |
| `DELETE` | `OBJECT_TAGGING` | Удаление меток объекта. |
| `GET` | `MULTIPART` | Вывод списка текущих [составных загрузок](concepts/multipart.md). |
| `POST` | `INIT_MULTIPART` | Вывод идентификатора для дальнейших операций по составной загрузке объекта. |
| `POST` | `COMPLETE_MULTIPART` | Завершение составной загрузки объекта. |
| `DELETE` | `ABORT_MULTIPART` | Прерывание составной загрузки и удаление всех уже сохраненных частей объекта из бакета. |
| `PUT` | `UPLOAD_PART` | Сохранение части объекта. |
| `PUT` | `UPLOAD_PART_COPY` | Копирование части объекта. |
| `GET` | `UPLOAD_PART` | Вывод списка уже загруженных частей объекта для указанной составной загрузки. |
