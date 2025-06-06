# Целевые группы

Целевая группа L7-балансировщика — это набор IP-адресов ВМ, на которых запущены ваши сетевые приложения. Чтобы балансировщик распределял запросы между ВМ целевой группы, ее нужно указать в конфигурации [группы бэкендов](backend-group.md).

Основными целевыми ресурсами в группе являются ВМ, которые подключены к [подсетям](../../vpc/concepts/network.md#subnet) {{ vpc-name }}. Чтобы L7-балансировщик работал корректно, подсети целевых ресурсов должны находиться в одной [облачной сети](../../vpc/concepts/network.md#network).

Также вы можете добавлять в целевую группу ресурсы, которые размещены вне {{ vpc-name }}, например в вашем ЦОД, подключенном к {{ yandex-cloud }} через [{{ interconnect-name }}](../../interconnect/index.yaml). Адреса ресурсов должны входить в [частные диапазоны из RFC 1918](https://datatracker.ietf.org/doc/html/rfc1918#section-3). Подробнее см. [Подсети](../../vpc/concepts/network.md#subnet).


Целевые группы можно создавать:

* [Вручную](../operations/target-group-create.md). В этом случае при изменении внутреннего IP-адреса ВМ, например при ее переносе в другую подсеть, целевую группу нужно обновить самостоятельно — удалить и заново добавить в нее измененную ВМ.
* [Через {{ ig-name }}](../../compute/operations/instance-groups/create-with-load-balancer.md). При изменении конфигурации ВМ {{ ig-name }} будет автоматически поддерживать актуальность IP-адресов машин в целевой группе. Если целевая группа подключена к группе бэкендов, {{ ig-name }} будет следить и за результатами проверок состояния, которые отправляет L7-балансировщик.

{% include [target-group-deletion-restriction](../../_includes/application-load-balancer/target-group-deletion-restriction.md) %}

## Примеры использования {#examples}

* [{#T}](../tutorials/virtual-hosting.md)
* [{#T}](../tutorials/alb-with-ddos-protection/console.md)
* [{#T}](../tutorials/migration-from-nlb-to-alb/nlb-as-target-resource-alb/index.md)
* [{#T}](../tutorials/migration-from-nlb-to-alb/nlb-with-target-resource-group-vm/index.md)
* [{#T}](../tutorials/tls-termination/console.md)
* [{#T}](../tutorials/logging.md)
* [{#T}](../tutorials/application-load-balancer-website/index.md)