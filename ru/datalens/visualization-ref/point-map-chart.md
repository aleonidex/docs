# Точечная карта ![](../../_assets/datalens/heatmap.svg)

Каждая точка откладывается по своим географическим координатам.

Размер точки может выражать значение показателя: чем больше площадь точки, тем выше значение показателя. Такую карту также можно назвать пузырьковой.

Точки могут быть раскрашены в разные цвета в зависимости от принадлежности к какой-либо категории. Также цвет точки может выражать значение показателя: чем выше значение показателя, тем интенсивнее будет закрашена точка.

Карта используется для отслеживания распределения точек на местности. Например, на точечную карту можно нанести расположение магазинов и выделить группу с высокой прибылью.

![point-chart](../../_assets/datalens/visualization-ref/point-chart/point-chart.png)

## Секции в визарде {#wizard-sections}

Секция<br/> в визарде| Описание
----- | ----
Точки (Геоточки) | Измерение с типом [Геоточка](../dataset/data-types.md#geopoint).
Размер точек | Показатель. Задает размер точки в зависимости от значения показателя.
Цвета | Измерение или показатель. Влияет на интенсивность закрашивания точек.
Подписи | Показатель. Отображается в виде подписи на точке. При использовании подписи блокируется управление размером точки.
Тултипы | Измерение или показатель. Подсказка, которая отобразится при наведении на точку. Для полей с типом `Строка` можно настроить использование базового синтаксиса [{#T}](../dashboard/markdown.md): нажмите на значок перед названием поля и включите опцию **Markdown**.
Фильтры слоя | Измерение или показатель. Используется в качестве фильтра текущего слоя.
Фильтры | Измерение или показатель. Используется в качестве фильтра всего чарта.

## Создание точечной карты {#create-diagram}


{% include [datalens-workbooks-collections-note](../../_includes/datalens/operations/datalens-workbooks-collections-note-step4.md) %}


1. Перейдите на [главную страницу]({{ link-datalens-main }}) {{ datalens-short-name }}.
1. На панели слева выберите ![chart](../../_assets/console-icons/chart-column.svg) **Чарты**.
1. Нажмите кнопку **Создать чарт** → **Чарт**.
1. Слева вверху нажмите ![image](../../_assets/console-icons/circles-intersection.svg) **Выберите датасет** и укажите датасет для визуализации. Если у вас нет датасета, [создайте его](../dataset/create-dataset.md#create).
1. Выберите тип чарта **Карта**.
1. Выберите тип слоя **Точки (Геоточки)**.
1. Перетащите измерение с типом [Геоточка](../dataset/data-types.md#geopoint) из датасета в секцию с выбором типа слоя.
1. Задайте размер отображаемых точек для точечной карты. Перенесите показатель в секцию **Размер точек** или нажмите значок ![image](../../_assets/console-icons/gear.svg) в строке с названием секции.
1. Закрасьте точки на карте. Перенесите показатель или измерение в секцию **Цвета**.
1. Добавьте тултипы из значений измерения и показателя, которые отображаются при наведении на точку.

{% include [datalens-map-scale-note](../../_includes/datalens/datalens-map-scale-note.md) %}

Дополнительно вы можете:

* добавить, переименовать, удалить слой;
* поменять слои местами в рамках их типа;
* применить фильтр ко всему чарту или к одному слою.

## Рекомендации {#recomendations}

* Используйте цветовые палитры, соответствующие целям и задачам визуализации.
* Если точки на карты расположены с высокой плотностью, сделайте их менее прозрачными. Так вы сможете увидеть пересечения данных.


## Примеры использования {#examples}

* [{#T}](../tutorials/data-from-ch-to-geolayers-visualization.md)
* [{#T}](../tutorials/data-from-ch-geocoder.md)


#### См. также {#see-also}

* [{#T}](../operations/dashboard/create.md)
* [{#T}](../operations/dashboard/add-chart.md)
* [{#T}](../operations/dashboard/add-selector.md)