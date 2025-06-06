---
title: Ошибки и предупреждения при экспорте и импорте
description: На данной странице приведены ошибки и предупреждения при экспорте и импорте, а также их описание.
---

# Ошибки и предупреждения при экспорте и импорте

При [экспорте и импорте воркбуков](./export-and-import.md) могут появляться предупреждения, отражающие особенности операции, а также сообщения об ошибках.

## При экспорте {#export-notifications}

**Сообщение** | **Пояснение**
--------------|-------------
В подключениях значения полей с паролями и токенами не экспортируются, после импорта их нужно проверить и при необходимости заменить. | При экспорте значения полей с паролями и токенами в подключениях заменяются на временные значения. Поэтому при импорте потребуется проверить корректность этих данных и при необходимости внести изменения.
В экспортированных датасетах не гарантируется корректная работа RLS. | Проверьте корректность работы и при необходимости настройте RLS заново.
Конфигурация воркбука нарушена. | Критическая ошибка. В {{ datalens-short-name }} у каждой сущности есть обязательные поля и их форматы. Ошибка указывает на отсутствие обязательных полей или неправильный тип данных. Исправьте конфигурацию или попробуйте заново экспортировать воркбук.
В воркбуке содержатся объекты с некорректным типом. | Возможно, конфигурация воркбука была отредактирована и тип некоторых объектов изменился. Например, у дашборда установился тип `Чарт`. Исправьте конфигурацию или попробуйте заново экспортировать воркбук.
Инсталляция {{ datalens-short-name }} не поддерживает конфигурацию воркбука. | Версия чарта или дашборда не совместима с инсталляцией {{ datalens-short-name }}, в которую производится импорт. Например, экспорт выполнен из новой облачной версии {{ datalens-short-name }}, а импорт осуществляется в устаревшую версию open source {{ datalens-short-name }}.
В воркбуке есть объекты, связанные с несуществующими объектами. | В экспортируемом воркбуке есть чарты или дашборды, связанные с объектами, которых нет в воркбуке. При этом экспорт не прерывается. При импорте проверьте связи объектов и при необходимости установите корректные.
Операция прервана из-за неизвестной ошибки. | Обнаружена критическая ошибка, которая привела к прерыванию текущей операции. Попробуйте заново экспортировать воркбук.
Объекты данного типа недоступны для экспорта и импорта. | Некоторые типы объектов недоступны для импорта и экспорта, подробнее см. в разделе [Ограничения](./export-and-import.md#restrictions).
Некоторые типы подключений не поддерживаются в текущей версии сервиса | Ошибка может возникать при экспорте подключений между разными инсталляциями {{ datalens-short-name }}, подключений с некорректным типом или подключений, которые более не поддерживаются.
Не удалось экспортировать датасет. | При попытке экспортировать датасет произошла неизвестная ошибка. Например, датасеты с отсутствующими подключениями недоступны для экспорта. Проверьте настройки датасета и попробуйте заново экспортировать воркбук.
Не удалось экспортировать подключение. | При попытке экспортировать подключение произошла неизвестная ошибка. Попробуйте заново экспортировать воркбук.
Неизвестная ошибка. | Неизвестен код и критичность ошибки. Попробуйте заново экспортировать воркбук.

## При импорте {#import-notifications}

**Сообщение** | **Пояснение**
--------------|-------------
Проверьте настройки подключений и при необходимости заново укажите пароли и токены. | В подключениях значения полей с паролями и токенами не экспортируются. Обратите внимание, что файл конфигурации могли изменить и внести актуальные значения. Поэтому после завершения импорта проверьте корректность данных и при необходимости внесите изменения.
Убедитесь, что RLS в датасетах работает корректно. | Проверьте корректность работы и при необходимости настройте RLS заново.
В воркбуке есть объекты, которые не поддерживаются в вашей инсталляции {{ datalens-short-name }}. | Например, при импорте воркбука, содержащего отчет или чарт в Editor, в инсталляцию open source {{ datalens-short-name }}, где они недоступны.
Настройка подключений не завершена, измените или повторно сохраните настройки подключений. | При импорте подключений с использованием сервисного аккаунта или подключения {{ connection-manager-name }} {{ datalens-short-name }} не может завершить настройку созданного подключения. После импорта повторно сохраните такие подключения без внесения изменений в настройки.
Конфигурация воркбука нарушена. | Критическая ошибка. В {{ datalens-short-name }} у каждой сущности есть обязательные поля и их форматы. Ошибка указывает на отсутствие обязательных полей или неправильный тип данных. Исправьте конфигурацию или попробуйте заново экспортировать воркбук.
В воркбуке содержатся объекты с некорректным типом. | Возможно, конфигурация воркбука была отредактирована и тип некоторых объектов изменился. Например, у дашборда установился тип `Чарт`. Исправьте конфигурацию или попробуйте заново экспортировать воркбук.
Инсталляция {{ datalens-short-name }} не поддерживает конфигурацию воркбука. | Версия чарта или дашборда не совместима с инсталляцией {{ datalens-short-name }}, в которую производится импорт. Например, экспорт выполнен из новой облачной версии {{ datalens-short-name }}, а импорт осуществляется в устаревшую версию open source {{ datalens-short-name }}.
В воркбуке есть объекты, связанные с несуществующими объектами. | В импортируемом воркбуке есть чарты или дашборды, связанные с объектами, которых нет в воркбуке. Проверьте связи объектов и при необходимости установите корректные.
Операция прервана из-за неизвестной ошибки. | Обнаружена критическая ошибка, которая привела к прерыванию текущей операции. Попробуйте заново импортировать воркбук.
Объекты данного типа недоступны для экспорта и импорта. | Некоторые типы объектов недоступны для импорта и экспорта, подробнее см. в разделе [Ограничения](./export-and-import.md#restrictions).
Некоторые типы подключений не поддерживаются в текущей версии сервиса | Ошибка может возникать при импорте подключений между разными инсталляциями {{ datalens-short-name }}, подключений с некорректным типом или подключений, которые более не поддерживаются.
Не удалось импортировать датасет. | При попытке импортировать датасет произошла неизвестная ошибка. Например, датасеты с отсутствующими подключениями недоступны для импорта. Проверьте настройки датасета и попробуйте заново импортировать воркбук.
Не удалось импортировать подключение. | При попытке импортировать подключение произошла неизвестная ошибка. Попробуйте заново импортировать воркбук.
Неизвестная ошибка. | Неизвестен код и критичность ошибки. Попробуйте заново импортировать воркбук.
