---
title: Настройка DMARC-политики
description: Следуя данной инструкции, вы настроите DMARC-политику.
---


# Настройка DMARC-политики

Чтобы защитить домен от подделки отправителей и повысить безопасность почтовых сообщений, необходимо настроить [DMARC-политику](https://{{ lang }}.wikipedia.org/wiki/DMARC). Настройки вашей доменной зоны должны содержать TXT-запись `_dmarc.<домен>.`. Вы можете добавить запись у вашего регистратора или в сервисе [{{ dns-full-name }}](../../dns/), если вы делегировали ваш домен.

**Пример создания DMARC-записи в {{ dns-name }}**

{% list tabs group=instructions %}

- Консоль управления {#console}

    1. В [консоли управления]({{ link-console-main }}) выберите каталог, в котором находятся адрес и ваша доменная зона.
    1. Выберите сервис **{{ ui-key.yacloud.iam.folder.dashboard.label_dns }}**.
    1. Выберите вашу доменную зону.
    1. Нажмите кнопку **{{ ui-key.yacloud.dns.button_record-set-create }}**.
    1. В поле **{{ ui-key.yacloud.common.name }}** укажите `_dmarc`.

        {% note info %}

        Для других DNS-сервисов может потребоваться ввести имя записи целиком. Итоговая запись должна называться `_dmarc.<домен>.`.

        {% endnote %}

    1. В списке **{{ ui-key.yacloud.common.type }}** выберите `TXT`.
    1. В поле **{{ ui-key.yacloud.dns.label_records }}** введите корректное значение DMARC-политики. Обратите внимание, что значение записи нужно взять в кавычки, например:

        ```text
        "v=DMARC1;p=none"
        ```

        Данная DMARC-запись означает, что если письмо не пройдет проверку DMARC, почтовый сервис не предпримет никаких действий, например не отклонит письмо или не отправит его в спам.

    1. Нажмите кнопку **{{ ui-key.yacloud.common.create }}**.

    Ответы DNS-сервера кешируются, поэтому возможны задержки при обновлении ресурсной записи.

{% endlist %}
