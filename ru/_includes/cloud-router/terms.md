## Терминология {{ cr-name }}

* **Гибридное облако (Hybrid Cloud)** — вид ИТ-инфраструктуры, который объединяет внутренние ИТ-ресурсы компании, с одной стороны, и ресурсы в публичных облаках — с другой.

* **On-Prem** — площадка клиента, на которой расположены его ИТ-ресурсы и сетевое оборудование для подключения к {{ yandex-cloud }} с помощью сервиса [{{ interconnect-name }}](../../interconnect/concepts/index.md).

* [Облачная сеть (VPC, Network)](../../vpc/concepts/network.md) — ресурс сервиса {{ vpc-name }}. Облачная сеть предназначена для организации сетевой связности во всех [зонах доступности](../../overview/concepts/geo-scope.md) одного региона.

* [Подсеть (Subnet)](../../vpc/concepts/network.md#subnet) — ресурс сервиса {{ vpc-name }}. Принадлежит облачной сети. У каждой подсети есть IP-префикс (CIDR). Каждая подсеть всегда привязана только к одной [зоне доступности](../../overview/concepts/geo-scope.md). К подсети подключаются облачные ресурсы.

* [Routing Instance (RI)](../../cloud-router/concepts/routing-instance.md) — это ресурс, который позволяет создать определенную сетевую топологию, в которую могут входить:
  * IP-префиксы [подсетей](../../vpc/concepts/network.md#subnet) одной или нескольких [сетей](../../vpc/concepts/network.md).
  * [Приватные соединения](../../interconnect/concepts/priv-con.md). Количество приватных соединений зависит от вида сетевой топологии.

* `Маршрутизация` — это возможность задать направление для передачи сетевого трафика, используя ресурсы сервисов [{{ cr-name }}](../../cloud-router/concepts/index.md) и [{{ interconnect-name }}](../../interconnect/concepts/index.md).

* **VPC Stitching** — один из механизмов сервиса `Cloud Router`, который обеспечивает сетевую связность между [облачными сетями (VPC)](../../vpc/concepts/network.md), которые расположены в каталогах одного или разных облаков в границах одной [облачной организации](../../organization/concepts/organization.md).

* **Склеивающие анонсы** - это агрегирующие маршруты, реализующие механизм `VPC-Stitching`, которые используются при создании отдельных видов сетевых топологий для настройки [приватных соединений](../../interconnect/concepts/priv-con.md). Имеют региональный характер - объединяют IP префиксы подсетей во всех [зонах доступности](../../overview/concepts/geo-scope.md). Склеивающие анонсы вычисляются клиентом и добавляются в конфигурацию [приватного соединения](../../interconnect/concepts/priv-con.md) самостоятельно. Автоматического вычисления склеивающих анонсов и добавления их в конфигурацию приватного соединения на стороне {{ yandex-cloud }} не происходит. Склеивающие анонсы отображаются в поле `ipv4_static_routes` API {{ interconnect-name }} для [приватных соединений](../../interconnect/concepts/priv-con.md). 

