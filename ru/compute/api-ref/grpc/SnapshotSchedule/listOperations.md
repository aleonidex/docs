---
editable: false
sourcePath: en/_api-ref-grpc/compute/v1/api-ref/grpc/SnapshotSchedule/listOperations.md
---

# Compute Cloud API, gRPC: SnapshotScheduleService.ListOperations

Lists operations for the specified snapshot schedule.

## gRPC request

**rpc ListOperations ([ListSnapshotScheduleOperationsRequest](#yandex.cloud.compute.v1.ListSnapshotScheduleOperationsRequest)) returns ([ListSnapshotScheduleOperationsResponse](#yandex.cloud.compute.v1.ListSnapshotScheduleOperationsResponse))**

## ListSnapshotScheduleOperationsRequest {#yandex.cloud.compute.v1.ListSnapshotScheduleOperationsRequest}

```json
{
  "snapshot_schedule_id": "string",
  "page_size": "int64",
  "page_token": "string"
}
```

#|
||Field | Description ||
|| snapshot_schedule_id | **string**

ID of the snapshot schedule to list operations for.

To get a snapshot schedule ID, make a [SnapshotScheduleService.List](/docs/compute/api-ref/grpc/SnapshotSchedule/list#List) request. ||
|| page_size | **int64**

The maximum number of results per page to return. If the number of available
results is larger than `page_size`, the service returns a [ListSnapshotScheduleOperationsResponse.next_page_token](#yandex.cloud.compute.v1.ListSnapshotScheduleOperationsResponse)
that can be used to get the next page of results in subsequent list requests.

Default value: 100. ||
|| page_token | **string**

Page token. To get the next page of results, set `page_token` to the
[ListSnapshotScheduleOperationsResponse.next_page_token](#yandex.cloud.compute.v1.ListSnapshotScheduleOperationsResponse) returned by a previous list request. ||
|#

## ListSnapshotScheduleOperationsResponse {#yandex.cloud.compute.v1.ListSnapshotScheduleOperationsResponse}

```json
{
  "operations": [
    {
      "id": "string",
      "description": "string",
      "created_at": "google.protobuf.Timestamp",
      "created_by": "string",
      "modified_at": "google.protobuf.Timestamp",
      "done": "bool",
      "metadata": "google.protobuf.Any",
      // Includes only one of the fields `error`, `response`
      "error": "google.rpc.Status",
      "response": "google.protobuf.Any"
      // end of the list of possible fields
    }
  ],
  "next_page_token": "string"
}
```

#|
||Field | Description ||
|| operations[] | **[Operation](#yandex.cloud.operation.Operation)**

List of operations for the specified snapshot schedule. ||
|| next_page_token | **string**

Token for getting the next page of the list. If the number of results is greater than
the specified [ListSnapshotScheduleOperationsRequest.page_size](#yandex.cloud.compute.v1.ListSnapshotScheduleOperationsRequest), use `next_page_token` as the value
for the [ListSnapshotScheduleOperationsRequest.page_token](#yandex.cloud.compute.v1.ListSnapshotScheduleOperationsRequest) parameter in the next list request.

Each subsequent page will have its own `next_page_token` to continue paging through the results. ||
|#

## Operation {#yandex.cloud.operation.Operation}

An Operation resource. For more information, see [Operation](/docs/api-design-guide/concepts/operation).

#|
||Field | Description ||
|| id | **string**

ID of the operation. ||
|| description | **string**

Description of the operation. 0-256 characters long. ||
|| created_at | **[google.protobuf.Timestamp](https://developers.google.com/protocol-buffers/docs/reference/google.protobuf#timestamp)**

Creation timestamp. ||
|| created_by | **string**

ID of the user or service account who initiated the operation. ||
|| modified_at | **[google.protobuf.Timestamp](https://developers.google.com/protocol-buffers/docs/reference/google.protobuf#timestamp)**

The time when the Operation resource was last modified. ||
|| done | **bool**

If the value is `false`, it means the operation is still in progress.
If `true`, the operation is completed, and either `error` or `response` is available. ||
|| metadata | **[google.protobuf.Any](https://developers.google.com/protocol-buffers/docs/proto3#any)**

Service-specific metadata associated with the operation.
It typically contains the ID of the target resource that the operation is performed on.
Any method that returns a long-running operation should document the metadata type, if any. ||
|| error | **[google.rpc.Status](https://cloud.google.com/tasks/docs/reference/rpc/google.rpc#status)**

The error result of the operation in case of failure or cancellation.

Includes only one of the fields `error`, `response`.

The operation result.
If `done == false` and there was no failure detected, neither `error` nor `response` is set.
If `done == false` and there was a failure detected, `error` is set.
If `done == true`, exactly one of `error` or `response` is set. ||
|| response | **[google.protobuf.Any](https://developers.google.com/protocol-buffers/docs/proto3#any)**

The normal response of the operation in case of success.
If the original method returns no data on success, such as Delete,
the response is [google.protobuf.Empty](https://developers.google.com/protocol-buffers/docs/reference/google.protobuf#google.protobuf.Empty).
If the original method is the standard Create/Update,
the response should be the target resource of the operation.
Any method that returns a long-running operation should document the response type, if any.

Includes only one of the fields `error`, `response`.

The operation result.
If `done == false` and there was no failure detected, neither `error` nor `response` is set.
If `done == false` and there was a failure detected, `error` is set.
If `done == true`, exactly one of `error` or `response` is set. ||
|#