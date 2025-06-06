# Создать облачную сеть

Помимо облачной сети в каталоге по умолчанию вы можете создавать дополнительные облачные сети в других каталогах.

{% list tabs group=instructions %}

- Консоль управления {#console}

  Чтобы создать [облачную сеть](../concepts/network.md):
  1. В [консоли управления]({{ link-console-main }}) перейдите в каталог, где требуется создать облачную сеть.
  1. В списке сервисов выберите **{{ ui-key.yacloud.iam.folder.dashboard.label_vpc }}**.
  1. В правом верхнем углу нажмите **{{ ui-key.yacloud.vpc.networks.button_create }}**.
  1. В поле **{{ ui-key.yacloud.vpc.networks.create.field_name }}** укажите имя сети. Требования к имени:

      {% include [name-format](../../_includes/name-format.md) %}
  
  1. (Опционально) В поле **{{ ui-key.yacloud.vpc.networks.create.field_description }}** добавьте описание сети.
  1. По умолчанию выбрана опция **{{ ui-key.yacloud.vpc.networks.create.field_is-default }}**. Если вы хотите [создать](subnet-create.md) подсети позднее, выключите опцию.
  1. Нажмите **{{ ui-key.yacloud.vpc.networks.button_create }}**.

- CLI {#cli}

  {% include [include](../../_includes/cli-install.md) %}

  {% include [default-catalogue](../../_includes/default-catalogue.md) %}

  Чтобы создать [облачную сеть](../concepts/network.md):
  1. Посмотрите описание команды CLI для создания облачной сети:

      ```
      yc vpc network create --help
      ```

  1. Создайте облачную сеть в каталоге по умолчанию:

      ```
      yc vpc network create \
        --name test-network-1 \
        --description "My test network"
      ```

      Параметры `--name` и `--description` необязательные — можно создать сеть без имени и описания и обращаться к ней по идентификатору.

      Требования к названию сети:

      {% include [name-format](../../_includes/name-format.md) %}

  1. Получите список всех облачных сетей в каталоге по умолчанию:

      ```
      yc vpc network list
      ```

      Результат:
      ```
      +----------------------+----------------+
      |          ID          |      NAME      |
      +----------------------+----------------+
      | enpiuvhhd4t8******** | test-network-1 |
      | enplom7a98s1******** | default        |
      +----------------------+----------------+
      ```

      Получите тот же список с большим количеством деталей в формате YAML:

      ```
      yc vpc network list --format yaml
      ```

      Результат:
      ```
      - id: enpiuvhhd4t8********
        folder_id: b1g6ci08ma55********
        created_at: "2018-10-23T11:12:51Z"
        name: test-network-1
        description: My first network
      - id: enplom7a98s1********
        folder_id: b1g6ci08ma55********
        created_at: "2018-09-24T08:23:00Z"
        name: default
        description: Auto-created default network
      ```

- {{ TF }} {#tf}

  {% include [terraform-definition](../../_tutorials/_tutorials_includes/terraform-definition.md) %}

  {% include [terraform-install](../../_includes/terraform-install.md) %}

  1. Опишите в конфигурационном файле параметры [облачной сети](../concepts/network.md):

     * `name` — имя облачной сети. Требования к имени:

        {% include [name-format](../../_includes/name-format.md) %}

     * `description` — описание облачной сети.
     * `labels` — [метки](../../resource-manager/concepts/labels.md) для облачной сети. Укажите пару ключ-значение.

     Пример структуры конфигурационного файла:

     ```hcl
     resource "yandex_vpc_network" "default" {
       name        = "<имя_сети>"
	   description = "<описание_сети>"
       labels = {
         tf-label    = "tf-label-value"
         empty-label = ""
       }
     }
     ```

     Более подробную информацию о параметрах ресурса `vpc_network` в {{ TF }} см. в [документации провайдера]({{ tf-provider-resources-link }}/vpc_network).

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

        После этого в указанном каталоге будут созданы все требуемые ресурсы. Проверить появление ресурсов и их настройки можно в [консоли управления]({{ link-console-main }}) или с помощью команд [CLI](../../cli/quickstart.md):

        ```
        yc vpc network list
        ```

- API {#api}

  Чтобы создать [облачную сеть](../concepts/network.md), воспользуйтесь методом REST API [create](../api-ref/Network/create.md) для ресурса [Network](../api-ref/Network/index.md) или вызовом gRPC API [NetworkService/Create](../api-ref/grpc/Network/create.md) и передайте в запросе:

    * Идентификатор каталога, в котором будет размещена сеть, в параметре `folderId`.
    * Имя новой сети в параметре `name`. Допустимая длина имени — от 3 до 63 символов, последний символ — не дефис.
    * Описание новой сети в параметре `description`. Максимальная длина описания — 256 символов.

  Параметры `name` и `description` необязательные — можно создать сеть без имени и описания и обращаться к ней по идентификатору.

  {% include [get-catalog-id](../../_includes/get-catalog-id.md) %}

{% endlist %}

## Примеры {#examples}

Создайте облачную сеть с именем и описанием в выбранном каталоге:

{% list tabs group=instructions %}

- CLI {#cli}

  ```
  yc vpc network create --name test-network-1 \
    --description "My test network" \
    --folder-id b1gnbfd11bq5********
  ```

- {{ TF }} {#tf}

  1. Опишите в конфигурационном файле параметры создаваемой облачной сети:

     ```hcl
     resource "yandex_vpc_network" "default" {
       name        = "network-1"
	   description = "My first network"
     }
     ```

     Более подробную информацию о параметрах ресурсов в {{ TF }} см. в [документации провайдера]({{ tf-provider-resources-link }}/vpc_network).

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

        После этого в указанном каталоге будут созданы все требуемые ресурсы. Проверить появление ресурсов и их настройки можно в [консоли управления]({{ link-console-main }}) или с помощью команд [CLI](../../cli/quickstart.md):

        ```
        yc vpc network list
        ```

{% endlist %}
