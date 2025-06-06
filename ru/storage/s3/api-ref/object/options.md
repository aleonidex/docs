# Метод options

Проверяет возможность [CORS-запроса](../../../concepts/cors.md) к объекту.

{% include [s3-api-intro-include](../../../../_includes/storage/s3-api-intro-include.md) %}

## Запрос {#request}

```http
OPTIONS /{bucket}/{key} HTTP/2
```

### Path параметры {#path-parameters}

Параметр | Описание
----- | -----
`bucket` | Имя бакета.
`key` | Ключ объекта. Идентификатор, под которым объект будет сохранен в {{ objstorage-name }}.

### Заголовки {#request-headers}

Заголовок | Описание
--------- | --------
`Origin` | Исходный домен запроса.<br/><br/>Например, `http://www.example.com`.<br/><br/>Обязательный.
`Access-Control-Request-Method` | HTTP метод, которым будет отправлен запрос к ресурсу.<br/><br/>Обязательный.
`Access-Control-Request-Headers` | Список заголовков, которые будут отправлены в последующем запросе к объекту. Заголовки отделяются запятыми.<br/><br/>Необязательный.

Также, используйте необходимые [общие заголовки](../common-request-headers.md)

## Ответ {#response}

### Заголовки {#response-headers}

Кроме [общих заголовков](../common-response-headers.md), ответ может содержать:

Заголовок | Описание
--------- | --------
`Access-Control-Allow-Origin` | Домен, который был передан в заголовке `Origin` запроса.<br/><br/>Если в [конфигурации CORS](../cors/upload.md#request-scheme) в элементе `AllowedOrigin` задано `*`, то значение заголовка `Access-Control-Allow-Origin` также будет `*`.<br/><br/>Если доступ с домена запрещен, то {{ objstorage-name }} возвращает ошибку 403 и отсутствуют все заголовки `Access-Control-*`.
`Access-Control-Max-Age` | Допустимый период (в секундах) кеширования ответа.
`Access-Control-Allow-Methods` | Допустимые для использования в запросе методы. Если допустимых методов нет, то {{ objstorage-name }} возвращает ошибку 403 и отсутствуют все заголовки `Access-Control-*`.
`Access-Control-Allow-Headers` | Список HTTP-заголовков, которые можно использовать в последующем запросе к объекту. Если все заголовки запрещены, то этот заголовок не входит в ответ.
`Access-Control-Expose-Headers` | Список HTTP-заголовков, которые получит JavaScript-клиент.

### Коды ответов {#response-codes}

Метод возвращает:

- 200 — если запросы к объекту разрешены.
- 403 — если запросы к объекту запрещены.

Подробные описания кодов ответов смотрите в разделе [{#T}](../response-codes.md).

{% include [the-s3-api-see-also-include](../../../../_includes/storage/the-s3-api-see-also-include.md) %}