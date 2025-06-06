---
title: Хранилище в {{ mmg-full-name }}
description: Из статьи вы узнаете, что такое хранилище в {{ mmg-name }}, ознакомитесь с особенностями управления дисковым пространством и выбора типа дисков при создании кластера.
---

# Хранилище в {{ mmg-name }}


{{ mmg-name }} позволяет использовать сетевые и локальные диски для организации хранилища кластеров баз данных. Сетевые диски реализованы на базе сетевых блоков — виртуальных дисков в инфраструктуре {{ yandex-cloud }}. Локальные диски физически размещаются в серверах хостов БД.

{% include [storage-type](../../_includes/mdb/mmg/storage-type.md) %}

## Выбор типа диска при создании кластера {#storage-type-selection}

Количество хостов, которые можно создать вместе с кластером {{ MG }}, зависит от выбранного типа диска:

* Вы можете создать кластер только из трех или более хостов при использовании следующих типов дисков:

    * локальные SSD-диски (`local-ssd`);
    * нереплицируемые SSD-диски (`network-ssd-nonreplicated`).

   Такой кластер будет отказоустойчивым.

   Хранилище на локальных SSD-дисках влияет на тарификацию кластера: он тарифицируется, даже если остановлен. Подробнее в [правилах тарификации](../pricing.md).

* Вы можете добавить любое количество хостов в пределах текущей квоты при использовании следующих типов дисков:

    * сетевые HDD-диски (`network-hdd`);
    * сетевые SSD-диски (`network-ssd`);
    * сверхбыстрые сетевые SSD-диски с тремя репликами (`network-ssd-io-m3`).

Подробнее об ограничениях на количество хостов в кластере или [шарде](./sharding.md) см. в разделе [Квоты и лимиты](./limits.md).


## Управление дисковым пространством {#manage-storage-space}

{% include [mmg-readonly-safeguard.md](../../_includes/mdb/mmg-readonly-safeguard.md) %}

При переходе в режим <q>только чтение</q>:

* На хосте запрещаются запросы на запись, возможны только на чтение.
* Если хост до перевода в режим <q>только чтение</q> был [первичной репликой](replication.md), то эта роль будет автоматически назначена другому хосту кластера, т. к. роль первичной реплики требует возможности записи на диск.

Если объем данных в кластере продолжает увеличиваться, то в режим <q>только чтение</q> перейдут один за другим все хосты, и кластер перестанет принимать данные на запись.

### Поддержка работоспособности кластера {#read-only-solutions}

Чтобы сохранить работоспособность кластера при переходе хоста в режим <q>только чтение</q>:
* [Увеличьте объем диска на хосте](../operations/update.md#change-disk-size). Если на хосте станет достаточно места, {{ yandex-cloud }} снимет режим <q>только чтение</q> автоматически.
* [Добавьте в кластер дополнительные шарды](../operations/shards.md#add-shard). Режим <q>только чтение</q> на этом хосте не снимется, но кластер сможет продолжить нормальную работу — при условии, что на остальных шардах есть свободное дисковое пространство.
* Попросите [техническую поддержку]({{ link-console-support }}) о временном снятии режима <q>только чтение</q> с этого хоста, чтобы вручную удалить часть данных. 

   {% note alert %}

   Если свободное дисковое пространство снизится до нуля, {{ MG }} аварийно завершит работу и кластер станет неработоспособным.
  
   {% endnote %}

* [Принудительно синхронизируйте](../operations/hosts.md#resetup) данные между хостами. Это поможет, если из кластера был удален большой объем данных, но дисковое пространство не было освобождено (т. е. помечено как доступное к переиспользованию).

## Примеры использования {#examples}

* [{#T}](../tutorials/data-migration.md)
* [{#T}](../tutorials/mongodb-migration-with-data-transfer.md)
* [{#T}](../tutorials/mongodb-versions.md)
* [{#T}](../tutorials/profiling.md)
