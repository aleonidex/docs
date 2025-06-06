# Viewing operations with {{ load-testing-name }} resources

The system logs all actions with {{ load-testing-name }} resources as a list of operations. Each operation gets its own unique ID.

## Getting a list of operations {#get-operations}

{% list tabs group=instructions %}

- Management console {#console}

  1. In the [management console]({{ link-console-main }}), open the folder where you want to view operations.
  1. In the list of services, select **{{ ui-key.yacloud.iam.folder.dashboard.label_load-testing }}**.
  1. In the left-hand panel, select ![image](../../_assets/operations.svg) **{{ ui-key.yacloud.common.operations-key-value }}**.

  In the list that opens, you will also see operations with the resources that were deleted.

{% endlist %}

## Getting operation details {#get-operations-info}

1. [Get a list of operations](#get-operations) for the resource.
1. Copy the operation ID.
1. Get details of the operation:

    {% list tabs group=instructions %}

    - CLI {#cli}

      {% include [cli-install](../../_includes/cli-install.md) %}

      {% include [default-catalogue](../../_includes/default-catalogue.md) %}

      Run this command:

      ```bash
      yc operation get <operation_ID>
      ```

      Result:

      ```text
      id: ff67zb1214j6********
      description: Create Job
      created_at: "2024-02-01T09:07:33.271288Z"
      created_by: ajego134p5h1********
      modified_at: "2024-02-01T09:07:33.398853Z"
      done: true
      metadata:
        '@type': type.googleapis.com/google.protobuf.Empty
        value: {}
      response:
        '@type': type.googleapis.com/google.protobuf.Empty
        value: {}
      ```

    - API {#api}

      Use the [get](../../api-design-guide/concepts/operation.md#monitoring) REST API method or the [OperationService/Get](../user/api-ref/grpc/Operation/get.md) gRPC API call.

    {% endlist %}

#### See also {#see-also}

* [{#T}](../../api-design-guide/concepts/about-async.md)
