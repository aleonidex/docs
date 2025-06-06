---
title: Какие заголовки запросов существуют в S3 API {{ objstorage-full-name }}
description: Из статьи вы узнаете, какие заголовки запросов существуют в S3 API.
---

# Общие заголовки запросов

Заголовок | Описание
----- | -----
`Authorization` | Любой запрос к Yandex {{ objstorage-name }} должен быть авторизован.<br/><br/>Вместе с этим заголовком обязательно надо использовать либо заголовок `Date`, либо заголовок `X-Amz-Date`.<br/><br/>О способах авторизации читайте в соответствующих разделах руководства.
`Cache-Control` | Набор директив для кеширования данных согласно [RFC 2616](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.9).
`Content-Disposition` | Имя, под которым {{ objstorage-name }} предложит сохранить объект как файл при выгрузке. Соответствует [RFC 2616](http://www.w3.org/Protocols/rfc2616/rfc2616-sec19.html#sec19.5.1).
`Content-Encoding` | Определяет кодировку содержимого согласно [RFC 2616](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.11).
`Content-Length` | Длина тела запроса (без заголовков) в соответствии с [RFC 2616](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.13).<br/><br/>Заголовок обязателен для всех запросов, которые передают в {{ objstorage-name }} какие-либо данные, например, при загрузке объекта.
`Content-Type` | Тип данных в запросе. Например, `text/html`. Подробнее про типы данных читайте в статье Википедии [Список MIME-типов](https://ru.wikipedia.org/wiki/Список_MIME-типов).<br/><br/>По умолчанию выставляется `binary/octet-stream`.
`Content-MD5` | 128-битный MD5-хэш тела запроса, закодированный `base64`.<br/><br/>Соответствует спецификации [RFC 1864](http://www.ietf.org/rfc/rfc1864.txt).<br/><br/>{{ objstorage-name }} использует заголовок, чтобы убедиться, что отправленные данные соответствуют полученным.<br/><br/>Если в бакете настроены [блокировки версий объектов по умолчанию](../../concepts/object-lock.md#default), использовать заголовок `Content-MD5` при загрузке или копировании версии объекта обязательно.
`Date` | Дата и время отправки запроса.<br/><br/>Формат: `Thu, 18 Jan 2018 09:57:35 GMT`.<br/><br/>Когда установлен `X-Amz-Date`, {{ objstorage-name }} игнорирует заголовок `Date`.
`Expect` | Ожидаемый код `100-continue`.<br/><br/>Приложение при загрузке данных в {{ objstorage-name }} может использовать следующую логику:<br/>- Отправлять запрос без тела, но с выставленным заголовком Expect: 100-continue.<br/>- Отправлять запрос с телом после получения ответа `100-continue`. В этом запросе заголовка `Expect` быть не должно.
`Expires` | Срок устаревания ответа. Соответствует [RFC 2616](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.21).
`Host` | Хост-получатель запроса.<br/><br/>Заголовок необходим для HTTP/1.1, но необязателен для HTTP/1.0 запросов.
`X-Amz-Date` | Дата и время на источнике запроса.<br/><br/>Формат: `20211102T145822Z`.<br/><br/>Когда установлен `X-Amz-Date`, {{ objstorage-name }} игнорирует заголовок `Date`.

Если отправляется кросс-доменный ([CORS](../../../glossary/cors.md)) запрос, то он может содержать [заголовки](object/options.md#request-headers) предварительного запроса `options`.

{% include [the-s3-api-see-also-include](../../../_includes/storage/the-s3-api-see-also-include.md) %}