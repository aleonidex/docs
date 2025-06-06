---
title: Настройка прав доступа к асимметричной ключевой паре шифрования
description: Следуя данной инструкции, вы сможете назначить роли на асимметричную ключевую пару шифрования.
---

# Настройка прав доступа к асимметричной ключевой паре шифрования

Вы можете предоставить пользователю, сервисному аккаунту или группе пользователей доступ к [асимметричной ключевой паре шифрования](../concepts/asymmetric-encryption-key.md). Для этого назначьте [роли](../../iam/concepts/access-control/roles.md) на ключевую пару. Чтобы выбрать нужные, [узнайте](../security/index.md#roles-list), какие роли действуют в сервисе.

## Назначить роль {#add-access-binding}

{% list tabs group=instructions %}

- Консоль управления {#console}

   1. В [консоли управления]({{ link-console-main }}) выберите каталог, в котором находится асимметричная ключевая пара шифрования.
   1. В списке сервисов выберите **{{ ui-key.yacloud.iam.folder.dashboard.label_kms }}**.
   1. На панели слева выберите ![image](../../_assets/kms/asymmetric-key.svg) **{{ ui-key.yacloud.kms.switch_asymmetric-keys }}**.
   1. На вкладке **{{ ui-key.yacloud.kms.asymmetric-key.form.label_encryption }}** нажмите на имя нужной ключевой пары.
   1. Перейдите в раздел ![image](../../_assets/console-icons/persons.svg) **{{ ui-key.yacloud.common.resource-acl.label_access-bindings }}** и нажмите кнопку **{{ ui-key.yacloud.common.resource-acl.button_new-bindings }}**.
   1. Выберите группу, пользователя или сервисный аккаунт, которым нужно предоставить доступ к ключевой паре.
   1. Нажмите кнопку ![image](../../_assets/console-icons/plus.svg) **{{ ui-key.yacloud_components.acl.button.add-role }}** и выберите необходимые роли.
   1. Нажмите кнопку **{{ ui-key.yacloud_components.acl.action.apply }}**.

- CLI {#cli}

   {% include [cli-install](../../_includes/cli-install.md) %}

   {% include [default-catalogue](../../_includes/default-catalogue.md) %}

   Чтобы назначить роль на асимметричную ключевую пару шифрования:

   1. Посмотрите описание команды CLI для назначения роли:

      ```bash
      yc kms asymmetric-encryption-key add-access-binding --help
      ```

   1. Получите список асимметричных ключевых пар вместе с их идентификаторами:

      ```bash
      yc kms asymmetric-encryption-key list
      ```

   1. Получите [идентификатор пользователя](../../iam/operations/users/get.md), [сервисного аккаунта](../../iam/operations/sa/get-id.md) или группы пользователей, которым назначаете роль.
   1. С помощью одной из команд ниже назначьте роль:

      * Пользователю:

         ```bash
         yc kms asymmetric-encryption-key add-access-binding \
           --id <идентификатор_ключевой_пары> \
           --role <роль> \
           --user-account-id <идентификатор_пользователя>
         ```

      * Федеративному пользователю:

         ```bash
         yc kms asymmetric-encryption-key add-access-binding \
           --id <идентификатор_ключевой_пары> \
           --role <роль> \
           --subject federatedUser:<идентификатор_пользователя>
         ```

      * Сервисному аккаунту:

         ```bash
         yc kms asymmetric-encryption-key add-access-binding \
           --id <идентификатор_ключевой_пары> \
           --role <роль> \
           --service-account-id <идентификатор_сервисного_аккаунта>
         ```

      * Группе пользователей:

         ```bash
         yc kms asymmetric-encryption-key add-access-binding \
           --id <идентификатор_ключевой_пары> \
           --role <роль> \
           --subject group:<идентификатор_группы>
         ```

- {{ TF }} {#tf}

   {% include [terraform-definition](../../_tutorials/_tutorials_includes/terraform-definition.md) %}

   {% include [terraform-install](../../_includes/terraform-install.md) %}

   Чтобы назначить роль на асимметричную ключевую пару шифрования с помощью {{ TF }}:

   1. Опишите в конфигурационном файле {{ TF }} параметры ресурсов, которые необходимо создать:

       ```hcl
       resource "yandex_kms_asymmetric_encryption_key_iam_member" "key-viewers" {
         asymmetric_encryption_key_id = "<идентификатор_ключевой_пары>"
         role                         = "<роль_1>"
         members                      = ["<тип_субъекта>:<идентификатор_субъекта>"]
       }
       ```

       Где:

       * `asymmetric_encryption_key_id` — идентификатор ассиметричной ключевой пары шифрования.
       * `role` — назначаемая [роль](../security/index.md#roles-list).
       * `members` — список типов и идентификаторов [субъектов](../../iam/concepts/access-control/index.md#subject), которым назначается роль. Указывается в формате `userAccount:<идентификатор_пользователя>` или `serviceAccount:<идентификатор_сервисного_аккаунта>`.

       Подробнее о параметрах ресурса `yandex_kms_asymmetric_encryption_key_iam_member` см. в [документации провайдера]({{ tf-provider-resources-link }}/kms_asymmetric_encryption_key_iam_member).

   1. Создайте ресурсы:

       {% include [terraform-validate-plan-apply](../../_tutorials/_tutorials_includes/terraform-validate-plan-apply.md) %}

       {{ TF }} создаст все требуемые ресурсы. Проверить появление ресурсов можно с помощью команды [CLI](../../cli/):

       ```bash
       yc kms asymmetric-key list-access-bindings <идентификатор_ключа>
       ```

- API {#api}

   Воспользуйтесь методом [UpdateAccessBindings](../asymmetricencryption/api-ref/AsymmetricEncryptionKey/updateAccessBindings.md) для ресурса [AsymmetricEncryptionKey](../asymmetricencryption/api-ref/AsymmetricEncryptionKey/index.md) или вызовом gRPC API [AsymmetricEncryptionKeyService/UpdateAccessBindings](../asymmetricencryption/api-ref/grpc/AsymmetricEncryptionKey/updateAccessBindings.md) и передайте в запросе:

   * Значение `ADD` в параметре `access_binding_deltas[].action`, чтобы добавить роль.
   * Роль в параметре `access_binding_deltas[].access_binding.role_id`.
   * Идентификатор субъекта, на кого назначается роль, в параметре `access_binding_deltas[].access_binding.subject.id`.
   * Тип субъекта, на кого назначается роль, в параметре `access_binding_deltas[].access_binding.subject.type`.

{% endlist %}

## Назначить несколько ролей {#set-access-bindings}

{% list tabs group=instructions %}

- Консоль управления {#console}

   1. В [консоли управления]({{ link-console-main }}) выберите каталог, в котором находится асимметричная ключевая пара шифрования.
   1. В списке сервисов выберите **{{ ui-key.yacloud.iam.folder.dashboard.label_kms }}**.
   1. На панели слева выберите ![image](../../_assets/kms/asymmetric-key.svg) **{{ ui-key.yacloud.kms.switch_asymmetric-keys }}**.
   1. На вкладке **{{ ui-key.yacloud.kms.asymmetric-key.form.label_encryption }}** нажмите на имя нужной ключевой пары.
   1. Перейдите в раздел ![image](../../_assets/console-icons/persons.svg) **{{ ui-key.yacloud.common.resource-acl.label_access-bindings }}** и нажмите кнопку **{{ ui-key.yacloud.common.resource-acl.button_new-bindings }}**.
   1. Выберите группу, пользователя или сервисный аккаунт, которым нужно предоставить доступ к ключевой паре.
   1. Нажмите кнопку ![image](../../_assets/console-icons/plus.svg) **{{ ui-key.yacloud_components.acl.button.add-role }}** и выберите необходимые роли.
   1. Нажмите кнопку **{{ ui-key.yacloud_components.acl.action.apply }}**.

- CLI {#cli}

   {% include [set-access-bindings-cli](../../_includes/iam/set-access-bindings-cli.md) %}

   {% include [cli-install](../../_includes/cli-install.md) %}

   {% include [default-catalogue](../../_includes/default-catalogue.md) %}

   Чтобы назначить несколько ролей на асимметричную ключевую пару шифрования:

   1. Убедитесь, что на ключевую пару не назначены роли, которые вы не хотите потерять:

      ```bash
      yc kms asymmetric-encryption-key list-access-bindings \
        --id <идентификатор_ключевой_пары>
      ```

   1. Посмотрите описание команды CLI для назначения ролей:

      ```bash
      yc kms asymmetric-encryption-key set-access-bindings --help
      ```

   1. Получите список асимметричных ключевых пар вместе с их идентификаторами:

      ```bash
      yc kms asymmetric-encryption-key list
      ```

   1. Получите [идентификатор пользователя](../../iam/operations/users/get.md), [сервисного аккаунта](../../iam/operations/sa/get-id.md) или группы пользователей, которым назначаете роли.
   1. С помощью одной из команд ниже назначьте роли:

      * Пользователю с аккаунтом на Яндексе:

         ```bash
         yc kms asymmetric-encryption-key set-access-bindings \
           --id <идентификатор_ключевой_пары> \
           --access-binding role=<роль>,user-account-id=<идентификатор_пользователя>
         ```

      * Федеративному пользователю:

         ```bash
         yc kms asymmetric-encryption-key set-access-bindings \
           --id <идентификатор_ключевой_пары> \
           --access-binding role=<роль>,subject=federatedUser:<идентификатор_пользователя>
         ```

      * Сервисному аккаунту:

         ```bash
         yc kms asymmetric-encryption-key set-access-bindings \
           --id <идентификатор_ключевой_пары> \
           --access-binding role=<роль>,service-account-id=<идентификатор_сервисного_аккаунта>
         ```

      * Группе пользователей:

         ```bash
         yc kms asymmetric-encryption-key set-access-bindings \
           --id <идентификатор_ключевой_пары> \
           --access-binding role=<роль>,subject=group:<идентификатор_группы>
         ```

      Для каждой роли передайте отдельный параметр `--access-binding`. Пример:

      ```bash
      yc kms asymmetric-encryption-key set-access-bindings \
        --id <идентификатор_ключевой_пары> \
        --access-binding role=<роль1>,service-account-id=<идентификатор_сервисного_аккаунта> \
        --access-binding role=<роль2>,service-account-id=<идентификатор_сервисного_аккаунта> \
        --access-binding role=<роль3>,service-account-id=<идентификатор_сервисного_аккаунта>
      ```

- {{ TF }} {#tf}

   {% include [terraform-definition](../../_tutorials/_tutorials_includes/terraform-definition.md) %}

   {% include [terraform-install](../../_includes/terraform-install.md) %}

   Чтобы назначить несколько ролей на асимметричную ключевую пару шифрования с помощью {{ TF }}:

   1. Опишите в конфигурационном файле {{ TF }} параметры ресурсов, которые необходимо создать:

       ```hcl
       # Роль 1
       resource "yandex_kms_asymmetric_encryption_key_iam_member" "key-viewers" {
         asymmetric_encryption_key_id = "<идентификатор_ключевой_пары>"
         role                         = "<роль_1>"
         members                      = ["<тип_субъекта>:<идентификатор_субъекта>"]
       }

       # Роль 2
       resource "yandex_kms_asymmetric_encryption_key_iam_member" "key-editors" {
         asymmetric_encryption_key_id = "<идентификатор_ключевой_пары>"
         role                         = "<роль_2>"
         members                      = ["<тип_субъекта>:<идентификатор_субъекта>"]
       }
       ```

       Где:

       * `asymmetric_encryption_key_id` — идентификатор ассиметричной ключевой пары шифрования.
       * `role` — назначаемая [роль](../security/index.md#roles-list).
       * `members` — список типов и идентификаторов [субъектов](../../iam/concepts/access-control/index.md#subject), которым назначается роль. Указывается в формате `userAccount:<идентификатор_пользователя>` или `serviceAccount:<идентификатор_сервисного_аккаунта>`.

       Подробнее о параметрах ресурса `yandex_kms_asymmetric_encryption_key_iam_member` см. в [документации провайдера]({{ tf-provider-resources-link }}/kms_asymmetric_encryption_key_iam_member).

   1. Создайте ресурсы:

       {% include [terraform-validate-plan-apply](../../_tutorials/_tutorials_includes/terraform-validate-plan-apply.md) %}

       {{ TF }} создаст все требуемые ресурсы. Проверить появление ресурсов можно с помощью команды [CLI](../../cli/):

       ```bash
       yc kms asymmetric-key list-access-bindings <идентификатор_ключа>
       ```

- API {#api}

   {% include [set-access-bindings-api](../../_includes/iam/set-access-bindings-api.md) %}

   Воспользуйтесь методом [SetAccessBindings](../asymmetricencryption/api-ref/AsymmetricEncryptionKey/setAccessBindings.md) для ресурса [AsymmetricEncryptionKey](../asymmetricencryption/api-ref/AsymmetricEncryptionKey/index.md) или вызовом gRPC API [AsymmetricEncryptionKeyService/SetAccessBindings](../asymmetricencryption/api-ref/grpc/AsymmetricEncryptionKey/setAccessBindings.md). Передайте в запросе массив из объектов, каждый из которых соответствует отдельной роли и содержит следующие данные:

   * Роль в параметре `access_bindings[].role_id`.
   * Идентификатор субъекта, на кого назначаются роли, в параметре `access_bindings[].subject.id`.
   * Тип субъекта, на кого назначаются роли, в параметре `access_bindings[].subject.type`.

{% endlist %}
