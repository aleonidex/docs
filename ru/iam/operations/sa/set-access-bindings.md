# Настройка прав доступа к сервисному аккаунту

Этот раздел про назначение [роли](../../concepts/access-control/roles.md) на [сервисный аккаунт](../../concepts/users/service-accounts.md) как на ресурс. Чтобы выдать роль сервисному аккаунту на другой ресурс, воспользуйтесь инструкцией [{#T}](assign-role-for-sa.md).

{% note info %}

Чтобы назначить роль на сервисный аккаунт, нужна роль `iam.serviceAccounts.admin`.

{% endnote %}

## Назначить роль на сервисный аккаунт {#assign-role-to-sa}

{% list tabs group=instructions %}

- Консоль управления {#console}

    1. В [консоли управления]({{ link-console-main }}) перейдите в каталог, которому принадлежит сервисный аккаунт.
    1. В списке сервисов выберите **{{ ui-key.yacloud.iam.folder.dashboard.label_iam }}**.
    1. На панели слева выберите ![FaceRobot](../../../_assets/console-icons/face-robot.svg) **{{ ui-key.yacloud.iam.label_service-accounts }}** и выберите нужный сервисный аккаунт.
    1. Перейдите на вкладку **{{ ui-key.yacloud.common.resource-acl.label_access-bindings }}**.
    1. Нажмите кнопку **{{ ui-key.yacloud.common.resource-acl.button_new-bindings }}**.
    1. В окне **{{ ui-key.yacloud_components.acl.label.title }}** нажмите кнопку **{{ ui-key.yacloud_components.acl.action.select-subject }}**.
    1. Выберите пользователя из списка или воспользуйтесь поиском по пользователям.
    1. Нажмите кнопку **{{ ui-key.yacloud_components.acl.button.add-role }}**.
    1. Выберите роль.
    1. Нажмите кнопку **{{ ui-key.yacloud_components.acl.action.apply }}**.

- CLI {#cli}

    {% include [cli-install](../../../_includes/cli-install.md) %}

    {% include [default-catalogue](../../../_includes/default-catalogue.md) %}

    1. Посмотрите описание команды для назначения роли на сервисный аккаунт как на ресурс:

        ```bash
        yc iam service-account add-access-binding --help
        ```

    1. Выберите сервисный аккаунт, например `my-robot`:

        ```bash
        yc iam service-account list
        ```

        Результат:

        ```
        +----------------------+----------+------------------+
        |          ID          |   NAME   |   DESCRIPTION    |
        +----------------------+----------+------------------+
        | ajebqtreob2d******** | test-sa  | test-description |
        | aje6o61dvog2******** | my-robot |                  |
        +----------------------+----------+------------------+
        ```

    1. Выберите [роль](../../concepts/access-control/roles.md):

        ```bash
        yc iam role list
        ```

        Результат:

        ```
        +--------------------------------+-------------+
        |               ID               | DESCRIPTION |
        +--------------------------------+-------------+
        | admin                          |             |
        | compute.images.user            |             |
        | editor                         |             |
        | ...                            |             |
        +--------------------------------+-------------+
        ```

    1. Узнайте ID пользователя по логину или адресу электронной почты. Чтобы назначить роль не пользователю, а сервисному аккаунту или группе пользователей, воспользуйтесь [примерами](#examples) ниже.

        ```bash
        yc iam user-account get test-user
        ```

        Результат:

        ```
        id: gfei8n54hmfh********
        yandex_passport_user_account:
            login: test-user
            default_email: test-user@yandex.ru
        ```

    1. Назначьте пользователю `test-user` роль `editor` на сервисный аккаунт `my-robot`. В субъекте укажите тип `userAccount` и ID пользователя:

        ```bash
        yc iam service-account add-access-binding my-robot \
          --role editor \
          --subject userAccount:gfei8n54hmfh********
        ```

- {{ TF }} {#tf}

    {% include [terraform-install](../../../_includes/terraform-install.md) %}

    1. Добавьте в конфигурационный файл параметры ресурса и укажите роль пользователей для доступа к сервисному аккаунту:

       * `service_account_id` — идентификатор сервисного аккаунта, к которому нужно настроить доступ.
       * `role` — назначаемая роль. Обязательный параметр.
       * `members` — список пользователей и сервисных аккаунтов, которым назначается роль. Указывается в виде `userAccount:<идентификатор_пользователя>` или `serviceAccount:<идентификатор_сервисного_аккаунта>`. Обязательный параметр.

       Пример структуры конфигурационного файла:

       ```
       resource "yandex_iam_service_account_iam_binding" "admin-account-iam" {
         service_account_id = "<идентификатор_сервисного_аккаунта>"
         role               = "<роль>"
         members            = [
           "federatedUser:<идентификатор_пользователя>",
         ]
       }
       ```

       Более подробную информацию о ресурсах, которые вы можете создать с помощью {{ TF }}, см. в [документации провайдера]({{ tf-provider-resources-link }}/iam_service_account_iam_binding).

    1. Проверьте корректность конфигурационных файлов.

       1. В командной строке перейдите в папку, где вы создали конфигурационный файл.
       1. Выполните проверку с помощью команды:

          ```
          terraform plan
          ```

       Если конфигурация описана верно, в терминале отобразится список создаваемых ресурсов и их параметров. Если в конфигурации есть ошибки, {{ TF }} на них укажет.

    1. Разверните облачные ресурсы.

       1. Если в конфигурации нет ошибок, выполните команду:

          ```
          terraform apply
          ```

       1. Подтвердите создание ресурсов: введите в терминал слово `yes` и нажмите **Enter**.

       После этого в указанном каталоге будут созданы все требуемые ресурсы. Проверить создание ресурса можно в [консоли управления]({{ link-console-main }}) или с помощью команды [CLI](../../../cli/quickstart.md):

       ```
       yc resource-manager service-account list-access-bindings <имя_или_идентификатор_сервисного_аккаунта>
       ```

- API {#api}

    Воспользуйтесь методом REST API [updateAccessBindings](../../api-ref/ServiceAccount/updateAccessBindings.md) для ресурса [ServiceAccount](../../api-ref/ServiceAccount/index.md) или вызовом gRPC API [ServiceAccountService/UpdateAccessBindings](../../api-ref/grpc/ServiceAccount/updateAccessBindings.md). Вам понадобится ID сервисного аккаунта и ID пользователя, которому назначается роль на сервисный аккаунт.

    1. Узнайте ID сервисного аккаунта с помощью метода REST API [list](../../api-ref/ServiceAccount/list.md):

        ```bash
        curl \
          --header "Authorization: Bearer <IAM-токен>" \
          https://iam.{{ api-host }}/iam/v1/serviceAccounts?folderId=b1gvmob95yys********
        ```

        Результат:

        ```
        {
        "serviceAccounts": [
            {
            "id": "aje6o61dvog2********",
            "folderId": "b1gvmob95yys********",
            "createdAt": "2018-10-19T13:26:29Z",
            "name": "my-robot"
            }
            ...
        ]
        }
        ```

    1. Узнайте ID пользователя по логину с помощью метода REST API [getByLogin](../../api-ref/YandexPassportUserAccount/getByLogin.md):
        
        ```bash
        curl \
          --header "Authorization: Bearer <IAM-токен>" \
          https://iam.{{ api-host }}/iam/v1/yandexPassportUserAccounts:byLogin?login=test-user
        ```

        Результат:

        ```
        {
        "id": "gfei8n54hmfh********",
        "yandexPassportUserAccount": {
            "login": "test-user",
            "defaultEmail": "test-user@yandex.ru"
        }
        }
        ```

    1. Назначьте пользователю роль `editor` на сервисный аккаунт `my-robot`. В свойстве `action` укажите `ADD`, а в свойстве `subject` - тип `userAccount` и ID пользователя:

       ```bash
       curl \
         --request POST \
         --header 'Content-Type: application/json' \
         --header "Authorization: Bearer <IAM-токен>" \
         --data '{
         "accessBindingDeltas": [{
             "action": "ADD",
             "accessBinding": {
                 "roleId": "editor",
                 "subject": {
                     "id": "gfei8n54hmfh********",
                     "type": "userAccount"
         }}}]}' \
         https://iam.{{ api-host }}/iam/v1/serviceAccounts/aje6o61dvog2********:updateAccessBindings
        ```

{% endlist %}

## Примеры {#examples}

* [Назначить несколько ролей](#multiple-roles).
* [Настроить имперсонацию](#impersonation).
* [Настроить доступ сервисного аккаунта к другому сервисному аккаунту](#access-to-sa).

### Назначить несколько ролей {#multiple-roles}

{% list tabs group=instructions %}

- CLI {#cli}

    {% include [cli-install](../../../_includes/cli-install.md) %}

    Команда `add-access-binding` позволяет добавить только одну роль. Вы можете назначить несколько ролей с помощью команды `set-access-binding`.

    {% note alert %}

    Команда `set-access-binding` полностью перезаписывает права доступа к ресурсу! Все текущие роли на ресурс будут удалены.

    {% endnote %}

    1. Убедитесь, что на ресурс не назначены роли, которые вы не хотите потерять:
    
        ```bash
        yc iam service-account list-access-bindings my-robot
        ```

    1. Например, назначьте роль нескольким пользователям:

        ```bash
        yc iam service-account set-access-bindings my-robot \
          --access-binding role=editor,subject=userAccount:gfei8n54hmfh******** \
          --access-binding role=viewer,subject=userAccount:helj89sfj80a********
        ```

- {{ TF }} {#tf}

  {% include [terraform-install](../../../_includes/terraform-install.md) %}

  Чтобы назначить несколько ролей на сервисный аккаунт, созданный с помощью {{ TF }}:

  1. Добавьте в конфигурационный файл параметры ресурса и укажите роль пользователей для доступа к сервисному аккаунту:

       * `service_account_id` — идентификатор сервисного аккаунта, к которому нужно настроить доступ.
       * `role` — назначаемая роль. Обязательный параметр.

       {% note info %}

       Для каждой роли можно использовать только один ресурс `yandex_iam_service_account_iam_binding`.

       {% endnote %}

       * `members` — список пользователей и сервисных аккаунтов, которым назначается роль. Указывается в виде `userAccount:<идентификатор_пользователя>` или `serviceAccount:<идентификатор_сервисного_аккаунта>`. Обязательный параметр.

     {% cut "Пример назначения нескольких ролей на сервисный аккаунт с помощью {{ TF }}" %}

     ```hcl
     ...
     resource "yandex_iam_service_account_iam_binding" "admin-account-iam" {
       service_account_id = "aje82upckiqh********"
       role               = "admin"
       members = [
         "userAccount:aje82upckiqh********",
       ]
     }
     resource "yandex_iam_service_account_iam_binding" "admin-account-iam2" {
       service_account_id = "aje82upckiqh********"
       role               = "viewer"
       members = [
         "userAccount:aje82upckiqh********",
       ]
     }
     ...
     ```

     {% endcut %}

     Более подробную информацию о ресурсах, которые вы можете создать с помощью {{ TF }}, см. в [документации провайдера]({{ tf-provider-resources-link }}/iam_service_account_iam_binding).
 
  1. Проверьте конфигурацию командой:
     ```
     terraform validate
     ``` 

     Если конфигурация является корректной, появится сообщение:

     ```
     Success! The configuration is valid.
     ```

  1. Выполните команду:
     ```
     terraform plan
     ```

     В терминале будет выведен список ресурсов с параметрами. На этом этапе изменения не будут внесены. Если в конфигурации есть ошибки, {{ TF }} на них укажет.

  1. Примените изменения конфигурации:
     ```
     terraform apply
     ```

  1. Подтвердите изменения: введите в терминал слово `yes` и нажмите **Enter**.

     Проверить изменение каталога можно в [консоли управления]({{ link-console-main }}) или с помощью команды [CLI](../../../cli/quickstart.md):

     ```
     yc resource-manager service-account list-access-bindings <имя_или_идентификатор_сервисного_аккаунта>
     ```

- API {#api}

    Назначьте одному пользователю роль `editor`, а другому `viewer`:

    ```bash
    curl \
      --request POST \
      --header 'Content-Type: application/json' \
      --header "Authorization: Bearer <IAM-токен>" \
      --data '{
      "accessBindingDeltas": [{
          "action": "ADD",
          "accessBinding": {
              "roleId": "editor",
              "subject": {
                  "id": "gfei8n54hmfh********",
                  "type": "userAccount"
              }
          }
      },{
          "action": "ADD",
          "accessBinding": {
              "roleId": "viewer",
              "subject": {
                  "id": "helj89sfj80a********",
                  "type": "userAccount"
      }}}]}' \
      https://iam.{{ api-host }}/iam/v1/serviceAccounts/aje6o61dvog2********:updateAccessBindings
    ```

    Вы также можете назначать роли с помощью метода REST API [setAccessBindings](../../api-ref/ServiceAccount/setAccessBindings.md) для ресурса [ServiceAccount](../../api-ref/ServiceAccount/index.md) или вызовом gRPC API [ServiceAccountService/SetAccessBindings](../../api-ref/grpc/ServiceAccount/setAccessBindings.md).

    {% note alert %}

    Метод `setAccessBindings` полностью перезаписывает права доступа к ресурсу! Все текущие роли на ресурс будут удалены.

    {% endnote %}

    ```bash
    curl \
      --request POST \
      --header 'Content-Type: application/json' \
      --header "Authorization: Bearer <IAM-токен>" \
      --data '{
      "accessBindings": [{
          "roleId": "editor",
          "subject": { "id": "ajei8n54hmfh********", "type": "userAccount" }
      },{
          "roleId": "viewer",
          "subject": { "id": "helj89sfj80a********", "type": "userAccount" }
      }]}' \
      https://iam.{{ api-host }}/iam/v1/serviceAccounts/aje6o61dvog2********:setAccessBindings
    ```

{% endlist %}

### Настроить имперсонацию {#impersonation}

[Имперсонация](../../concepts/access-control/index.md#impersonation) позволяет пользователю выполнять действия от имени сервисного аккаунта с помощью параметра `--impersonate-service-account-id`. Для этого у сервисного аккаунта должны быть нужные права, а у пользователя — роль `iam.serviceAccounts.tokenCreator`.

{% list tabs group=instructions %}

- CLI {#cli}

  {% include [cli-install](../../../_includes/cli-install.md) %}

  1. Узнайте ID сервисного аккаунта, например, `test-sa`, которому вы хотите назначить роль. Чтобы узнать ID, получите список доступных сервисных аккаунтов (в профиле администратора):

      ```bash
      yc iam service-account list
      ```

      Результат:

      ```
      +----------------------+----------+------------------+
      |          ID          |   NAME   |   DESCRIPTION    |
      +----------------------+----------+------------------+
      | ajebqtreob2d******** | test-sa  | test-description |
      | aje6o61dvog2******** | my-robot |                  |
      +----------------------+----------+------------------+
      ```

  1. Назначьте сервисному аккаунту `test-sa` роль `viewer` на каталог `my-folder`. В типе субъекта укажите `serviceAccount`, а в значении — ID сервисного аккаунта (в профиле администратора):

      ```
      yc resource-manager folder add-access-binding my-folder \
        --role viewer \
        --subject serviceAccount:ajebqtreob2d********
      ```

  1. Получите ID пользователя и назначьте ему роль `iam.serviceAccounts.tokenCreator` на сервисный аккаунт `test-sa` (в профиле администратора):

      ```
      yc iam service-account add-access-binding test-sa \
        --role iam.serviceAccounts.tokenCreator \
        --subject userAccount:gfei8n54hmfh********
      ```

  1. Пользователь может выполнить команду от имени сервисного аккаунта `test-sa` с помощью параметра `--impersonate-service-account-id`.

      Например, пользователь может получить список виртуальных машин в каталоге `my-folder`:

      ```
      yc compute instance list --folder-name my-folder \
        --impersonate-service-account-id ajebqtreob2d********
      ```

      Также пользователь может получить [IAM-токен](../../concepts/authorization/iam-token.md) сервисного аккаунта `test-sa` для кратковременного доступа:

      ```
      yc iam create-token --impersonate-service-account-id ajebqtreob2d********
      ```

      Срок действия полученного токена закончится автоматически.

  1. Если доступ больше не нужен пользователю, отзовите роль у сервисного аккаунта (в профиле администратора):

      ```
      yc resource-manager folder remove-access-binding my-folder \
        --role viewer \
        --subject serviceAccount:ajebqtreob2d********
      ```
  1. Отзовите роль `iam.serviceAccounts.tokenCreator` у пользователя, получавшего права сервисного аккаунта:

      ```
      yc iam service-account remove-access-binding test-sa \
        --role iam.serviceAccounts.tokenCreator \
        --subject userAccount:gfei8n54hmfh********
      ```

{% endlist %}

### Настроить доступ сервисного аккаунта к другому сервисному аккаунту {#access-to-sa}

Разрешите сервисному аккаунту `test-sa` управлять сервисным аккаунтом `my-robot`:

{% list tabs group=instructions %}

- CLI {#cli}

  {% include [cli-install](../../../_includes/cli-install.md) %}

  1. Узнайте ID сервисного аккаунта `test-sa`, которому вы хотите назначить роль. Чтобы узнать ID, получите список доступных сервисных аккаунтов:

      ```bash
      yc iam service-account list
      ```

      Результат:

      ```
      +----------------------+----------+------------------+
      |          ID          |   NAME   |   DESCRIPTION    |
      +----------------------+----------+------------------+
      | ajebqtreob2d******** | test-sa  | test-description |
      | aje6o61dvog2******** | my-robot |                  |
      +----------------------+----------+------------------+
      ```

  1. Назначьте роль `editor` сервисному аккаунту `test-sa`, указав его ID. В типе субъекта укажите `serviceAccount`:

      ```bash
      yc iam service-account add-access-binding my-robot \
        --role editor \
        --subject serviceAccount:ajebqtreob2d********
      ```

- {{ TF }} {#tf}

  {% include [terraform-install](../../../_includes/terraform-install.md) %}

  Чтобы разрешить сервисному аккаунту `test-sa` управлять сервисным аккаунтом `my-robot`, созданным при помощи {{ TF }}:

    1. Добавьте в конфигурационный файл параметры ресурса и укажите роль пользователей для доступа к сервисному аккаунту:

       * `service_account_id` — идентификатор сервисного аккаунта, к которому нужно настроить доступ.
       * `role` — назначаемая роль. Обязательный параметр.
       * `members` — список пользователей и сервисных аккаунтов, которым назначается роль. Указывается в виде `userAccount:<идентификатор_пользователя>` или `serviceAccount:<идентификатор_сервисного_аккаунта>`. Обязательный параметр.

     {% cut "Пример разрешения сервисному аккаунту `test-sa` управлять сервисным аккаунтом `my-robot` с помощью {{ TF }}" %}

     ```hcl
     ...
     resource "yandex_iam_service_account_iam_binding" "admin-account-iam" {
       service_account_id = "aje82upckiqh********"
       role               = "admin"
       members = [
         "serviceAccount:aje82upckiqh********",
       ]
     }
     ...
     ```

     {% endcut %}

     Более подробную информацию о ресурсах, которые вы можете создать с помощью {{ TF }}, см. в [документации провайдера]({{ tf-provider-resources-link }}/iam_service_account_iam_binding).

  1. Проверьте конфигурацию командой:
     ```
     terraform validate
     ```

     Если конфигурация является корректной, появится сообщение:

     ```
     Success! The configuration is valid.
     ```

  1. Выполните команду:
     ```
     terraform plan
     ```

     В терминале будет выведен список ресурсов с параметрами. На этом этапе изменения не будут внесены. Если в конфигурации есть ошибки, {{ TF }} на них укажет.

  1. Примените изменения конфигурации:
     ```
     terraform apply
     ```

  1. Подтвердите изменения: введите в терминал слово `yes` и нажмите **Enter**.

     Проверить изменение каталога можно в [консоли управления]({{ link-console-main }}) или с помощью команды [CLI](../../../cli/quickstart.md):

     ```
     yc resource-manager service-account list-access-bindings <имя_или_идентификатор_сервисного_аккаунта>
     ```

- API {#api}

  1. Узнайте ID сервисного аккаунта `test-sa`, которому вы хотите назначить роль. Чтобы узнать ID, получите список доступных сервисных аккаунтов:

      ```bash
      curl \
        --header "Authorization: Bearer <IAM-токен>" \
        https://iam.{{ api-host }}/iam/v1/serviceAccounts?folderId=b1gvmob95yys********
      ```

      Результат:

      ```
      {
       "serviceAccounts": [
        {
         "id": "ajebqtreob2d********",
         "folderId": "b1gvmob95yys********",
         "createdAt": "2018-10-18T13:42:40Z",
         "name": "test-sa",
         "description": "test-description"
        },
        {
         "id": "aje6o61dvog2********",
         "folderId": "b1gvmob95yys********",
         "createdAt": "2018-10-15T18:01:25Z",
         "name": "my-robot"
        }
       ]
      }
      ```

  1. Назначьте сервисному аккаунту `test-sa` роль `editor` на другой сервисный аккаунт `my-robot`. В свойстве `subject` укажите тип `serviceAccount` и ID `test-sa`. В URL запроса в качестве ресурса укажите ID `my-robot`:

      ```bash
      curl \
        --request POST \
        --header 'Content-Type: application/json' \
        --header "Authorization: Bearer <IAM-токен>" \
        --data '{
        "accessBindingDeltas": [{
            "action": "ADD",
            "accessBinding": {
                "roleId": "editor",
                "subject": {
                    "id": "ajebqtreob2d********",
                    "type": "serviceAccount"
        }}}]}' \
        https://iam.{{ api-host }}/iam/v1/serviceAccounts/aje6o61dvog2********:updateAccessBindings
      ```

{% endlist %}
