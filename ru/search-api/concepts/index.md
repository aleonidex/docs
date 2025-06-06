---
title: Описание и возможности сервиса {{ search-api-full-name }}
description: Из данного раздела вы узнаете, что представляет собой сервис {{ search-api-name }}, какие инструменты и возможности он представляет и какие задачи способен решать.
---

# О сервисе {{ search-api-full-name }}

Сервис {{ search-api-full-name }} позволяет автоматически отправлять запросы к поисковой базе Яндекса, получая в ответ поисковую выдачу. Сервис предназначен для разработчиков и вебмастеров.

С помощью {{ search-api-name }} можно организовать поиск по сайту, группе сайтов или интернету, а также создать приложение, использующее возможности поиска. Кроме того, сервис дает возможность отслеживать позиции сайтов по поисковым запросам в поисковой выдаче Яндекса. Сервис предоставляет возможности [текстового поиска](web-search.md) и поиска по базе изображений, позволяет настроить параметры поиска, в том числе оптимизировать результаты для конкретного устройства, и выбрать формат предоставления результатов. 

{{ search-api-name }} позволяет отправлять запросы в синхронном и отложенном (асинхронном) режиме. В синхронном режиме сервис вернет результат сразу после обработки запроса. Задержка ответа в синхронном режиме минимальна, однако он не придет моментально: для работы требуется время. В асинхронном режиме в ответ на полученный запрос придет [объект Operation](../../api-design-guide/concepts/operation.md), который содержит идентификатор выполняемой операции. По идентификатору вы сможете узнать статус запроса и позже получить его результат. 

{{ search-api-name }} предоставляет два интерфейса для выполнения запросов: API v2 и API v1. 

## Интерфейс API v2 {#api-v2}

[API v2](../operations/web-search.md) — интерфейс, тесно интегрированный в [экосистему {{ yandex-cloud }}](../../overview/concepts/services.md). API v2 спроектирован на основе [механизма gRPC](../../glossary/grpc.md) и позволяет отправлять текстовые запросы с помощью gRPC-вызовов или метода `POST` в [синхронном](../operations/web-search-sync.md) и [отложенном](../operations/web-search.md) режиме. Результат можно получить в формате [XML](./response.md) или [HTML](./html-response.md). Результаты поиска формируются по умолчанию или с оптимизацией для [мобильных устройств](../operations/v2-mobile.md).

Также с помощью интерфейса API v2 вы можете использовать текстовый поиск в сочетании с генеративными возможностями [{{ yagpt-name }}](../../foundation-models/concepts/yandexgpt/index.md). Это позволяет получить единый емкий и понятный [генеративный ответ](./generative-response.md), при формировании которого нейросеть анализирует релевантные результаты текстового поиска {{ search-api-name }} по сайтам.

При использовании API v2 тип поиска задается непосредственно в [теле](./web-search.md#parameters) каждого запроса. Для отправки запросов необходим [пользовательский](../../iam/concepts/users/accounts.md) или [сервисный](../../iam/concepts/users/service-accounts.md) аккаунт {{ yandex-cloud }} и [данные для аутентификации](../api-ref/authentication.md). Задавать доверенные IP-адреса при использовании API v2 не нужно.

## Интерфейс API v1 {#api-v1}

{% include [api-v1](../../_includes/search-api/api-v1.md) %}

API v1 — интерфейс, совместимый с [Яндекс XML](https://ru.wikipedia.org/wiki/Яндекс_XML). API v1 позволяет [выполнять](../operations/searching.md) текстовые запросы методами GET и POST, получая результат в формате XML или HTML. Поддерживается только синхронный режим работы.

Интерфейс API v1 {{ search-api-name }} позволяет получать результаты поиска по умолчанию и поиска, оптимизированного для мобильных устройств. Также API v1 позволяет искать изображения по текстовым запросам. Поиск по базе изображений возможен только с помощью метода `GET`.
