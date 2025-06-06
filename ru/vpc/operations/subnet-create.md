---
title: Создать подсеть
description: Следуя данной инструкции, вы сможете создать подсеть.
---

# Создать подсеть


[Подсети](../concepts/network.md#subnet) создаются в облачных сетях. Созданная подсеть размещается в одной из зон доступности. К подсети можно подключить ресурсы из той же зоны, где находится подсеть.

{% list tabs group=instructions %}

- Консоль управления {#console}

  1. В [консоли управления]({{ link-console-main }}) перейдите в каталог, где требуется создать подсеть.
  1. В списке сервисов выберите **{{ ui-key.yacloud.iam.folder.dashboard.label_vpc }}**.
  1. На панели слева выберите ![subnets](../../_assets/console-icons/nodes-right.svg) **{{ ui-key.yacloud.vpc.switch_networks }}**.
  1. Справа сверху нажмите **{{ ui-key.yacloud.common.create }}**.
  1. В поле **{{ ui-key.yacloud.vpc.subnetworks.create.field_name }}** укажите название подсети. Требования к названию:

     {% include [name-format](../../_includes/name-format.md) %}

  1. (Опционально) В поле **{{ ui-key.yacloud.vpc.subnetworks.create.field_description }}** добавьте описание.
  1. В поле **{{ ui-key.yacloud.vpc.subnetworks.create.field_zone }}** выберите зону доступности из выпадающего списка.
  1. В поле **{{ ui-key.yacloud.vpc.subnetworks.create.field_network }}** укажите облачную сеть. Она должна быть создана заранее.
  1. В поле **{{ ui-key.yacloud.vpc.subnetworks.create.field_ip }}** введите IP-адрес и маску подсети. 
     Подробнее про диапазоны IP-адресов в подсетях читайте в разделе [Облачные сети и подсети](../concepts/network.md). 
     Если нужно указать еще один или несколько CIDR, нажмите **{{ ui-key.yacloud.vpc.subnetworks.create.button_add-cidr }}**.
  1. (Опционально) Задайте **{{ ui-key.yacloud.vpc.subnetworks.create.section_dhcp-options }}**. Для этого:
      1. В поле **{{ ui-key.yacloud.vpc.subnetworks.create.field_domain-name }}** укажите домен DNS для поиска неквалифицированных имен.
      1. В поле **{{ ui-key.yacloud.vpc.subnetworks.create.field_domain-name-servers }}** нажмите **{{ ui-key.yacloud.vpc.subnetworks.create.button_add-domain-name-server }}** и укажите адрес вашего [DNS-сервера](../../glossary/dns.md#dns-server). Можно указать несколько DNS-серверов.
      1. В поле **{{ ui-key.yacloud.vpc.subnetworks.create.field_ntp-servers }}** нажмите **{{ ui-key.yacloud.vpc.subnetworks.create.button_add-ntp-server }}** и укажите адрес вашего NTP-сервера. Можно указать несколько NTP-серверов.

  1. Нажмите **{{ ui-key.yacloud.vpc.subnetworks.create.button_create }}**.

- CLI {#cli}

  {% include [include](../../_includes/cli-install.md) %}

  {% include [default-catalogue](../../_includes/default-catalogue.md) %}

  Чтобы создать подсеть:

  1. Посмотрите описание команды CLI для создания подсети:

      ```
      yc vpc subnet create --help
      ```

  1. Получите список облачных сетей в требуемом каталоге:

      ```
      yc vpc network list --folder-id b1g6ci08ma55********
      ```

      Результат:

      ```
      +----------------------+----------------+
      |          ID          |      NAME      |
      +----------------------+----------------+
      | enpavfmgapum******** | test-network-1 |
      | enplom7a98s1******** | default        |
      +----------------------+----------------+
      ```

  1. Выберите `NAME` или `ID` требуемой облачной сети. Создайте подсеть в каталоге по умолчанию:

      ```
      yc vpc subnet create \
        --name test-subnet-1 \
        --description "My test subnet" \
        --network-id enplom7a98s1******** \
        --zone {{ region-id }}-a \
        --range 192.168.0.0/24
      ```

      Где:

      * `--network-id` — идентификатор облачной сети. Также можно выбрать облачную сеть, указав ее имя с помощью параметра `--network-name`. При создании подсети указывается имя облачной сети, в которой создается подсеть, и CIDR.
      * `--zone` — зона доступности, в которой создается подсеть. Если параметр не указан, подсеть будет создана в зоне доступности по умолчанию.
      * `--range` — список внутренних IPv4-адресов, определенных для данной подсети. Например, `10.0.0.0/22` или `192.168.0.0/16`. Адреса должны быть уникальными внутри сети. Минимальный размер подсети — /28, а максимальный размер подсети — /16. Поддерживается только IPv4.

      Требования к названию подсети:

      {% include [name-format](../../_includes/name-format.md) %}

      ```
      yc vpc subnet create \
        --name test-subnet-1 \
        --description "My test subnet" \
        --network-name test-network-1 \
        --zone {{ region-id }}-a \
        --range 192.168.0.0/24
      ```

      Параметры `--name` и `--description` необязательны — можно создать подсеть без имени и описания и обращаться к ней по идентификатору.

  1. Получите список всех подсетей в каталоге по умолчанию:

      ```
      yc vpc subnet list
      ```

      Результат:

      ```
      +----------------------+-----------------------+------------------------+
      |          ID          |         NAME          | ... |       RANGE      |
      +----------------------+-----------------------+------------------------+
      ...
      | e2l0psbfoloe******** | test-subnet-1         | ... | [192.168.0.0/24] |
      ...
      +----------------------+-----------------------+-----+------------------+
      ```

      Получите тот же список с большим количеством деталей в формате YAML:

      ```
      yc vpc subnet list --format yaml
      ```

      Результат:
      
      ```
      ...

      - id: e2l0psbfoloe********
        folder_id: b1g6ci08ma55********
        created_at: "2018-10-24T12:25:58Z"
        name: test-subnet-1
        description: My test subnet
        network_id: enplom7a98s1********
        zone_id: {{ region-id }}-a
        v4_cidr_blocks:
        - 192.168.0.0/24

      ...
      ```

- {{ TF }} {#tf}

  {% include [terraform-definition](../../_tutorials/_tutorials_includes/terraform-definition.md) %}

  {% include [terraform-install](../../_includes/terraform-install.md) %}

  1. Опишите в конфигурационном файле параметры подсети:

     * `name` — имя подсети. Требования к имени:

        {% include [name-format](../../_includes/name-format.md) %}

     * `description` — описание подсети.
     * `v4_cidr_blocks` — список IPv4-адресов, откуда или куда будет поступать трафик. Например, `10.0.0.0/22` или `192.168.0.0/16`. Адреса должны быть уникальными внутри сети. Минимальный размер подсети — `/28`, а максимальный размер подсети — `/16`. Поддерживается только IPv4.
     * `zone` — [зона доступности](../../overview/concepts/geo-scope.md).
     * `network_id` — идентификатор сети, в которой создается подсеть.

     Пример структуры конфигурационного файла:

     ```hcl
     resource "yandex_vpc_subnet" "lab-subnet-a" {
       name           = "<имя_подсети>"
	   description    = "<описание_подсети>"
       v4_cidr_blocks = ["<IPv4-адрес>"]
       zone           = "<зона_доступности>"
       network_id     = "<идентификатор_сети>"
     }
     ```

     Чтобы добавить, изменить или удалить подсеть, используйте ресурс `yandex_vpc_subnet` с указанием на сеть в поле `network_id` (см. [пример](#examples)).

     Более подробную информацию о параметрах ресурса `yandex_vpc_subnet` в {{ TF }} см. в [документации провайдера]({{ tf-provider-resources-link }}/vpc_subnet).

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
        yc vpc subnet list
        ```

- API {#api}

  Чтобы создать подсеть, воспользуйтесь методом REST API [create](../api-ref/Subnet/create.md) для ресурса [Subnet](../api-ref/Subnet/index.md) или вызовом gRPC API [SubnetService/Create](../api-ref/grpc/Subnet/create.md) и передайте в запросе:

    * Идентификатор каталога, в котором будет размещена подсеть, в параметре `folderId`.
    * Идентификатор сети, в которой будет размещена подсеть, в параметре `networkId`.
    * Идентификатор зоны доступности, в которой будет размещена подсеть, в параметре `zoneId`.
    * Список внутренних IPv4-адресов, определенных для данной подсети, в массиве `v4CidrBlocks[]`. Например, `10.0.0.0/22` или `192.168.0.0/16`. Адреса должны быть уникальными внутри сети. Минимальный размер подсети — `/28`, а максимальный размер подсети — `/16`. Поддерживается только IPv4.

  {% include [get-subnet-id](../../_includes/vpc/get-subnet-id.md) %}

  {% include [get-catalog-id](../../_includes/get-catalog-id.md) %}

{% endlist %}

## Примеры {#examples}

{% list tabs group=instructions %}

- CLI {#cli}

  Создайте подсеть с именем и описанием в выбранном каталоге:

    ```
    yc vpc subnet create \
      --name test-subnet-1 \
      --description "My test subnet" \
      --folder-id b1g6ci08ma55******** \
      --network-id enplom7a98s1******** \
      --zone {{ region-id }}-a \
      --range 192.168.0.0/24
    ```

    Создайте подсеть с настройками DHCP:
    ```
    yc vpc subnet create \
      --name test-subnet-1 \
      --description "My test subnet" \
      --folder-id b1g6ci08ma55******** \
      --network-id enplom7a98s1******** \
      --zone {{ region-id }}-a \
      --range 192.168.0.0/24 \
      --domain-name test.domain \
      --domain-name-server 192.168.0.100 \
      --ntp-server 192.168.0.101
    ```

- {{ TF }} {#tf}

  1. Опишите в конфигурационном файле параметры ресурса `yandex_vpc_subnet`:

     ```hcl
     resource "yandex_vpc_network" "lab-net" {
       name        = "network-1"
       description = "My first network"
     }

     resource "yandex_vpc_subnet" "lab-subnet-a" {
       name           = "subnet-1"
       description    = "My first subnet"
       v4_cidr_blocks = ["10.2.0.0/16"]
       zone           = "{{ region-id }}-a"
       network_id     = "${yandex_vpc_network.lab-net.id}"
     }
     ```

     Более подробную информацию о параметрах ресурсов в {{ TF }} см. в [документации провайдера]({{ tf-provider-resources-link }}/vpc_subnet).

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
        yc vpc subnet list
        ```

{% endlist %}
