---
title: Удалить правило из группы безопасности
description: Следуя данной инструкции, вы сможете удалить правило из группы безопасности.
---

# Удалить правило из группы безопасности

{% list tabs group=instructions %}

- Консоль управления {#console}

  Чтобы удалить [правило](../concepts/security-groups.md#security-groups-structure) из [группы безопасности](../concepts/security-groups.md):

  1. В [консоли управления]({{ link-console-main }}) перейдите в каталог, где требуется изменить группу безопасности.
  1. В списке сервисов выберите **{{ ui-key.yacloud.iam.folder.dashboard.label_vpc }}**.
  1. На панели слева выберите ![image](../../_assets/console-icons/shield.svg) **{{ ui-key.yacloud.vpc.label_security-groups }}**.
  1. Нажмите значок ![image](../../_assets/console-icons/ellipsis.svg) в строке группы, в которой требуется удалить правило, и выберите **{{ ui-key.yacloud.common.edit }}**.
  1. В блоке **{{ ui-key.yacloud.vpc.network.security-groups.forms.label_section-rules }}** нажмите значок ![image](../../_assets/console-icons/ellipsis.svg) в строке правила, которое требуется удалить.
  1. В открывшемся меню нажмите кнопку **{{ ui-key.yacloud.common.delete }}**.
  1. В открывшемся окне нажмите кнопку **{{ ui-key.yacloud.common.delete }}**.

- CLI {#cli}

  Чтобы удалить [правило](../concepts/security-groups.md#security-groups-structure) из [группы](../concepts/security-groups.md):

  1. Узнайте имя или идентификатор группы, которую требуется изменить:

     ```
     yc vpc security-groups list
     ```
     Результат:
     ```
     +----------------------+---------------------------------+------------------------------------+----------------------+
     |          ID          |              NAME               |          DESCRIPTION               |      NETWORK-ID      |
     +----------------------+---------------------------------+------------------------------------+----------------------+
     | enp9bmjge93b******** | default-sg-enp509crtquf******** | Default security group for network | enp509crtquf******** |
     | enp9rs9u4h6j******** | sg-1                            |                                    | enp509crtquf******** |
     | enp9d8m73d1c******** | sg-2                            |                                    | enp509crtquf******** |
     +----------------------+---------------------------------+------------------------------------+----------------------+
     ```
  1. Получите список правил в группе безопасности, указав ее имя или идентификатор:

     ```
     yc vpc security-groups get <имя_или_идентификатор_группы>
     ```
     Результат:
     ```
     id: enp8rs9i4h6j********
     folder_id: b1gaus8l79li********
     created_at: "2022-06-24T15:46:31Z"
     name: sg-1
     network_id: enp559cr9quf********
     status: ACTIVE
     rules:
       - id: enpbbmv8ici********
         description: SSH
         direction: INGRESS
         ports:
           from_port: "22"
           to_port: "22"
         protocol_name: TCP
         protocol_number: "6"
         cidr_blocks:
           v4_cidr_blocks:
             - 0.0.0.0/0
     ...
     ```

  1. Чтобы удалить правило, укажите в команде его идентификатор:

     ```
     yc vpc security-group update-rules <имя_или_идентификатор_группы> --delete-rule-id <идентификатор_правила>
     ```
     Результат:
     ```
     done (12s)
     id: enp8rs9i4h6j********
     folder_id: b1gaus8l79li********
     created_at: "2022-06-24T15:46:31Z"
     name: sg-1
     network_id: enp559cr9quf********
     status: ACTIVE
     rules:
     ...
     ```

- {{ TF }} {#tf}

  {% include [terraform-definition](../../_tutorials/_tutorials_includes/terraform-definition.md) %}

  {% include [terraform-install](../../_includes/terraform-install.md) %}

  Чтобы удалить [правило](../concepts/security-groups.md#security-groups-structure) из [группы безопасности](../concepts/security-groups.md), созданное с помощью {{ TF }}:

  1. Откройте файл конфигурации {{ TF }} и удалите блок `ingress` или `egress` в описании группы безопасности:

     {% cut "Пример описания группы безопасности с правилами в конфигурации {{ TF }}" %}

     ```hcl
     ...
     resource "yandex_vpc_security_group" "test-sg" {
       name        = "Test security group"
       description = "Description for security group"
       network_id  = "${yandex_vpc_network.lab-net.id}"

       ingress {
         protocol       = "TCP"
         description    = "Rule description 1"
         v4_cidr_blocks = ["10.0.1.0/24", "10.0.2.0/24"]
         port           = 8080
       }

       egress {
         protocol       = "ANY"
         description    = "Rule description 2"
         v4_cidr_blocks = ["10.0.1.0/24", "10.0.2.0/24"]
         from_port      = 8090
         to_port        = 8099
       }
     }
     ...
     ```

     {% endcut %}

  1. В командной строке перейдите в папку, где расположен файл конфигурации {{ TF }}.

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

     Проверить изменение группы безопасности можно в [консоли управления]({{ link-console-main }}) или с помощью команды [CLI](../../cli/quickstart.md):

     ```
     yc vpc security-group get <имя_группы_безопасности>
     ```

- API {#api}

  Чтобы удалить [правило](../concepts/security-groups.md#security-groups-structure) из [группы безопасности](../concepts/security-groups.md), воспользуйтесь методом REST API [updateRules](../api-ref/SecurityGroup/updateRules.md) для ресурса [SecurityGroup](../api-ref/SecurityGroup/index.md) или вызовом gRPC API [SecurityGroupService/UpdateRules](../api-ref/grpc/SecurityGroup/updateRules.md) и передайте в запросе:

  * Идентификатор группы безопасности, из которой будут удалены правила, в параметре `securityGroupId`.
  * Идентификаторы правил, которые будут удалены, в массиве `deletionRuleIds[]`.

  {% include [get-security-group-id](../../_includes/vpc/get-security-group-id.md) %}

  {% include [get-catalog-id](../../_includes/get-catalog-id.md) %}

{% endlist %}
