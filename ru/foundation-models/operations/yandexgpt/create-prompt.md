---
title: Как отправить запрос в промт-режиме к {{ gpt-lite }} или {{ gpt-pro }}
description: Следуя данной инструкции, вы научитесь использовать {{ gpt-lite }} и {{ gpt-pro }} в промт-режиме.
---

# Отправить запрос в промт-режиме

Чтобы сгенерировать текст в [промт-режиме](../../concepts/index.md#working-mode), отправьте запрос к модели с помощью метода [completion](../../text-generation/api-ref/TextGeneration/completion.md) или [{{ ml-sdk-full-name }}](../../sdk/index.md).

## Перед началом работы {#before-begin}

{% list tabs group=programming_language %}

- SDK {#sdk}

  Чтобы воспользоваться примерами запросов с использованием SDK:

  {% include [sdk-before-begin-ai-langmodel-user](../../../_includes/foundation-models/sdk-before-begin-ai-langmodel-user.md) %}

- cURL {#curl}

  {% include notitle [ai-before-beginning](../../../_includes/foundation-models/yandexgpt/ai-before-beginning.md) %}

  {% include [curl](../../../_includes/curl.md) %}

- Python {#python}

  {% include notitle [ai-before-beginning](../../../_includes/foundation-models/yandexgpt/ai-before-beginning.md) %}

{% endlist %}


## Запрос к модели через REST API {#request}

{% list tabs group=programming_language %}

- cURL {#curl}
  
  {% include [bash-windows-note-single](../../../_includes/translate/bash-windows-note-single.md) %}

  1. Создайте файл с телом запроса (например, `body.json`):
  
     ```json
     {
       "modelUri": "gpt://<идентификатор_каталога>/yandexgpt",
       "completionOptions": {
         "stream": false,
         "temperature": 0.1,
         "maxTokens": "1000",
         "reasoningOptions": {
           "mode": "DISABLED"
         }
       },
       "messages": [
         {
           "role": "system",
           "text": "Переведи текст"
         },
         {
           "role": "user",
           "text": "To be, or not to be: that is the question."
         }
       ]
     }
     ```
  
     Где:
  
     {% include [api-parameters](../../../_includes/foundation-models/yandexgpt/api-parameters.md) %}
  
  1. Отправьте запрос к модели, выполнив команду:
  
     ```bash
     export FOLDER_ID=<идентификатор_каталога>
     export IAM_TOKEN=<IAM-токен>
     curl \
       --request POST \
       --header "Content-Type: application/json" \
       --header "Authorization: Bearer ${IAM_TOKEN}" \
       --header "x-folder-id: ${FOLDER_ID}" \
       --data "@<путь_до_файла_json>" \
       "https://llm.{{ api-host }}/foundationModels/v1/completion"
     ```
  
     Где:
  
     * `FOLDER_ID`— идентификатор каталога, на который у вашего аккаунта есть роль `{{ roles-yagpt-user }}` или выше.
     * `IAM_TOKEN` — IAM-токен, полученный [перед началом работы](#before-begin).
  
     {% cut "Результат:" %}
  
     ```json
     {
       "result": {
         "alternatives": [
           {
             "message": {
               "role": "assistant",
               "text": "Быть или не быть, вот в чём вопрос."
             },
             "status": "ALTERNATIVE_STATUS_FINAL"
           }
         ],
         "usage": {
           "inputTextTokens": "30",
           "completionTokens": "10",
           "totalTokens": "40",
           "completionTokensDetails": {
             "reasoningTokens": "0"
           }
         },
         "modelVersion": "09.02.2025"
       }
     }
     ```

     {% endcut %}

- Python {#python}

  1. Создайте файл `test.py` с кодом запроса к модели:

     ```python
     import requests
     import argparse
     
     URL = "https://llm.api.cloud.yandex.net/foundationModels/v1/completion"
     
     def run(iam_token, folder_id, user_text):
         # Собираем запрос
         data = {}
         # Указываем тип модели
         data["modelUri"] = f"gpt://{folder_id}/yandexgpt"
         # Настраиваем опции
         data["completionOptions"] = {"temperature": 0.3, "maxTokens": 1000}
         # Указываем контекст для модели
         data["messages"] = [
             {"role": "system", "text": "Исправь ошибки в тексте."},
             {"role": "user", "text": f"{user_text}"},
         ]
         
         # Отправляем запрос
         response = requests.post(
             URL,
             headers={
                 "Accept": "application/json",
                 "Authorization": f"Bearer {iam_token}"
             },
             json=data,
         ).json()
     
         #Распечатываем результат
         print(response)
     
     if __name__ == '__main__':
         parser = argparse.ArgumentParser()
         parser.add_argument("--iam_token", required=True, help="IAM token")
         parser.add_argument("--folder_id", required=True, help="Folder id")
         parser.add_argument("--user_text", required=True, help="User text")
         args = parser.parse_args()
         run(args.iam_token, args.folder_id, args.user_text)
     ```

  1. Запустите файл `test.py`, подставив значение [IAM-токена](../../../iam/concepts/authorization/iam-token.md) и [идентификатор каталога](../../../resource-manager/operations/folder/get-id.md):

     ```bash
     export IAM_TOKEN=<IAM-токен>
     export FOLDER_ID=<идентификатор_каталога>
     export TEXT='Ашипки саме сибя ни исрпвят'
     python test.py \
       --iam_token ${IAM_TOKEN} \
       --folder_id ${FOLDER_ID} \
       --user_text "${TEXT}"
     ```

     {% cut "Результат:" %}

     ```text
     {'result': {'alternatives': [{'message': {'role': 'assistant', 'text': 'Ошибки сами себя не исправят.'}, 'status': 'ALTERNATIVE_STATUS_FINAL'}], 'usage': {'inputTextTokens': '32', 'completionTokens': '8', 'totalTokens': '40', 'completionTokensDetails': {'reasoningTokens': '0'}}, 'modelVersion': '09.02.2025'}}
     ```

     {% endcut %}

{% endlist %}

## Запрос к модели через gRPC API {#request-grpc}

{% list tabs group=programming_language %}

- SDK {#sdk}

  {% include [yandexgpt-sdk-entire-generation-block](../../../_includes/foundation-models/yandexgpt/yandexgpt-sdk-entire-generation-block.md) %}

- Python {#python}

  {% include [bash-windows-note-single](../../../_includes/translate/bash-windows-note-single.md) %}

  1. Склонируйте репозиторий {{ yandex-cloud }} API:

     ```bash
     git clone https://github.com/yandex-cloud/cloudapi
     ```

  1. Установите пакет `grpcio-tools` с помощью менеджера пакетов pip:

     ```bash
     pip install grpcio-tools
     ```

  1. Перейдите в папку со склонированным репозиторием {{ yandex-cloud }} API:

     ```bash
     cd <путь_к_папке_cloudapi>
     ```

  1. Создайте папку `output`:

     ```bash
     mkdir output
     ```

  1. Сгенерируйте код интерфейса клиента:

     ```bash
     python -m grpc_tools.protoc -I . -I third_party/googleapis \
       --python_out=output \
       --grpc_python_out=output \
         google/api/http.proto \
         google/api/annotations.proto \
         yandex/cloud/api/operation.proto \
         google/rpc/status.proto \
         yandex/cloud/operation/operation.proto \
         yandex/cloud/validation.proto \
         yandex/cloud/ai/foundation_models/v1/text_generation/text_generation_service.proto \
         yandex/cloud/ai/foundation_models/v1/text_common.proto
     ```

  1. В папке `output` создайте файл `test.py` с кодом запроса к модели:

     ```python
     # coding=utf8
     import argparse
     import grpc
     
     import yandex.cloud.ai.foundation_models.v1.text_common_pb2 as pb
     import yandex.cloud.ai.foundation_models.v1.text_generation.text_generation_service_pb2_grpc as service_pb_grpc
     import yandex.cloud.ai.foundation_models.v1.text_generation.text_generation_service_pb2 as service_pb
     
     def run(iam_token, folder_id, user_text):
         cred = grpc.ssl_channel_credentials()
         channel = grpc.secure_channel('llm.api.cloud.yandex.net:443', cred)
         stub = service_pb_grpc.TextGenerationServiceStub(channel)
     
         request = service_pb.CompletionRequest(
             model_uri=f"gpt://{folder_id}/yandexgpt",
             completion_options=pb.CompletionOptions(
                 max_tokens={"value": 2000}, 
                 temperature={"value": 0.5}
             ),
         )
         message_system = request.messages.add()
         message_system.role = "system"
         message_system.text = "Исправь ошибки в тексте."
     
         message_user = request.messages.add()
         message_user.role = "user"
         message_user.text = user_text
     
         it = stub.Completion(request, metadata=(
             ('authorization', f'Bearer {iam_token}'),
         ))
         for response in it:
             for alternative in response.alternatives:
                 print (alternative.message.text)
     
     if __name__ == '__main__':
         parser = argparse.ArgumentParser()
         parser.add_argument("--iam_token", required=True, help="IAM token")
         parser.add_argument("--folder_id", required=True, help="Folder id")
         parser.add_argument("--user_text", required=True, help="User text")
         args = parser.parse_args()
         run(args.iam_token, args.folder_id, args.user_text)
     ```

  1. Запустите файл `test.py`, подставив значение [IAM-токена](../../../iam/concepts/authorization/iam-token.md) и [идентификатор каталога](../../../resource-manager/operations/folder/get-id.md):

     ```bash
     export IAM_TOKEN=<IAM-токен>
     export FOLDER_ID=<идентификатор_каталога>
     export TEXT='Ашипки саме сибя ни исрпвят'
     python output/test.py \
       --iam_token ${IAM_TOKEN} \
       --folder_id ${FOLDER_ID} \
       --user_text "${TEXT}"
     ```

     {% cut "Результат:" %}

     ```text
     Ошибки сами себя не исправят.
     ```

     {% endcut %}

{% endlist %}

### Потоковый запрос через gRPC API {#stream}

{% list tabs group=programming_language %}

- SDK {#sdk}

  При использовании метода `run_stream` сервер будет присылать не только финальный результат генерации текста, но и промежуточные результаты. Каждый промежуточный ответ содержит весь результат генерации, доступный на текущий момент. До получения финального ответа результаты генерации могут меняться с приходом новых сообщений.

  Работа метода `run_stream` максимально наглядна при создании и обработке больших текстов.

  1. Создайте файл `generate-text.py` и добавьте в него следующий код:

      {% include [yandexgpt-stream-sdk](../../../_includes/foundation-models/examples/yandexgpt-stream-sdk.md) %}

      Где:

      {% include [the-messages-parameter](../../../_includes/foundation-models/yandexgpt/the-messages-parameter.md) %}

      {% include [sdk-code-legend](../../../_includes/foundation-models/examples/sdk-code-legend.md) %}

      {% include [yandexgpt-model-version-sdk-info](../../../_includes/foundation-models/yandexgpt/yandexgpt-model-version-sdk-info.md) %}

  1. Выполните созданный файл:

      ```bash
      python3 generate-text.py
      ```

      Результат:

      ```text
      Alternative(role='assistant', text='О', status=<AlternativeStatus.PARTIAL: 1>)
      Alternative(role='assistant', text='Ошибки сами себя не исправят.', status=<AlternativeStatus.FINAL: 3>)
      ```

- Python {#python}

  При включенном параметре `stream` сервер будет присылать не только финальный результат генерации текста, но и промежуточные результаты. Каждый промежуточный ответ содержит весь результат генерации, доступный на текущий момент. До получения финального ответа результаты генерации могут меняться с приходом новых сообщений. 

  Работа параметра `stream` максимально наглядна при создании и обработке больших текстов.

  {% note warning %}

  Параметр `stream` недоступен для [асинхронного режима](async-request.md) работы модели.

  {% endnote %}

  Сгенерируйте код интерфейса клиента gRPC, как описано в [инструкции](#request-grpc). На шаге 6 создайте файл `test.py`, содержащий код обращения к модели:

  ```python
  # coding=utf8
  import argparse
  import grpc

  import yandex.cloud.ai.foundation_models.v1.text_common_pb2 as pb
  import yandex.cloud.ai.foundation_models.v1.text_generation.text_generation_service_pb2_grpc as service_pb_grpc
  import yandex.cloud.ai.foundation_models.v1.text_generation.text_generation_service_pb2 as service_pb

  def run(iam_token, folder_id, user_text):
      cred = grpc.ssl_channel_credentials()
      channel = grpc.secure_channel('llm.api.cloud.yandex.net:443', cred)
      stub = service_pb_grpc.TextGenerationServiceStub(channel)

      request = service_pb.CompletionRequest(
          model_uri=f"gpt://{folder_id}/yandexgpt",
          completion_options=pb.CompletionOptions(
              max_tokens={"value": 2000},
              temperature={"value": 0.5},
              stream=True
              ),
      )
      message_system = request.messages.add()
      message_system.role = "system"
      message_system.text = "Исправь ошибки в тексте."
      
      message_user = request.messages.add()
      message_user.role = "user"
      message_user.text = user_text
      
      it = stub.Completion(request, metadata=(
          ('authorization', f'Bearer {iam_token}'),
      ))
          
      for response in it:
          print(response)

  if __name__ == '__main__':
      parser = argparse.ArgumentParser()
      parser.add_argument("--iam_token", required=True, help="IAM token")
      parser.add_argument("--folder_id", required=True, help="Folder id")
      parser.add_argument("--user_text", required=True, help="User text")
      args = parser.parse_args()
      run(args.iam_token, args.folder_id, args.user_text)
  ```

  {% cut "Результат:" %}

  ```text
  alternatives {
    message {
      role: "assistant"
      text: "О"
    }
    status: ALTERNATIVE_STATUS_PARTIAL
  }
  usage {
    input_text_tokens: 32
    completion_tokens: 1
    total_tokens: 33
  }
  model_version: "09.02.2025"

  alternatives {
    message {
      role: "assistant"
      text: "Ошибки сами себя не исправят."
    }
    status: ALTERNATIVE_STATUS_FINAL
  }
  usage {
    input_text_tokens: 32
    completion_tokens: 8
    total_tokens: 40
  }
  model_version: "09.02.2025"
  ```

  {% endcut %}

{% endlist %}

#### См. также {#see-also}

* [{#T}](../../concepts/yandexgpt/index.md)
* Примеры работы с {{ ml-sdk-name }} на [GitHub](https://github.com/yandex-cloud/yandex-cloud-ml-sdk/tree/master/examples/sync/completions)
