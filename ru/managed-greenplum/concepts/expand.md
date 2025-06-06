---
title: Расширение кластера {{ GP }}
description: Как устроена процедура расширения (expand) кластера {{ mgp-full-name }}.
---

# Расширение кластера {{ GP }}

Кластер можно [расширить](../operations/cluster-expand.md), чтобы добавить в него дополнительные хосты-сегменты.

Процедура расширения состоит из [этапа подготовки](#preparation) и [этапа перераспределения данных](#redistribution). Этап перераспределения данных можно выполнить как сразу после этапа подготовки, так и позднее в [фоновом режиме](#setting-delay-redistribution).

Выполнение каждого из этапов [может занять длительное время](#duration). Нельзя повлиять на длительность этапа подготовки, но можно повлиять на длительность этапа перераспределения данных, таким образом контролируя общую длительность процедуры расширения кластера.

## Этап подготовки {#preparation}

На этом этапе происходят следующие процессы:

1. В кластер добавляются новые хосты-сегменты.
1. Утилита [gpexpand]({{ gp.docs.broadcom }}/7/greenplum-database/utility_guide-ref-gpexpand.html) производит подготовку к перераспределению таблиц:

    1. Создает служебную схему данных `gpexpand` в базе данных `postgres`.

    1. Формирует очередь таблиц для перераспределения.

        Будут перераспределены все таблицы из всех баз данных кластера, но им будет назначен разный приоритет, который влияет на положение таблицы в очереди.

        [Приоритетом можно управлять](../operations/cluster-expand.md#table-priority), если перераспределение данных для конкретной таблицы [еще не началось](../operations/cluster-expand.md#redistribute-monitoring) и кластер не [закрыт от нагрузки](#setting-close-cluster).

    1. Подготавливает [партиционированные таблицы]({{ gp.docs.broadcom }}/7/greenplum-database/admin_guide-ddl-ddl-partition.html#about-the-table-partitioning-methods) к перераспределению данных.

Ориентировочная длительность этапа — несколько часов, повлиять на нее нельзя. Подробнее о длительности этапов см. [ниже](#duration).

{% note warning %}

Фактически новые хосты-сегменты будут добавлены в кластер уже на этом этапе, но расширение кластера считается выполненным только после завершения этапа перераспределения данных.

{% endnote %}

## Этап перераспределения данных {#redistribution}

На этом этапе происходят следующие процессы:

1. Данные таблиц в кластере перераспределяются с помощью утилиты [gpexpand]({{ gp.docs.broadcom }}/7/greenplum-database/utility_guide-ref-gpexpand.html) таким образом, чтобы эти данные были равномерно распределены по всем хостам-сегментам.

1. Служебная схема данных `gpexpand` удаляется.

Ориентировочная длительность этапа — несколько дней, на нее можно повлиять с помощью [настроек](#settings). Подробнее о длительности этапов см. [ниже](#duration).

## Длительность этапов и контроль за длительностью {#duration}

Ориентировочная длительность каждого этапа составляет:

* Несколько часов для [этапа подготовки](#preparation).
* Несколько дней для [этапа перераспределения данных](#redistribution).

Реальная длительность каждого этапа зависит не только от объема баз данных в кластере и общего количества таблиц, но также от уровня и характера нагрузки на кластер.

Это связано с тем, что на каждом этапе расширения кластера работает утилита [gpexpand]({{ gp.docs.broadcom }}/6/greenplum-database/utility_guide-ref-gpexpand.html), которая в ходе своей работы захватывает эксклюзивные [блокировки]({{ gp.docs.broadcom }}/7/greenplum-database/ref_guide-sql_commands-LOCK.html) на уровне отдельных таблиц. Пользовательские запросы в процессе выполнения тоже могут захватывать блокировки. Поэтому может существенно замедляться как работа `gpexpand`, так и обработка пользовательских запросов: это зависит от того, какой процесс первым захватил блокировку, а какой — ожидает особождения этой блокировки. Оба этих процесса могут создавать повышенную нагрузку на кластер.

Этап подготовки сократить нельзя, но можно повлиять на длительность этапа перераспределения данных. Для этого перед запуском процедуры [укажите](../operations/cluster-expand.md) необходимые [настройки](#settings), которые определяют поведение кластера на этом этапе. Комбинируя настройки, можно найти нужный баланс между скоростью перераспределения данных и скоростью обработки пользовательских запросов.

Поскольку этап перераспределения данных может длиться достаточно долго, для кластеров {{ mgp-name }} предоставляются [инструменты для мониторинга](../operations/cluster-expand.md#redistribute-monitoring) процесса перераспределения данных. Используйте эти инструменты, когда запущено расширение кластера, чтобы получить более точную информацию о ходе процесса и оценить время до его завершения.

## Настройки, влияющие на процесс перераспределения данных {#settings}

Доступны следующие настройки:

* **{{ ui-key.yacloud.greenplum.field_expand-close-cluster }}**{#setting-close-cluster} {{ tag-con }} {{ tag-cli }} {{ tag-api }}

    Если настройка включена (`true`), то к кластеру нельзя подключиться и в него не поступают новые пользовательские запросы. В результате расширение кластера будет выполняться быстрее, потому что не нужно ожидать освобождения блокировок, которые бы захватывались при выполнении поступающих пользовательских запросов.

    {% include [close-and-delay-warning](../../_includes/mdb/mgp/expand/close-and-delay-warning.md) %}

* **{{ ui-key.yacloud.greenplum.field_expand-delay-redistribution }}**{#setting-delay-redistribution} {{ tag-con }} {{ tag-cli }} {{ tag-api }}

    Эта настройка влияет на стратегию перераспределения данных:

    * Если настройка выключена (`false`), то данные будут перераспределены сразу же после завершения этапа подготовки к расширению кластера.

        {% include [updating-long](../../_includes/mdb/mgp/expand/updating-long.md) %}

        Процесс перераспределения данных будет запущен однократно и продлится либо до завершения перераспределения всех таблиц в кластере, либо [до истечения соответствующего таймаута](#setting-duration).

        {% include [manual-redistribution](../../_includes/mdb/mgp/expand/manual-redistribution.md) %}

    * Если настройка включена (`true`), то перераспределение данных будет отложено.

        {% include [updating-short](../../_includes/mdb/mgp/expand/updating-short.md) %}

        Процесс перераспределения данных будет запускаться по расписанию во время выполнения [регламентных операций](./maintenance.md#regular-ops) до тех пор, пока не будут обработаны все таблицы.

        При включенном фоновом перераспределении данных регламентные операции выполняются по следующему алгоритму:

        1. Выполняется [очистка пользовательских таблиц](./maintenance.md#custom-table-vacuum) (`VACUUM`).

        1. Выполняется перераспределение данных (`REDISTRIBUTE`):

            1. Если до [истечения таймаута](#setting-duration) были обработаны все таблицы, то процесс перераспределения данных будет удален из расписания регламентных работ и больше не будет запускаться.
            1. Если до истечения таймаута была обработана только часть таблиц, то процесс запустится повторно при следующем проведении регламентных операций и обработка таблиц будет продолжена.

        1. Выполняется [сбор статистики](./maintenance.md#get-statistics) (`ANALYZE`).

* **{{ ui-key.yacloud.greenplum.field_expand-duration }}**{#setting-duration} {{ tag-con }} {{ tag-cli }} {{ tag-api }}

    Таймаут (в секундах), после истечения которого процесс перераспределения данных будет прерван.

    {% include [setting-expand-duration](../../_includes/mdb/mgp/expand/setting-expand-duration.md) %}

* **{{ ui-key.yacloud.greenplum.field_expand-parallel }}**{#setting-parallel} {{ tag-con }} {{ tag-cli }} {{ tag-api }}

    Количество потоков, которые будут запущены в ходе процесса перераспределения данных.

    {% include [setting-expand-parallel](../../_includes/mdb/mgp/expand/setting-expand-parallel.md) %}

{% include [greenplum-trademark](../../_includes/mdb/mgp/trademark.md) %}
