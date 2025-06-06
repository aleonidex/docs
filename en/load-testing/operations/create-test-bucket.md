---
title: How to create a test with data from a bucket in {{ load-testing-full-name }}
description: Follow this guide to create a test with data from a bucket.
---

# Creating a test with data from a bucket

You can upload test data to {{ load-testing-name }} from a [bucket](../../storage/concepts/bucket.md) in [{{ objstorage-full-name }}](../../storage/). This method enables you to:
* Handle large payloads (over 100 MB).
* Store payloads and reuse them easily.
* Ensure maximum security (payloads remain in the user's cloud and associated networks).

## Setting up your infrastructure {#infrastructure-prepare}

1. [Create](create-agent.md) a test agent.
1. [Create](../../storage/operations/buckets/create.md) a bucket and [upload](../../storage/operations/objects/upload.md) the payload file to it.
1. Grant the service account linked to the [agent](../concepts/agent.md) read permissions for the bucket. To do this, edit the bucket [ACL](../../storage/concepts/acl.md):
    1. In the [management console]({{ link-console-main }}), select the folder with the bucket.
    1. In the list of services, select **{{ ui-key.yacloud.iam.folder.dashboard.label_storage }}**.
    1. Click ![image](../../_assets/horizontal-ellipsis.svg) next to the bucket and select **{{ ui-key.yacloud.storage.buckets.button_permissions }}**.
    1. In the window that opens, enter the service account name, select `READ` for bucket permissions, and click **{{ ui-key.yacloud.common.add }}**.
    1. Click **{{ ui-key.yacloud.common.save }}**.

    The service account will only get read permissions for this bucket.

    {% note info %}

    As an alternative to editing the bucket ACL, you can [assign](../../iam/operations/roles/grant.md) the `storage.editor` [role](../../storage/security/#storage-editor) to the service account. 

    This method is less secure because the service account will have unrestricted access to any operations with all buckets in the [folder](../../resource-manager/concepts/resources-hierarchy.md#folder), such as creating, deleting, updating, or enabling public access.

    {% endnote %}

## Creating a test {#create-test}



{% list tabs group=instructions %}

- Management console {#console}
  1. In the [management console]({{ link-console-main }}), select the folder for testing.
  1. In the list of services, select **{{ ui-key.yacloud.iam.folder.dashboard.label_load-testing }}**.
  1. In the ![image](../../_assets/load-testing/test.svg) **{{ ui-key.yacloud.load-testing.label_tests-list }}** tab, click **{{ ui-key.yacloud.load-testing.button_create-test }}**.
  1. [Select](../concepts/agent-select.md) a test agent or create a new one.
  1. Under **Attached files**:
      1. Click **{{ ui-key.yacloud.load-testing.label_choose-from-bucket }}**.
      1. Find the bucket in the drop-down list and specify the path to the payload file.

      Alternatively, you can **Select files** from your computer. In this case, you will have to reupload the files every time you run the test.

  1. Based on the selected configuration type, set the parameters under **{{ ui-key.yacloud.load-testing.label_test-settings }}**:

      {% include [test settings](../../_includes/load-testing/test-settings.md) %}

  1. Click **{{ ui-key.yacloud.common.create }}**.

- CLI {#cli}

  {% include [cli-install](../../_includes/cli-install.md) %}

  {% include [default-catalogue](../../_includes/default-catalogue.md) %}

  1. See the description of the [CLI](../../cli/) command for creating a test:

     ```bash
     yc loadtesting test create --help
     ```

  1. Upload the YAML configuration file for your test:

     ```bash
     yc loadtesting test-config create --from-yaml-file <file_path>
     ```

  1. Upload the [payload](../../load-testing/concepts/payload). To do this, create an [IAM token](../../ydb/terraform/credentials):

     ```bash
     export YC_TOKEN=$(yc iam create-token)
     export TEST_PAYLOAD_FILE="sample/_requests.uri"
     export S3_PAYLOAD_BUCKET="my_bucket"
     export S3_PAYLOAD_FILENAME="my_requests.uri"
     curl -H "X-YaCloud-SubjectToken: ${$YC_TOKEN}" --upload-file - "https://storage.yandexcloud.net/${$S3_PAYLOAD_BUCKET}/${$S3_PAYLOAD_FILENAME}" < ${$TEST_PAYLOAD_FILE}
     ```

  1. Create a test in the default [folder](../../resource-manager/concepts/resources-hierarchy.md#folder):

     ```bash
     loadtesting test create \
     --name "yc-examples-test" \
     --description "Test has been created using YC" \
     --labels source=gh,type=tutorial \
     --configuration id=ff6hvk749g71********,agent-id=ff633vbrst7a********,test-data=requests.uri \
     --test-data name=requests.uri,s3bucket=my_bucket,s3file=my_requests.uri
     ```

     Where:
     * `--name`: Test name.
     * `--description`: Test description.
     * `--labels`: Test [labels](../../resource-manager/concepts/labels.md).
     * `--configuration`: Test configuration parameters:
         * `id`: Test configuration ID. Specify the ID of the configuration file you uploaded in _Step 2_. To get a list of configuration IDs, run the `yc loadtesting test-config list` command.
         * `agent-id`: Test agent ID.
         * `test-data`: Payload file name.
     * `--test-data`: Payload settings:
         * `name`: Payload object name.
         * `s3bucket`: Name of the bucket storing the payload.
         * `s3file`: Name of the payload file in the bucket.

     For more information on how to create a test with the CLI, see the [Yandex Cloud Examples repository](https://github.com/yandex-cloud-examples/yc-load-testing-start-test).

  1. To view a list of tests in the folder for load testing using the CLI, run the following command:

     ```bash
     yc loadtesting test list
     ```

     For more information on how to view test info using the CLI, see the [Yandex Cloud Examples repository](https://github.com/yandex-cloud-examples/yc-load-testing-list-tests).
    

{% endlist %}