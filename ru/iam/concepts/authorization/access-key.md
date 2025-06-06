---
title: Статические ключи доступа, совместимые с AWS API
description: Из статьи вы узнаете, зачем нужен статический ключ доступа, какой у него формат и какие сервисы поддерживают этот способ аутентификации.
---

# Статические ключи доступа, совместимые с AWS API


Статический ключ доступа необходим для аутентификации [сервисного аккаунта](../users/service-accounts.md) в [AWS-совместимых API](#supported-services).

Статический ключ состоит из двух частей:

* Идентификатор ключа.
* Секретный ключ.

Обе части используются в запросах к AWS-совместимому API. Идентификатор ключа указывается в открытом виде. Секретным ключом подписывают параметры запроса. Сам секретный ключ в запросе не указывается.

Секретный ключ пользователь хранит самостоятельно. {{ yandex-cloud }} дает доступ к ключу только в процессе [создания статического ключа](../../operations/authentication/manage-access-keys.md#create-access-key).

Срок действия статического ключа не ограничен.

{% note alert %}

Исключите доступ третьих лиц к секретному ключу. Храните ключ в надежном месте. Если ключ стал известен третьим лицам, [перевыпустите](../../operations/compromised-credentials.md#access-key-reissue) его.

{% endnote %}

{% include [key-has-last-used-data](../../../_includes/iam/key-has-last-used-data.md) %}

Помимо статических ключей доступа, для работы с {{ objstorage-full-name }} вы можете использовать временные ключи [{{ sts-name }}](sts.md) также совместимые с AWS API.

## Формат статического ключа {#access-key-format}

### Идентификатор ключа {#key-id}

Состоит из 25 символов и всегда начинается с букв `YC`. Остальными символами могут быть:

* буквы латинского алфавита;
* цифры;
* символы `_`, `-`.

Пример идентификатора ключа: `YCchbYEDdcsYFBnxSWbcjDJDn`.

### Секретный ключ {#private-key}

Состоит из 40 символов и всегда начинается с букв `YC`. Остальными символами могут быть:

* буквы латинского алфавита;
* цифры;
* символы `_`, `-`.

Пример секретного ключа: `YCVdheub7w9bImcGAnd3dZnf08FRbvjeUFvehGvc`.

Чтобы ознакомиться с примером использования секретного ключа и его идентификатора в AWS-совестимом API, смотрите раздел [AWS Command Line Interface](../../../storage/tools/aws-cli.md#config-files).

## Сервисы, поддерживающие этот способ аутентификации {#supported-services}

Статические ключи доступа в качестве способа аутентификации поддерживают несколько сервисов:

* [{{ objstorage-name }}](../../../storage/s3/index.md)
* [{{ message-queue-name }}](../../../message-queue/api-ref/index.md)
* [{{ ydb-name }}](../../../ydb/docapi/tools/aws-setup.md)
* [{{ yds-full-name }}](../../../data-streams/index.yaml)
* [{{ postbox-full-name }}](../../../postbox/aws-compatible-api/index.md)

## Примеры использования {#examples}

* [Использование секрета {{ lockbox-full-name }} для хранения статического ключа доступа](../../tutorials/static-key-in-lockbox/index.md)

#### См. также {#see-also}

* [{#T}](../../operations/authentication/manage-access-keys.md#create-access-key)
* [{#T}](./index.md)
* [{#T}](./sts.md)
* [{#T}](../../tutorials/static-key-in-lockbox/index.md)