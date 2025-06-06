# Routing Instance

Объект `Routing Instance` (RI) — это основной ресурс в ресурсной модели сервиса [{{ cr-name }}](./index.md), предназначенный для создания сетевых топологий, в рамках которых облачные ресурсы в разных [сетях](../../vpc/concepts/network.md) будут иметь сетевую связность между собой.

## Общие правила и рекомендации {#cr-general}

При использовании сервисов [{{ cr-name }}](./index.md) и [{{ interconnect-name }}](../../interconnect/concepts/index.md) настоятельно рекомендуется следовать следующим правилам и рекомендациям:

1. Во всех сегментах сети (как со стороны облака, так и со стороны On-Prem), которые будут объединяться с помощью `Routing Instance (RI)`, должна использоваться согласованная и непересекающаяся IP-адресация. IP-префиксы [подсетей](../../vpc/concepts/network.md#subnet), которые добавляются в RI, должны быть согласованы между собой. Нельзя добавить два одинаковых IP-префикса в один RI. IP-адресация на стороне `On-Prem` должна быть согласована с IP-адресацией в виртуальных облачных сетях. Пересечение IP-адресации всегда будет приводить к нежелательным потерям IP-связности. **Тщательно планируйте IP-адресацию**.

1. Сетевое взаимодействие между [виртуальными сетями](../../vpc/concepts/network.md) организуется только в пределах одной облачной [организации](../../organization/concepts/organization.md). Все ресурсы сервисов {{ cr-name }} и {{ interconnect-name }}, а также виртуальные облачные сети, которые будут объединяться в сетевую топологию с помощью `RI`, должны размещаться в [облачных каталогах](../../resource-manager/concepts/resources-hierarchy.md#folder) в пределах одной облачной организации. При необходимости обеспечить сетевую связность между виртуальными облачными сетями, которые принадлежат разным облачным организациям рекомендуется мигрировать облака из нескольких организаций в одну.

    Если мигрировать облака в одну организацию не представляется возможным, необходимо получить документальные (бумажные) подтверждения о разрешении таких межорганизационных сетевых связей от всех владельцев этих облачных организаций. За подробностями о процессе подтверждения следует обращаться в [службу технической поддержки](../../support/overview.md) {{ yandex-cloud }}.

1. Прямой обмен маршрутной информацией между любыми [приватными соединениями](../../interconnect/concepts/priv-con.md) (PRC) в пределах `RI` не допускается.

1. IP-префиксы, которые анонсирует сетевое оборудование клиента, также попадают в `RI` и далее в виртуальные сети, которые подключены к этому `RI`. В настоящее время посмотреть эти IP-префиксы на уровне `RI` нельзя. Рекомендуется получать эту информацию от сетевого оборудования клиента. В отдельных случаях при решении проблем с маршрутизацией можно обратиться с запросом в [техническую поддержку]({{ link-console-support }}) для получения такой информации.

## Рекомендации по настройке приватных соединений {#prc} 

1. Всегда организовывайте два канала связи {{ interconnect-name }} через две [точки присутствия](../../interconnect/concepts/pops.md). Это поможет избежать отказов в работе ваших сервисов в случае отключения одного из каналов.

1. Для упрощения настройки отказоустойчивой BGP-маршрутизации на клиентских маршрутизаторах рекомендуется использовать одинаковый номер [BGP ASN](../../interconnect/concepts/priv-con.md#bgp-asn), если со стороны клиента используются несколько маршрутизаторов для подключения к услуге {{ interconnect-name }}. Допускается использовать разные номера **BGP ASN**, например, при организации подключений через операторов связи. Следует помнить, что настройка сетевого оборудования клиента и оператора связи лежит вне зоны ответственности {{ yandex-cloud }}.

1. Если у клиента два канала связи {{ interconnect-name }} подключаются к разным маршрутизаторам, то между этими клиентскими маршрутизаторами необходимо дополнительно установить взаимодействие (связность) по протоколу **iBGP**. В случае отсутствия такой связности при выходе из строя одного из каналов связи на одном из маршрутизаторов соседний маршрутизатор ничего не узнает об этом отказе, и трафик из отказавшего канала не будет перемаршрутизирован на оставшийся в работе канал связи средствами протокола BGP.

1. При необходимости распределения исходящего трафика из облачных сетей по нескольким каналам связи в {{ interconnect-name }} используйте на клиентских маршрутизаторах префиксы разной длины для BGP-анонсов:
    * длина префикса `/8` (короткий префикс) соответствует самому низкому приоритету маршрута;
    * длина префикса `/32` (длинный префикс) соответствует самому высокому приоритету маршрута.

    Подробнее о распределении трафика по нескольким каналам связи можно узнать в разделе [сценарии использования](../scenarios/index.md).

1. Для выбора канала связи для исходящего трафика из инфраструктуры клиента в направлении облачных сетей на клиентском маршрутизаторе можно использовать, например, стандартный BGP-атрибут `Local Preference`. Подробнее о распределении трафика по нескольким каналам связи можно узнать в разделе [сценарии использования](../scenarios/index.md).


## Ограничения при сетевом взаимодействии {#gaps}

1. Допускается совместное использование {{ interconnect-name }} и [NAT-шлюза](../../vpc/operations/create-nat-gateway.md), если клиентские маршрутизаторы не анонсируют маршрут «по умолчанию» `0.0.0.0/0` по протоколу BGP в направлении {{ yandex-cloud }}. В случае анонса маршрута по умолчанию `0.0.0.0/0` по протоколу BGP со стороны клиентских маршрутизаторов в направлении {{ yandex-cloud }} использование [NAT-шлюза](../../vpc/operations/create-nat-gateway.md) не допускается.

1. Сейчас в {{ yandex-cloud }} нет возможности распределять исходящий трафик из облачных подсетей в направлении инфраструктуры клиента на основе метода [BGP community](https://linkmeup.gitbook.io/sdsm/8.1.-ibgp/3.-atributy-bgp/4.-community/0.-teoriya).

{% note alert %}

Одновременно использовать одинаковые префиксы в таблице маршрутизации {{ vpc-short-name }} и в анонсах клиентского маршрутизатора нельзя.

{% endnote %}


## Взаимодействие с группами безопасности {#cic-sg}

[Группы безопасности](../../vpc/concepts/security-groups.md) используются для защиты облачных ресурсов {{ yandex-cloud }} и не могут быть использованы для фильтрации трафика вне {{ yandex-cloud }}.

Правила в группах безопасности должны настраиваться для префиксов, которые анонсируют клиентские маршрутизаторы в направлении {{ yandex-cloud }}. Например, для разрешения доступа из инфраструктуры клиента к веб-приложению (порт 443), развернутому в {{ yandex-cloud }}, группа безопасности должна быть настроена следующим образом:
```json
ingress {
      protocol       = "TCP"
      port           = 443
      description    = "Allow ingress traffic from Interconnect to Web server"
      v4_cidr_blocks = ["172.16.1.5/32"]
    }
egress {
      protocol       = "ANY"
      description    = "We allow any egress traffic"
      v4_cidr_blocks = ["10.0.0.0/8"]
    }
```

`Egress` правило группы безопасности разрешает всем облачным ресурсам обращаться к ресурсам в инфраструктуре клиента по любым портам без ограничений.

При необходимости можно использовать более гранулярные правила, открывая доступ только к отдельным IP-адресам или подсетям и портам:

```json
ingress {
      protocol       = "TCP"
      port           = 443
      description    = "Allow ingress traffic from Interconnect to Web server"
      v4_cidr_blocks = ["172.16.1.5/32"]
    }
egress {
      protocol       = "TCP"
      port           = 3389
      description    = "Allow RDP traffic to server behind Interconnect"
      v4_cidr_blocks = ["10.10.10.10/32"]
    }
```

