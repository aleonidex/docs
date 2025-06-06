# Regular recognition of audio files from {{ objstorage-full-name }}


The {{ speechkit-short-name }} [asynchronous recognition API](../../speechkit/stt/api/transcribation-api.md) is integrated with {{ objstorage-full-name }}. Therefore, you can set up automatic recognition of audio files of [supported formats](../../speechkit/formats.md) that are regularly uploaded to an {{ objstorage-name }} bucket. A cloud function in {{ sf-full-name }} regularly checks the bucket for audio files and sends them to the {{ speechkit-short-name }} API for recognition. The recognition result and status are saved to the same {{ objstorage-name }} bucket.

To set up automatic recognition of audio files using {{ speechkit-short-name }}:

1. [Create a cloud function](#create-function) to read files from your {{ objstorage-name }} bucket, send them to the API, and check the file recognition status.
1. [Create a trigger](#create-trigger) to regularly invoke your cloud function.
1. [Test the function](#check-function).

## Getting started {#before-you-begin}

1. [Create](../../iam/operations/sa/create.md) a service account named `asr-batch-sa`.
1. [Assign](../../iam/operations/sa/assign-role-for-sa.md) the `storage.editor`, `{{ roles-functions-invoker }}`, and `{{ roles-speechkit-stt }}` roles to the service account for the folder in which it was created.
1. [Create](../../iam/operations/authentication/manage-access-keys.md#create-access-key) a static access key for the service account.


1. [Create](../../iam/operations/authentication/manage-api-keys.md#create-api-key) an API key to access the service account.
1. [Create](../../storage/operations/buckets/create.md) an {{ objstorage-name }} bucket named `asr-batch-bucket` in the service account folder.
1. Open `asr-batch-bucket`, click **{{ ui-key.yacloud.storage.bucket.button_create }}**, and specify `input` in the **{{ ui-key.yacloud.storage.bucket.popup-create-folder_field_name}}** field.
1. [Upload](../../storage/operations/objects/upload.md#simple) the `config.json` file with the specified [recognition language](../../speechkit/stt/models.md#languages) to the bucket's `input` folder. The file only contains one setting:

   ```json
   {
     "lang": "<language_code>"
   }
   ```

   {% note info %}

   If there is no `config.json` file in the bucket, the recognition language will be Russian.

   {% endnote %}

## Create a cloud function {#create-function}

1. In the [management console]({{ link-console-main }}), navigate to the folder with the new service account.
1. Select **{{ ui-key.yacloud.iam.folder.dashboard.label_serverless-functions }}**.
1. Click **{{ ui-key.yacloud.serverless-functions.list.button_create }}** and specify `asr-batch-function` as the function name.
1. Click **{{ ui-key.yacloud.common.create }}**.
1. Under **{{ ui-key.yacloud.serverless-functions.item.editor.label_title }}**, select the `Python` `3.8` runtime environment and click **{{ ui-key.yacloud.serverless-functions.item.editor.button_action-continue }}**.
1. Download a [script file](https://github.com/yandex-cloud-examples/yc-speechkit-async-recognizer/blob/main/examples/asr-batch-function/functions/main.py) from the {{ yandex-cloud }} repository.
1. Under **{{ ui-key.yacloud.serverless-functions.item.editor.label_title-source }}**, clear the contents of the `index.py` file and paste the downloaded script.
1. Under **{{ ui-key.yacloud.serverless-functions.item.editor.label_title-source }}**, create a file named `requirements.txt` and add the following code to it:

   ```text
   boto3
   botocore
   requests
   ```

1. Specify the function run settings:
   * **{{ ui-key.yacloud.serverless-functions.item.editor.field_entry }}**: `index.handler`
   * **{{ ui-key.yacloud.serverless-functions.item.editor.field_timeout }}**: `60`
   * **{{ ui-key.yacloud.forms.label_service-account-select }}**: `asr-batch-sa`
1. Add these environment variables:
   * `S3_BUCKET`: `asr-batch-bucket`
   * `S3_PREFIX`: `input`
   * `S3_PREFIX_LOG`: `log`
   * `S3_PREFIX_OUT`: `out`
   * `S3_KEY`: Static access key ID
   * `S3_SECRET`: Static access key secret
   * `API_KEY`: API key ID
   * `API_SECRET`: API key secret

1. Click **{{ ui-key.yacloud.serverless-functions.item.editor.button_deploy-version }}**.

## Create a trigger {#create-trigger}

1. In the management console, select **{{ ui-key.yacloud.iam.folder.dashboard.label_serverless-functions }}**.
1. Select **{{ ui-key.yacloud.serverless-functions.switch_list-triggers }}**.
1. Click **{{ ui-key.yacloud.serverless-functions.triggers.list.button_create }}**.
1. Specify the trigger settings:
   * **{{ ui-key.yacloud.serverless-functions.triggers.form.field_name }}**: `asr-batch-cron`.
   * **{{ ui-key.yacloud.serverless-functions.triggers.form.field_type }}**: `{{ ui-key.yacloud.serverless-functions.triggers.form.label_timer }}`.
   * **{{ ui-key.yacloud.serverless-functions.triggers.form.field_invoke }}**: `{{ ui-key.yacloud.serverless-functions.triggers.form.label_function }}`.
   * **{{ ui-key.yacloud.serverless-functions.triggers.form.field_cron-expression }}**: `{{ ui-key.yacloud.common.button_cron-1min }}`.
   * **{{ ui-key.yacloud.serverless-functions.triggers.form.field_function }}**: `asr-batch-function`.
   * **{{ ui-key.yacloud.serverless-functions.triggers.form.field_function-tag }}**: `$latest`.
   * **{{ ui-key.yacloud.serverless-functions.triggers.form.field_service-account }}**: `asr-batch-sa`.
1. Click **{{ ui-key.yacloud.serverless-functions.triggers.form.button_create-trigger }}**.

The trigger you created will fire once a minute and invoke the [cloud function](#create-function).

## Test the function {#check-function}

1. In the management console, select **{{ ui-key.yacloud.iam.folder.dashboard.label_storage }}** and open `asr-batch-bucket`.
1. [Upload](../../storage/operations/objects/upload.md#simple) audio files of any [supported format](../../speechkit/formats.md) to the `input` folder.
1. Wait a few minutes and make sure the bucket now contains the `log` and `out` folders.
1. Check the recognition status in the `log` folder. The status of each audio file sent for recognition is saved to an auxiliary file named `<audio_file_name>.json` (e.g., `audio.mp3.json`). The `"done": "false"` parameter in the file indicates the recognition process is not completed.
1. Check the recognition result in the `out` folder. The result is saved to a JSON file named `<audio_file_name>.json` (e.g., `audio.mp3.json`). To learn more about the recognition result format, see [Asynchronous recognition API](../../speechkit/stt/api/transcribation-api.md#get-result-response).

{% note info %}

You can monitor the progress of the script in the [logs](../../functions/operations/function/function-logs.md) of `asr-batch-function`.

{% endnote %}


#### See also {#see-also}

* [{#T}](../../speechkit/stt/api/transcribation-api.md)
* [{#T}](../../speechkit/stt/api/transcribation-lpcm.md)
* [{#T}](../../speechkit/stt/api/transcribation-ogg.md)

