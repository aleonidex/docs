---
title: Посмотреть статистику CDN-ресурса
description: Следуя данной инструкции, вы сможете посмотреть статистику CDN-ресурса.
---

# Просмотр статистики CDN-ресурса

{{ cdn-name }} автоматически отправляет метрики работы CDN-ресурса в сервис [{{ monitoring-full-name }}](../../../monitoring/).

Метрики {{ cdn-short-name }} обновляются в среднем раз в 3 минуты.

Полный список метрик представлен в [справочнике](../../metrics.md).

## Посмотреть общую статистику на странице CDN-ресурса {#cdn-ui}

{% list tabs group=instructions %}

- Консоль управления {#console}

  1. В [консоли управления]({{ link-console-main }}) выберите каталог, в котором создан CDN-ресурс.
  1. В списке сервисов выберите **{{ ui-key.yacloud.iam.folder.dashboard.label_cdn }}**.
  1. Выберите CDN-ресурс, статистику которого вы хотите посмотреть.
  1. Перейдите на вкладку **{{ ui-key.yacloud.common.monitoring }}**.
  1. Выберите временной промежуток, за который хотите посмотреть статистику: 1 час, 3 часа, 1 день, 1 неделя, 1 месяц.

      На вкладке отображаются следующие дашборды:

      * **Requests** — количество запросов к CDN-серверам в секунду.
      * **Origin_requests** — количество запросов к источникам в секунду.
      * **Bandwidth** — скорость соединения клиентов с CDN-серверами.
      * **Origin_bandwidth** — скорость соединения CDN-серверов с [источником](../../concepts/origins.md).
      * **Requests_codes** — количество ответов с HTTP-кодами от CDN-серверов.
      * **Origin_requests_codes** — количество ответов с HTTP-кодами от источников.
      * **Requests_total** — общее количество запросов к CDN-серверам.
      * **Requests_waf_blocked/waf_passed** — общее количество запросов к CDN-серверам, разрешенных и заблокированных WAF CDN-провайдера.
      * **Hit_bytes** — объем кешированного трафика.
      * **Hit_ratio** — соотношение кешированного трафика.

{% endlist %}

## Посмотреть подробную статистику в {{ monitoring-name }} {#monitoring}

{% list tabs group=instructions %}

- Консоль управления {#console}

  1. В [консоли управления]({{ link-console-main }}) выберите каталог, в котором создан CDN-ресурс.
  1. В списке сервисов выберите **{{ ui-key.yacloud.iam.folder.dashboard.label_monitoring }}**.
  1. На панели слева выберите ![image](../../../_assets/monitoring/concepts/visualization/legend-goto-chart.svg) **{{ ui-key.yacloud_monitoring.aside-navigation.menu-item.explorer.title }}**.
  1. В строке запроса выберите параметры:
      1. Сервис **{{ cdn-name }}**.
      1. Метрику, значения которой вы хотите визуализировать.
      1. Метку для метрики.

{% endlist %}