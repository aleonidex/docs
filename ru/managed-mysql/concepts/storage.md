---
title: Типы дисков в {{ mmy-full-name }}
description: Из статьи вы узнаете, какие типы дисков есть в {{ mmy-name }} и познакомитесь с особенностями выбора типа дисков при создании кластера.
---

# Типы дисков в {{ mmy-name }}



{{ mmy-name }} позволяет использовать сетевые и локальные диски для организации хранилища кластеров баз данных. Сетевые диски реализованы на базе сетевых блоков — виртуальных дисков в инфраструктуре {{ yandex-cloud }}. Локальные диски физически размещаются в серверах хостов БД.

{% include [storage-type](../../_includes/mdb/mmy/storage-type.md) %}


## Выбор типа дисков при создании кластера {#storage-type-selection}

Количество хостов, которые можно создать вместе с кластером {{ MY }}, зависит от выбранного типа диска:

* Вы можете создать кластер только из трех или более хостов при использовании следующих типов дисков:

    * локальные SSD-диски (`local-ssd`);
    * нереплицируемые SSD-диски (`network-ssd-nonreplicated`).

   Такой кластер будет отказоустойчивым.

   Хранилище на локальных SSD-дисках влияет на тарификацию кластера: он тарифицируется, даже если остановлен. Подробнее в [правилах тарификации](../pricing.md).

* Вы можете добавить любое количество хостов в пределах [текущей квоты](./limits.md) при использовании следующих типов дисков:

    * сетевые HDD-диски (`network-hdd`);
    * сетевые SSD-диски (`network-ssd`);
    * сверхбыстрые сетевые SSD-диски с тремя репликами (`network-ssd-io-m3`).

Подробнее об ограничениях на количество хостов в кластере см. в разделе [Квоты и лимиты](./limits.md).


## Примеры использования {#examples}

* [{#T}](../tutorials/sync-mysql.md)
* [{#T}](../tutorials/bitrix-shop/index.md)
* [{#T}](../tutorials/bitrix-website/index.md)
* [{#T}](../tutorials/opencart/index.md)
