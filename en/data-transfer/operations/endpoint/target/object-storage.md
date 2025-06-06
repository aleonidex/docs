---
title: Transferring data to a {{ objstorage-full-name }} target endpoint
description: In this tutorial, you will learn how to set up an {{ objstorage-name }} target endpoint.
---

# Transferring data to a {{ objstorage-full-name }} target endpoint

{{ data-transfer-full-name }} enables you to migrate data to the {{ objstorage-full-name }} storage and implement various data transfer, processing, and transformation scenarios. To implement a transfer:

1. [Explore possible data transfer scenarios](#scenarios).
1. [Configure one of the supported data sources](#supported-sources).
1. [Configure the target endpoint](#endpoint-settings) in {{ data-transfer-full-name }}.
1. [Create](../../transfer.md#create) a transfer and [start](../../transfer.md#activate) it.
1. Perform required operations with the storage and [control the transfer](../../monitoring.md).
1. In case of any issues, [use ready-made solutions](../../../../data-transfer/troubleshooting/index.md) to resolve them.

## Scenarios for transferring data to {{ objstorage-full-name }} {#scenarios}

1. {% include [queue](../../../../_includes/data-transfer/scenario-captions/queue.md) %}
    
    * [Delivering data from {{ DS }} to {{ objstorage-name }}](../../../tutorials/yds-to-objstorage.md).   

1. {% include [data-mart](../../../../_includes/data-transfer/scenario-captions/storage.md) %}
    
    * [Loading data from {{ MY }} to {{ objstorage-name }}](../../../tutorials/mmy-objs-migration.md).
    * [Loading data from {{ PG }} to {{ objstorage-name }}](../../../tutorials/mpg-to-objstorage.md).
    * [Loading data from {{ OS }} to {{ objstorage-name }}](../../../tutorials/opensearch-to-object-storage.md).
    * [Loading data from {{ ydb-name }} to {{ objstorage-name }}](../../../tutorials/ydb-to-object-storage.md).

For a detailed description of possible {{ data-transfer-full-name }} scenarios, see [Tutorials](../../../tutorials/index.md).

## Configuring the data source {#supported-sources}

Configure one of the supported data sources:

* [{{ PG }}](../source/postgresql.md)
* [{{ MY }}](../source/mysql.md)
* [{{ MG }}](../source/mongodb.md)
* [{{ KF }}](../source/kafka.md)
* [{{ AB }}](../../../transfer-matrix.md#airbyte)
* [{{ DS }}](../source/data-streams.md)
* [Oracle](../source/oracle.md)
* [{{ ydb-name }}](../source/ydb.md)
* [{{ ES }}](../source/elasticsearch.md)
* [{{ OS }}](../source/opensearch.md).

For a complete list of supported sources and targets in {{ data-transfer-full-name }}, see [Available transfers](../../../transfer-matrix.md).

{% note warning %}

{{ objstorage-name }} only supports inserting new data but not updating it. If there is a data update going on in the source, it should not be used to supply data to {{ objstorage-name }}; otherwise, the transfer will fail with an [error](#update-not-supported).

{% endnote %}

## Configuring the {{ objstorage-name }} target endpoint {#endpoint-settings}

When [creating](../index.md#create) or [updating](../index.md#update) an endpoint, you can define:

* [Configuration settings](#bucket-config) for an {{ objstorage-full-name }} bucket or custom S3-compatible storage.
* [Additional parameters](#additional-settings).

### Bucket configurations {#bucket-config}

{% list tabs group=instructions %}

- {{ objstorage-full-name }} bucket {#obj-storage}

    
    * **{{ ui-key.yc-data-transfer.data-transfer.console.form.object_storage.console.form.object_storage.ConnectionSettings.bucket.title }}**: Name of the [bucket](../../../../storage/concepts/bucket.md) to upload source data to.
    * **{{ ui-key.yc-data-transfer.data-transfer.console.form.object_storage.console.form.object_storage.ObjectStorageConnectionSettings.service_account_id.title }}**: [Service account](../../../../iam/concepts/users/service-accounts.md) with the `storage.uploader` role that will be used to access [{{ yds-full-name }}](../../../../data-streams/).


- Custom S3-compatible storage {#s3-storage}

    * **{{ ui-key.yc-data-transfer.data-transfer.console.form.object_storage.console.form.object_storage.ConnectionSettings.bucket.title }}**: Bucket name.
    * **{{ ui-key.yc-data-transfer.data-transfer.console.form.object_storage.console.form.object_storage.ObjectStorageSource.ObjectStorageEventSource.SQS.aws_access_key_id.title }}** and **{{ ui-key.yc-data-transfer.data-transfer.console.form.object_storage.console.form.object_storage.ObjectStorageSource.ObjectStorageEventSource.SQS.aws_secret_access_key.title }}**: [ID and contents of the AWS key](https://docs.aws.amazon.com/general/latest/gr/aws-sec-cred-types.html#access-keys-and-secret-access-keys) used to access a private bucket.
    * (Optional) **{{ ui-key.yc-data-transfer.data-transfer.console.form.object_storage.console.form.object_storage.ConnectionSettings.endpoint.title }}**: Endpoint for an Amazon S3-compatible service. Leave this field empty to use Amazon.
    * **{{ ui-key.yc-data-transfer.data-transfer.console.form.object_storage.console.form.object_storage.ObjectStorageConnectionSettings.region.title }}**: Region to send requests.
    * **{{ ui-key.yc-data-transfer.data-transfer.console.form.object_storage.console.form.object_storage.ConnectionSettings.use_ssl.title }}**: Select this option if the remote server uses a secure SSL/TLS connection.
    * **{{ ui-key.yc-data-transfer.data-transfer.console.form.object_storage.console.form.object_storage.ConnectionSettings.verify_ssl_cert.title }}**: Allow self-signed certificates.

{% endlist %}

### Additional settings {#additional-settings}

* **{{ ui-key.yc-data-transfer.data-transfer.console.form.object_storage.console.form.object_storage.ObjectStorageTarget.output_format.title }}**: Format in which the data will be written to the bucket: `{{ ui-key.yc-data-transfer.data-transfer.console.form.object_storage.console.form.object_storage.ObjectStorageSerializationFormatUI.OBJECT_STORAGE_SERIALIZATION_FORMAT_JSON.title }}`, `{{ ui-key.yc-data-transfer.data-transfer.console.form.object_storage.console.form.object_storage.ObjectStorageSerializationFormatUI.OBJECT_STORAGE_SERIALIZATION_FORMAT_CSV.title }}`, `PARQUET`, or `{{ ui-key.yc-data-transfer.data-transfer.console.form.object_storage.console.form.object_storage.ObjectStorageSerializationFormatUI.OBJECT_STORAGE_SERIALIZATION_FORMAT_RAW.title }}`. To learn more, see [{#T}](../../../concepts/serializer.md#serializer-s3).

* **{{ ui-key.yc-data-transfer.data-transfer.console.form.object_storage.console.form.object_storage.ObjectStorageTarget.any_as_string.title }}**: Conversion of complex values to strings for `{{ ui-key.yc-data-transfer.data-transfer.console.form.object_storage.console.form.object_storage.ObjectStorageSerializationFormatUI.OBJECT_STORAGE_SERIALIZATION_FORMAT_JSON.title }}` output format.

* **{{ ui-key.yc-data-transfer.data-transfer.console.form.object_storage.console.form.object_storage.ObjectStorageTarget.output_encoding.title }}**: Compression of output data (`{{ ui-key.yc-data-transfer.data-transfer.console.form.object_storage.console.form.object_storage.ObjectStorageCodecUI.GZIP.title }}` or `{{ ui-key.yc-data-transfer.data-transfer.console.form.object_storage.console.form.object_storage.ObjectStorageCodecUI.UNCOMPRESSED.title }}`).

* **{{ ui-key.yc-data-transfer.data-transfer.console.form.object_storage.console.form.object_storage.ObjectStorageAdvancedSettings.buffer_size.title }}**: Size of files the data will be split into.

* **{{ ui-key.yc-data-transfer.data-transfer.console.form.object_storage.console.form.object_storage.ObjectStorageAdvancedSettings.buffer_interval.title }}**: Time after which the file will be written, regardless of its size.

* **{{ ui-key.yc-data-transfer.data-transfer.console.form.object_storage.console.form.object_storage.ObjectStorageAdvancedSettings.bucket_layout.title }}**: Object folder name. It supports the data layout pattern by date. Example: `2006/01/02/<folder_name>`.

* **{{ ui-key.yc-data-transfer.data-transfer.console.form.object_storage.console.form.object_storage.ObjectStorageAdvancedSettings.bucket_layout_timezone.title }}**: Time zone according to which time the files are distributed. Only affects the distribution of files to folders in the bucket, but does not affect the data within the files.

* **{{ ui-key.yc-data-transfer.data-transfer.console.form.object_storage.console.form.object_storage.ObjectStorageAdvancedSettings.bucket_layout_column.title }}**: Column name to specify logical time for the data. The default value is the system recording time. When recording data to the target, the time is converted to UTC. This behavior cannot be changed.

After configuring the data source and target, [create and start the transfer](../../transfer.md#create).

## Troubleshooting data transfer issues {#troubleshooting}

For more troubleshooting tips, see [Troubleshooting](../../../troubleshooting/index.md).

{% include [update-not-supported](../../../../_includes/data-transfer/troubles/object-storage/update-not-supported.md) %}
