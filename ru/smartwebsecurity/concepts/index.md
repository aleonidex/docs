# Обзор сервиса {{ sws-full-name }}

{% include [about-sws](../../_includes/smartwebsecurity/about-sws.md) %}

## Принцип работы {#how-it-works}

{{ sws-name }} проверяет HTTP-запросы к защищаемому ресурсу через виртуальный хост L7-балансировщика на соответствие [правилам](rules.md), заданным в [профиле безопасности](profiles.md). В зависимости от результатов проверки запросы направляются к виртуальному хосту, блокируются или отправляются в сервис [{{ captcha-full-name }}](../../smartcaptcha/) для дополнительной верификации.

![schema](../../_assets/smartwebsecurity/schema.svg)

{% include [realized-waf-concept](../../_includes/smartwebsecurity/realized-waf-concept.md) %}

{% include [realized-arl-concept](../../_includes/smartwebsecurity/realized-arl-concept.md) %}

## Мониторинг и аудит {#monitoring-audit}

Логи {{ sws-name }} передаются в сервис [{{ cloud-logging-full-name }}](../../logging/).

Метрики {{ sws-name }} передаются в сервис [{{ monitoring-full-name }}](../../monitoring/).

Аудитные логи {{ sws-name }} передаются в сервис [{{ at-full-name }}](../../audit-trails/).

{% include [user-data-to-ml](../../_includes/smartwebsecurity/user-data-to-ml.md)%}

## Рекомендации по настройке {{ alb-name }} {#alb-settings-recommendation}

{% include [alb-settings-recommendation](../../_includes/smartwebsecurity/alb-settings-recommendation.md) %}
