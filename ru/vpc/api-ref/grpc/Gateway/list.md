---
editable: false
sourcePath: en/_api-ref-grpc/vpc/v1/api-ref/grpc/Gateway/list.md
---

# Virtual Private Cloud API, gRPC: GatewayService.List

Retrieves the list of Gateway resources in the specified folder.

## gRPC request

**rpc List ([ListGatewaysRequest](#yandex.cloud.vpc.v1.ListGatewaysRequest)) returns ([ListGatewaysResponse](#yandex.cloud.vpc.v1.ListGatewaysResponse))**

## ListGatewaysRequest {#yandex.cloud.vpc.v1.ListGatewaysRequest}

```json
{
  "folder_id": "string",
  "page_size": "int64",
  "page_token": "string",
  "filter": "string"
}
```

#|
||Field | Description ||
|| folder_id | **string**

Required field. ID of the folder to list gateways in.

To get the folder ID use a [yandex.cloud.resourcemanager.v1.FolderService.List](/docs/resource-manager/api-ref/grpc/Folder/list#List) request. ||
|| page_size | **int64**

The maximum number of results per page to return. If the number of available
results is larger than `page_size`, the service returns a [ListGatewaysResponse.next_page_token](#yandex.cloud.vpc.v1.ListGatewaysResponse)
that can be used to get the next page of results in subsequent list requests.
Default value: 100. ||
|| page_token | **string**

Page token. To get the next page of results, set `page_token` to the
[ListGatewaysResponse.next_page_token](#yandex.cloud.vpc.v1.ListGatewaysResponse) returned by a previous list request. ||
|| filter | **string**

A filter expression that filters Gateway listed in the response.

The expression must specify:
1. The field name. Currently you can use filtering only on [Gateway.name](#yandex.cloud.vpc.v1.Gateway) field.
2. An `=` operator.
3. The value in double quotes (`"`). Must be 3-63 characters long and match the regular expression `[a-z][-a-z0-9]{1,61}[a-z0-9]`.
Example of a filter: `name=my-gateway`. ||
|#

## ListGatewaysResponse {#yandex.cloud.vpc.v1.ListGatewaysResponse}

```json
{
  "gateways": [
    {
      "id": "string",
      "folder_id": "string",
      "created_at": "google.protobuf.Timestamp",
      "name": "string",
      "description": "string",
      "labels": "map<string, string>",
      // Includes only one of the fields `shared_egress_gateway`
      "shared_egress_gateway": "SharedEgressGateway"
      // end of the list of possible fields
    }
  ],
  "next_page_token": "string"
}
```

#|
||Field | Description ||
|| gateways[] | **[Gateway](#yandex.cloud.vpc.v1.Gateway)**

List of gateways. ||
|| next_page_token | **string**

Token for getting the next page of the list. If the number of results is greater than
the specified [ListGatewaysRequest.page_size](#yandex.cloud.vpc.v1.ListGatewaysRequest), use `next_page_token` as the value
for the [ListGatewaysRequest.page_token](#yandex.cloud.vpc.v1.ListGatewaysRequest) parameter in the next list request.

Each subsequent page will have its own `next_page_token` to continue paging through the results. ||
|#

## Gateway {#yandex.cloud.vpc.v1.Gateway}

A Gateway resource. For more information, see [Gateway](/docs/vpc/concepts/gateways).

#|
||Field | Description ||
|| id | **string**

ID of the gateway. Generated at creation time. ||
|| folder_id | **string**

ID of the folder that the gateway belongs to. ||
|| created_at | **[google.protobuf.Timestamp](https://developers.google.com/protocol-buffers/docs/reference/google.protobuf#timestamp)**

Creation timestamp. ||
|| name | **string**

Name of the gateway.
The name is unique within the folder.
Value must match the regular expression ``\\|[a-z]([-a-z0-9]{0,61}[a-z0-9])?``. ||
|| description | **string**

Description of the gateway. 0-256 characters long. ||
|| labels | **object** (map<**string**, **string**>)

Gateway labels as `key:value` pairs.
No more than 64 per resource.
The maximum string length in characters for each value is 63.
Each value must match the regular expression `[-_./\\@0-9a-z]*`.
The string length in characters for each key must be 1-63.
Each key must match the regular expression `[a-z][-_./\\@0-9a-z]*`. ||
|| shared_egress_gateway | **[SharedEgressGateway](#yandex.cloud.vpc.v1.SharedEgressGateway)**

Includes only one of the fields `shared_egress_gateway`.

Gateway specification ||
|#

## SharedEgressGateway {#yandex.cloud.vpc.v1.SharedEgressGateway}

Shared Egress Gateway configuration

#|
||Field | Description ||
|| Empty | > ||
|#