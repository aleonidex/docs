---
editable: false
sourcePath: en/_api-ref-grpc/dataproc/v1/api-ref/grpc/ResourcePreset/get.md
---

# Yandex Data Processing API, gRPC: ResourcePresetService.Get

Returns the specified ResourcePreset resource.

To get the list of available ResourcePreset resources, make a [List](/docs/data-proc/api-ref/grpc/ResourcePreset/list#List) request.

## gRPC request

**rpc Get ([GetResourcePresetRequest](#yandex.cloud.dataproc.v1.GetResourcePresetRequest)) returns ([ResourcePreset](#yandex.cloud.dataproc.v1.ResourcePreset))**

## GetResourcePresetRequest {#yandex.cloud.dataproc.v1.GetResourcePresetRequest}

```json
{
  "resource_preset_id": "string"
}
```

#|
||Field | Description ||
|| resource_preset_id | **string**

Required field. Required. ID of the resource preset to return.
To get the resource preset ID, use a [ResourcePresetService.List](/docs/data-proc/api-ref/grpc/ResourcePreset/list#List) request. ||
|#

## ResourcePreset {#yandex.cloud.dataproc.v1.ResourcePreset}

```json
{
  "id": "string",
  "zone_ids": [
    "string"
  ],
  "cores": "int64",
  "memory": "int64"
}
```

A ResourcePreset resource for describing hardware configuration presets.

#|
||Field | Description ||
|| id | **string**

ID of the ResourcePreset resource. ||
|| zone_ids[] | **string**

IDs of availability zones where the resource preset is available. ||
|| cores | **int64**

Number of CPU cores for a Yandex Data Processing host created with the preset. ||
|| memory | **int64**

RAM volume for a Yandex Data Processing host created with the preset, in bytes. ||
|#