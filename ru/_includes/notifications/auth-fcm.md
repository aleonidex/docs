Вы можете аутентифицироваться на мобильной платформе FCM через API:
* _HTTP v1 API_ — понадобится **{{ ui-key.yacloud.cns.field_gcm-file }}**. С помощью ключа генерируются временные токены OAuth 2.0 для аутентификации запросов в FCM HTTP v1 API. Можно получить в консоли управления Google Cloud.
* _Legacy API_ — понадобится **{{ ui-key.yacloud.cns.field_gcm-api-key }}** (server key). Можно получить в консоли управления Firebase.

{% note info %}

С июня 2024 года Legacy API [не поддерживается FCM](https://firebase.google.com/docs/cloud-messaging/migrate-v1).

{% endnote %}

Подробнее см. в [документации Firebase](https://firebase.google.com/docs/cloud-messaging/android/client).