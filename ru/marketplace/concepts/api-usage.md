# Тарификация продуктов на основе метрик издателя

{{ marketplace-short-name }} предоставляет API учета потребления, чтобы продукты могли учитывать потребление и выставлять счета пользователям {{ yandex-cloud }} через сервис {{ billing-name }}. Интеграция с API требуется для любого продукта, использующего один или несколько планов тарификации с пользовательскими измерениями, чтобы учитывать события потребления.

На основе установленных вами метрик приложение (продукт) должно фиксировать события потребления и отправлять данные о них с помощью API учета потребления. В одном запросе вы можете отправить одно или несколько событий потребления. 

Спецификацию API выставления счетов по тарифу см. в разделе [{#T}](../metering/api-ref/index.md).

## Рекомендации по разработке и тестированию {#guidelines}

* Для авторизации приложения используйте [сервисный аккаунт](../../iam/concepts/users/service-accounts.md) с ролью `marketplace.meteringAgent`.
* Чтобы создать тариф для продукта с пользовательскими метриками, оставьте заявку в кабинете партнера {{ marketplace-short-name }}. Используйте выданный тарифу `skuId` для отправки записей о потреблении.
* При установке приложение должно проверить, что у пользователя достаточно прав для его использования и отправка событий потребления возможна. Для этого необходимо отправить запрос [write](../metering/api-ref/ImageProductUsage/write.md) с параметром `validateOnly: true`. 
* Приложение должно учитывать возможность неудачной попытки отправки событий потребления.
* Перед публикацией продукта рекомендуем тестировать интеграцию с API с помощью ID и SKU демонстрационного продукта: `productId: {{ mp-test-product-id }}`, `skuId: {{ mp-test-sku-id }}`. Примеры работы с API см. в разделе [{#T}](../api-ref/quickstart.md).

## Тестирование интеграции {#test}

Примеры кода и тестовый сервер для проверки работы интеграции с Metering API доступны в [репозитории на GitHub](https://github.com/yandex-cloud-examples/yc-marketplace-api-usage-examples/blob/main/metering/README.md).
