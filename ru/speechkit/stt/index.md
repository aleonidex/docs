# Распознавание речи

_[Распознавание речи](../../glossary/transcribation.md) (speech-to-text — STT)_ — это процесс преобразования речи в текст.

{% include [api-concepts](../../_includes/speechkit/api-concepts.md) %}

Сервис доступен по адресу `{{ api-host-sk-stt }}:443`.

Для работы со {{ speechkit-name }} также доступен [Python SDK](../sdk/python/index.md). Он основан на [API v3 {{ speechkit-name }}](../stt-v3/api-ref/grpc/).

{% include [try-speechkit](../../_includes/speechkit/try-speechkit.md) %}

## Способы распознавания {#stt-ways}

{{ speechkit-name }} предоставляет два способа распознавания речи:

1. [Потоковое распознавание](streaming.md) применяется для распознавания в режиме реального времени. При потоковом распознавании {{ speechkit-name }} получает короткие аудиофрагменты и отправляет результаты, в том числе промежуточные, в рамках одного соединения.
1. Распознавание аудиофайлов. {{ speechkit-name }} может распознавать аудиозаписи в [синхронном](request.md) и [асинхронном](transcribation.md) режиме.
   * Синхронное распознавание имеет жесткие ограничения на размер и длительность файла и подходит для распознавания одноканальных аудио до {{ stt-short-audioLength }}.
   * Асинхронное распознавание может обрабатывать многоканальные аудиозаписи. Максимальная длительность файла — {{ stt-long-audioLength }}.


### Какое распознавание выбрать {#choose-stt}


|  | [Потоковое распознавание](streaming.md) | [Синхронное распознавание](request.md) | [Асинхронное распознавание](transcribation.md) |
|---|---------|----------------------|----------------------|
| **Сценарии использования** | Телефонные ассистенты и роботы </br> Виртуальные ассистенты         | Виртуальные ассистенты </br> Голосовое управление </br> Распознавание коротких голосовых сообщений в мессенджерах | Транскрибация аудиозвонков и выступлений </br> Создание субтитров </br> Контроль соблюдения скриптов колл-центров </br> Выявление успешных скриптов </br> Оценка качества работы операторов колл-центров |
| **Входные данные** | Голос в режиме реального времени | Предзаписанные одноканальные короткие аудиофайлы | Предзаписанные многоканальные и длинные аудиофайлы |
| **Принцип работы** | Обмен сообщениями с сервером в рамках одного соединения | Запрос — быстрый ответ | Запрос — отложенный ответ |
| **Поддерживаемые API** | [gRPC v2](api/streaming-api.md) </br> [gRPC v3](../stt-v3/api-ref/grpc/index.md) | [REST v1](api/request-api.md) | [REST v2](api/transcribation-api.md) </br> [REST v3](../stt-v3/api-ref/index.md) </br> [gRPC v3](api/transcribation-api-v3.md) |
| **Максимальная длительность аудиоданных** | {{ stt-streaming-audioLength }} | {{ stt-short-audioLength }} | {{ stt-long-audioLength }} |
| **Максимальный объем переданных данных** | {{ stt-streaming-fileSize }} | {{ stt-short-fileSize }} | {{ stt-long-fileSize }} |
| **Количество распознаваемых каналов** | 1  | 1 | 2  |


## Процесс распознавания {#process}

Распознавание аудио происходит в три этапа:

1. Акустическая модель определяет, какой набор низкоуровневых признаков соответствует звуковому сигналу.
1. Языковая модель на основе выхода акустической модели генерирует текст по словам.
1. Сервис обрабатывает текст: производит расстановку пунктуации, преобразование числительных в цифры и т.п.

## Точность распознавания {#speed_and_accuracy}

Точность распознавания зависит от модели распознавания. Вы можете повысить точность распознавания модели, предоставив данные для [дообучения](../../glossary/ml-models.md#fine-tuning) модели. Подробнее о дообучении моделей см. в разделе [{#T}](additional-training.md).

Также на точность распознавания влияют:

* качество исходного звука;
* качество кодирования аудио;
* разборчивость и темп речи;
* сложность фраз и их длина.

## Примеры использования {#examples}

* [{#T}](api/streaming-examples-v3.md)
* [{#T}](api/stt-language-labels-example.md)
* [{#T}](api/transcribation-api-v3.md)
* [{#T}](api/request-examples.md)

#### См. также {#see-also}

* [{#T}](../formats.md)
* [{#T}](models.md)
* [{#T}](streaming.md)
* [{#T}](eou.md)
* [{#T}](normalization.md)
* [{#T}](request.md)
* [{#T}](transcribation.md)
* [{#T}](additional-training.md)