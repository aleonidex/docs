---
title: Как создать подключение к AppMetrica в {{ datalens-full-name }}
description: Следуя данной инструкции, вы сможете создать подключение к AppMetrica.
---

# Создание подключения к AppMetrica

Чтобы создать подключение к AppMetrica:


1. Перейдите на [страницу создания нового подключения]({{ link-datalens-main }}/connections/new).
1. В разделе **Файлы и сервисы** выберите подключение **AppMetrica**.
1. Укажите параметры подключения:

   * **OAuth-токен**. Нажмите кнопку **Получить токен** или укажите вручную [OAuth-токен](#get-oauth-token) для доступа к данным AppMetrica.
   * **Приложение**. Укажите одно или несколько приложений для подключения.
   * **Точность**. Задайте точность данных (сэмплирование). Вы можете изменить точность после создания подключения.

     {% include [datalens-get-token](../../../_includes/datalens/datalens-change-account-note.md) %}


1. Оставьте опцию **Автоматически создать дашборд, чарты и датасет над подключением** включенной, если хотите получить каталог со стандартным набором датасетов, чартов и готовый дашборд.


1. Нажмите кнопку **Создать подключение**.


1. Выберите [воркбук](../../workbooks-collections/index.md), в котором сохранится подключение, или создайте новый. Если вы пользуетесь старой навигацией по папкам, выберите папку для сохранения подключения. Нажмите кнопку **Создать**.


1. Укажите название подключения и нажмите кнопку **Создать**.

{% include [datalens-appmetrica-note](../../../_includes/datalens/datalens-appmetrica-note.md) %}

Для датасета на основе подключения AppMetrica доступны следующие группы метрик:

- Установки
- Аудитории
- Клиентские события
- Push-кампании
- Аудитории + соц.дем 


Подключение к AppMetrica не поддерживает [публичный доступ](../../concepts/datalens-public.md) к объектам, созданным на его основе. Чтобы поделиться дашбордом или чартом, созданным на основе данного подключения, воспользуйтесь одним из способов:

{% include [datalens-metrica-appmetrica-share](../../../_includes/datalens/datalens-metrica-appmetrica-share.md) %}


{% include [datalens-get-token](../../../_includes/datalens/operations/datalens-get-token.md) %}

{% include [clickhouse-disclaimer](../../../_includes/clickhouse-disclaimer.md) %}




## Примеры использования {#examples}
* [{#T}](../../tutorials/data-from-appmetrica-visualization.md)
* [{#T}](../../tutorials/data-from-appmetrica-yc-visualization.md)

