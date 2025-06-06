---
title: Как создать виртуальную машину Linux на базе публичного образа
description: Следуя данной инструкции, вы сможете создать виртуальную машину с операционной системой Linux.
---

# Создать виртуальную машину из публичного образа Linux

{% include [role-note](../../../_includes/compute/role-note.md) %}

{% list tabs group=instructions %}

- Консоль управления {#console}

  
  <iframe width="640" height="360" src="https://runtime.strm.yandex.ru/player/video/vplv2quzqpa76crq4el5?autoplay=0&mute=0" allow="autoplay; fullscreen; picture-in-picture; encrypted-media" frameborder="0" scrolling="no"></iframe>

  [Смотреть видео на YouTube](https://www.youtube.com/watch?v=PN3b26KXb78).



  {% include [create-instance-via-console-linux](../../_includes_service/create-instance-via-console-linux.md) %}

- CLI {#cli}

  {% include [cli-install](../../../_includes/cli-install.md) %}

  {% include [default-catalogue](../../../_includes/default-catalogue.md) %}

  1. Посмотрите описание команды [CLI](../../../cli/) для создания ВМ:

     ```bash
     yc compute instance create --help
     ```

  1. [Подготовьте](../vm-connect/ssh.md#creating-ssh-keys) пару ключей (открытый и закрытый) для [SSH-доступа](../../../glossary/ssh-keygen.md) на ВМ.
  1. Выберите один из публичных [образов](../images-with-pre-installed-software/get-list.md) [{{ marketplace-full-name }}](../../../marketplace/) на базе операционной системы Linux (например, [CentOS 7](/marketplace/products/yc/centos-7)).

     {% include [standard-images](../../../_includes/standard-images.md) %}

  1. Выберите [подсеть](../../../vpc/concepts/network.md#subnet):

     ```bash
     yc vpc subnet list
     ```

     Результат:

     ```text
     +----------------------+---------------------------+----------------------+----------------+-------------------+-----------------+
     |          ID          |           NAME            |      NETWORK ID      | ROUTE TABLE ID |       ZONE        |      RANGE      |
     +----------------------+---------------------------+----------------------+----------------+-------------------+-----------------+
     | e9bnlm18l70a******** |   default-{{ region-id }}-a   | enpe3m3fa00u******** |                |   {{ region-id }}-a   | [10.128.0.0/24] |
     +----------------------+---------------------------+----------------------+----------------+-------------------+-----------------+
     ```

  1. Создайте ВМ в [каталоге](../../../resource-manager/concepts/resources-hierarchy.md#folder) по умолчанию:

     ```bash
     yc compute instance create \
       --name first-instance \
       --zone {{ region-id }}-a \
       --network-interface subnet-name=default-{{ region-id }}-a,nat-ip-version=ipv4 \
       --create-boot-disk image-folder-id=standard-images,image-family=centos-7,kms-key-id=<идентификатор_ключа>,auto-delete=true \
       --ssh-key ~/.ssh/id_ed25519.pub
     ```

     Где:
     * `--name` — имя ВМ. Требования к имени:

       {% include [name-format](../../../_includes/name-format.md) %}

       {% include [name-fqdn](../../../_includes/compute/name-fqdn.md) %}

     * `--zone` — [зона доступности](../../../overview/concepts/geo-scope.md), которая соответствует выбранной подсети.
     * `--network-interface` — настройки [сетевого интерфейса](../../concepts/network.md) ВМ:
         * `subnet-name` — имя выбранной подсети.
         * `nat-ip-version=ipv4` – [публичный IP-адрес](../../../vpc/concepts/address.md#public-addresses). Чтобы создать ВМ без публичного IP-адреса, исключите параметр.

         {% include [add-several-net-interfaces-notice-cli](../../../_includes/compute/add-several-net-interfaces-notice-cli.md) %}

     * `--create-boot-disk` — настройки загрузочного диска ВМ:
         * `auto-delete` — настройка автоудаления загрузочного диска вместе с ВМ. См. [{#T}](../../concepts/disk.md#autodelete-disks).
         * `image-family` — [семейство образов](../../concepts/image.md#family), например, `centos-7`. Эта опция позволит установить последнюю версию ОС из указанного семейства.
         * `kms-key-id` — идентификатор [симметричного ключа {{ kms-short-name }}](../../../kms/concepts/key.md) для создания зашифрованного загрузочного диска. Необязательный параметр.

           {% include [encryption-role](../../../_includes/compute/encryption-role.md) %}

           {% include [encryption-disable-warning](../../../_includes/compute/encryption-disable-warning.md) %}

           {% include [encryption-keys-note](../../../_includes/compute/encryption-keys-note.md) %}

     * `--ssh-key` — путь к файлу с [публичным SSH-ключом](../vm-connect/ssh.md#creating-ssh-keys). Для этого ключа на ВМ будет автоматически создан пользователь `yc-user`.

         {% include [ssh-note](../../../_includes/compute/ssh-note.md) %}

         Если вы хотите добавить на ВМ одновременно нескольких пользователей с SSH-ключами, [задайте](../../concepts/vm-metadata.md#how-to-send-metadata) данные этих пользователей с помощью параметра `--metadata-from-file`. С помощью метаданных вы также можете [установить дополнительное ПО](./create-with-cloud-init-scripts.md) на ВМ при ее создании.

  {% include [ip-fqdn-connection](../../../_includes/ip-fqdn-connection.md) %}

- {{ TF }} {#tf}

  {% include [terraform-install](../../../_includes/terraform-install.md) %}

  1. Опишите в конфигурационном файле параметры ресурсов, которые необходимо создать:

     ```hcl
     resource "yandex_compute_disk" "boot-disk" {
       name     = "<имя_диска>"
       type     = "<тип_диска>"
       zone     = "<зона_доступности>"
       size     = "<размер_диска>"
       image_id = "<идентификатор_образа>"
     }

     resource "yandex_compute_instance" "vm-1" {
       name                      = "linux-vm"
       allow_stopping_for_update = true
       platform_id               = "standard-v3"
       zone                      = "<зона_доступности>"

       resources {
         cores  = "<количество_ядер_vCPU>"
         memory = "<объем_RAM_ГБ>"
       }

       boot_disk {
         auto_delete = true
         disk_id = yandex_compute_disk.boot-disk.id
       }

       network_interface {
         subnet_id = "${yandex_vpc_subnet.subnet-1.id}"
         nat       = true
       }

       metadata = {
         ssh-keys = "<имя_пользователя>:<содержимое_SSH-ключа>"
       }
     }

     resource "yandex_vpc_network" "network-1" {
       name = "network1"
     }

     resource "yandex_vpc_subnet" "subnet-1" {
       name           = "subnet1"
       zone           = "<зона_доступности>"
       v4_cidr_blocks = ["192.168.10.0/24"]
       network_id     = "${yandex_vpc_network.network-1.id}"
     }
     ```

     Где:
     * `yandex_compute_disk` — описание загрузочного [диска](../../concepts/disk.md):
       * `name` — имя диска.
       * `type` — тип создаваемого диска.
       * `zone` — [зона доступности](../../../overview/concepts/geo-scope.md), в которой будет находиться диск.
       * `size` — размер диска в ГБ.
       * `image_id` — идентификатор образа для ВМ. Вы можете получить идентификатор образа из [списка публичных образов](../images-with-pre-installed-software/get-list.md).

         {% include [id-info](../../../_includes/compute/id-info.md) %}

     * `yandex_compute_instance` — описание ВМ:
       * `name` — имя ВМ.
       * {% include [terraform-allow-stopping](../../../_includes/compute/terraform-allow-stopping.md) %}
       * `platform_id` — [платформа](../../concepts/vm-platforms.md).
       * `zone` — зона доступности, в которой будет находиться ВМ.
       * `resources` — количество ядер vCPU и объем RAM, доступные ВМ. Значения должны соответствовать выбранной [платформе](../../concepts/vm-platforms.md).
       * `boot_disk` — настройки загрузочного диска. Укажите идентификатор диска.
       * `auto_delete` — настройка автоудаления загрузочного диска вместе с ВМ. См. [{#T}](../../concepts/disk.md#autodelete-disks).
       * `network_interface` — настройки [сетевого интерфейса](../../concepts/network.md) ВМ. Укажите идентификатор выбранной [подсети](../../../vpc/concepts/network.md#subnet). Чтобы автоматически назначить ВМ [публичный IP-адрес](../../../vpc/concepts/address.md#public-addresses), укажите `nat = true`.

           {% include [add-several-net-interfaces-notice-tf](../../../_includes/compute/add-several-net-interfaces-notice-tf.md) %}

       * `metadata` — в метаданных необходимо передать открытый SSH-ключ для доступа на ВМ. Подробнее в разделе [{#T}](../../concepts/vm-metadata.md).

           Если вы хотите добавить на ВМ одновременно нескольких пользователей с SSH-ключами, [задайте](../../concepts/vm-metadata.md#how-to-send-metadata) данные этих пользователей в файле и передайте в блоке `metadata`. С помощью метаданных вы также можете [установить дополнительное ПО](./create-with-cloud-init-scripts.md) на ВМ при ее создании.
     * `yandex_vpc_network` — описание облачной сети.
     * `yandex_vpc_subnet` — описание подсети, к которой будет подключена ВМ.

     {% note info %}

     Если у вас уже есть подходящие ресурсы (облачная сеть и подсеть), описывать их повторно не нужно. Используйте их имена и идентификаторы в соответствующих параметрах.

     {% endnote %}

     Более подробную информацию о ресурсах, которые вы можете создать с помощью {{ TF }}, см. в [документации провайдера]({{ tf-provider-link }}).
  1. Создайте ресурсы:

     {% include [terraform-validate-plan-apply](../../../_tutorials/_tutorials_includes/terraform-validate-plan-apply.md) %}

     После этого в указанном каталоге будут созданы все требуемые ресурсы. Проверить появление ресурсов и их настройки можно в [консоли управления]({{ link-console-main }}).

  {% include [ip-fqdn-connection](../../../_includes/ip-fqdn-connection.md) %}

- API {#api}

  Создайте ВМ с помощью метода REST API [create](../../api-ref/Instance/create.md) для ресурса [Instance](../../api-ref/Instance/):
  1. [Подготовьте](../vm-connect/ssh.md#creating-ssh-keys) пару ключей (открытый и закрытый) для [SSH-доступа](../../../glossary/ssh-keygen.md) на ВМ.
  1. Получите [{{ iam-short-name }}-токен](../../../iam/concepts/authorization/iam-token.md), используемый для аутентификации в примерах:
     * [Инструкция](../../../iam/operations/iam-token/create.md) для пользователя с аккаунтом на Яндексе.
     * [Инструкция](../../../iam/operations/iam-token/create-for-sa.md) для [сервисного аккаунта](../../../iam/concepts/users/service-accounts.md).
     * [Инструкция](../../../iam/operations/iam-token/create-for-federation.md) для федеративного аккаунта.
  1. [Получите идентификатор](../../../resource-manager/operations/folder/get-id.md) [каталога](../../../resource-manager/concepts/resources-hierarchy.md#folder).
  1. Получите информацию об [образе](../../concepts/image.md), из которого надо создать ВМ (идентификатор образа и минимальный размер [диска](../../concepts/disk.md)):
     * Если вы знаете [семейство образа](../../concepts/image.md#family), получите информации о последнем образе в этом семействе:

       ```bash
       export IAM_TOKEN=CggaATEVAgA...
       export FAMILY=ubuntu-1804
       curl \
         --header "Authorization: Bearer ${IAM_TOKEN}" \
         "https://compute.{{ api-host }}/compute/v1/images:latestByFamily?folderId=standard-images&family=${FAMILY}"
       ```

     * Вы можете получить информацию об образе из [списка публичных образов](../images-with-pre-installed-software/get-list.md).
  1. Получите идентификатор [подсети](../../../vpc/concepts/network.md#subnet) и идентификатор [зоны доступности](../../../overview/concepts/geo-scope.md). В запросе укажите идентификатор каталога, в котором создана подсеть:

     ```bash
     export IAM_TOKEN=CggaATEVAgA...
     export FOLDER_ID=b1gvmob95yys********
     curl \
       --header "Authorization: Bearer ${IAM_TOKEN}" \
       "https://vpc.{{ api-host }}/vpc/v1/subnets?folderId=${FOLDER_ID}"
     ```

     Результат:

     ```
     {
       "subnets": [
        {
          "v4CidrBlocks": [
            "10.130.0.0/24"
          ],
          "id": "b0c6n43ftldh********",
          "folderId": "b1gvmob95yys********",
          "createdAt": "2018-09-23T12:15:00Z",
          "name": "default-{{ region-id }}-a",
          "description": "Auto-created default subnet for zone {{ region-id }}-a",
          "networkId": "enpe3m3faglu********",
          "zoneId": "{{ region-id }}-a"
        },
        ...
       ]
     }
     ```

  1. Создайте файл с телом запроса на создание ВМ, например, `body.json`:

     ```json
     {
       "folderId": "b1gvmob95yys********",
       "name": "instance-demo-no-pwauth",
       "zoneId": "{{ region-id }}-a",
       "platformId": "standard-v3",
       "resourcesSpec": {
         "memory": "2147483648",
         "cores": "2"
       },
       "metadata": {
         "user-data": "#cloud-config\nusers:\n  - name: user\n    groups: sudo\n    shell: /bin/bash\n    sudo: 'ALL=(ALL) NOPASSWD:ALL'\n    ssh_authorized_keys:\n      - ssh-ed25519 AAAAB3N... user@example.com"
       },
       "bootDiskSpec": {
         "autoDelete": true,
         "diskSpec": {
           "size": "8589934592",
           "imageId": "fd8rc75pn12f********"
         }
       },
       "networkInterfaceSpecs": [
         {
           "subnetId": "b0c6n43ftldh********",
           "primaryV4AddressSpec": {
             "oneToOneNatSpec": {
               "ipVersion": "IPV4"
             }
           }
         }
       ]
     }
     ```

     Где:
     * `folderId` — идентификатор каталога.
     * `name` — имя, которое будет присвоено ВМ при создании.
     * `zoneId` — зона доступности, которая соответствует выбранной подсети.
     * `platformId` — [платформа](../../concepts/vm-platforms.md).
     * `resourceSpec` — ресурсы, доступные ВМ. Значения должны соответствовать выбранной платформе.
     * `metadata` — в метаданных необходимо передать открытый ключ для SSH-доступа на ВМ. Подробнее в разделе [{#T}](../../concepts/vm-metadata.md).
     * `bootDiskSpec` — настройки загрузочного диска. Укажите идентификатор выбранного образа и размер диска.
     * `autoDelete` — настройка автоудаления загрузочного диска вместе с ВМ. См. [{#T}](../../concepts/disk.md#autodelete-disks).

       {% include [id-info](../../../_includes/compute/id-info.md) %}

       Размер диска должен быть не меньше минимального размера диска, указанного в информации об образе.
     * `networkInterfaceSpecs` — настройки [сетевого интерфейса](../../concepts/network.md) ВМ:
       * `subnetId` — идентификатор выбранной подсети.
       * `primaryV4AddressSpec` — IP-адрес, который будет присвоен ВМ. Чтобы добавить [публичный IP-адрес](../../../vpc/concepts/address.md#public-addresses) ВМ, укажите:

         ```json
         "primaryV4AddressSpec": {
           "oneToOneNatSpec": {
             "ipVersion": "IPV4"
           }
         }
         ```
       Если вы хотите добавить на ВМ несколько [сетевых интерфейсов](../../concepts/network.md), передайте в `networkInterfaceSpecs` массив с необходимым количеством объектов с настройками сетевых интерфейсов. На одну ВМ можно добавить до восьми сетевых интерфейсов.

     Подробнее про формат тела запроса в [справочнике API](../../api-ref/Instance/create.md).
  1. Создайте ВМ:

     ```bash
     export IAM_TOKEN=CggaATEVAgA...
     curl \
       --request POST \
       --header "Content-Type: application/json" \
       --header "Authorization: Bearer ${IAM_TOKEN}" \
       --data '@body.json' \
       https://compute.{{ api-host }}/compute/v1/instances
     ```

  {% include [ip-fqdn-connection](../../../_includes/ip-fqdn-connection.md) %}

{% endlist %}

#### См. также {#see-also}

* [{#T}](../vm-connect/ssh.md).