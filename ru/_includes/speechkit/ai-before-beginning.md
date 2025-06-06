Для работы с API {{ speechkit-name }} пройдите аутентификацию. Ее способ зависит от типа аккаунта:

{% list tabs group=authentication %}

- Аккаунт на Яндексе или федеративный аккаунт {#yandex-account}

  1. Получите IAM-токен для [аккаунта на Яндексе](../../iam/operations/iam-token/create.md) или [федеративного аккаунта](../../iam/operations/iam-token/create-for-federation.md).
  1. Получите [идентификатор каталога](../../resource-manager/operations/folder/get-id.md), на который у вашего аккаунта есть роли `{{ roles-speechkit-stt }}`, `{{ roles-speechkit-tts }}` или выше.
  1. {% include [iam-token-usage](../iam-token-usage-speechkit-v3.md) %}

- Сервисный аккаунт {#service-account}

  {{ speechkit-name }} поддерживает два способа аутентификации с сервисным аккаунтом:

   * С помощью [IAM-токена](../../iam/concepts/authorization/iam-token.md):

      1. [Получите IAM-токен](../../iam/operations/iam-token/create-for-sa.md).
      1. Полученный IAM-токен передавайте в заголовке `Authorization` в следующем формате:

          ```yaml
          Authorization: Bearer <IAM-токен>
          ```

   * С помощью [API-ключей](../../iam/concepts/authorization/api-key).

     {% include [api-keys-disclaimer](../iam/api-keys-disclaimer.md) %}

      1. [Получите API-ключ](../../iam/operations/authentication/manage-api-keys.md#create-api-key).
      1. Полученный API-ключ передавайте в заголовке `Authorization` в следующем формате:

          ```yaml
          Authorization: Api-Key <API-ключ>
          ```

   Не указывайте в запросах идентификатор каталога — сервис использует каталог, в котором был создан сервисный аккаунт.

{% endlist %}
