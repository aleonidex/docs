# История изменений в сервисе распознавания {{ speechkit-full-name }}

Сервис {{ speechkit-name }} предоставляет обновления в соответствии с системой моделей и версий.

Подробнее о способах распознавания речи читайте в разделе [О технологии](stt/index.md).



## Релиз 24.04.2025 {#240425}

В `general:rc` доступны следующие изменения:

* улучшено качество распознавания на узбекском языке.
* улучшено качество распознавания медицинской тематики на русском языке.

## Релиз 11.04.2025 {#110425}

Изменения `general:rc` стали доступны в модели `general`.

## Релиз 05.03.2025 {#050325}

В `general:rc` улучшено качество распознавания речи на узбекском и русском языках

## Релиз 17.12.2024 {#171224}

В `general:rc` улучшено качество распознавания речи на узбекском и казахском языках.

## Релиз 10.12.2024 {#101224}

Изменения `general:rc` от 3 декабря стали доступны в модели `general`.

## Релиз 03.12.2024 {#031224}

В `general:rc` исправлены и улучшены сообщения об ошибках при использовании неподдерживаемых языков распознавания и форматов аудио.

## Релиз 31.10.24 {#311024}

В `general:rc` улучшено качество распознавания речи на узбекском и турецком языках.

## Релиз 09.08.24 {#090924}

В `general:rc` доступны следующие изменения:

* Улучшено качество распознавания речи на узбекском и казахском языках.
* Теперь можно ограничить языки распознавания, указав в поле `language_restriction` несколько значений. 

## Релиз 26.06.24 {#260624}

Изменения `general:rc` от [3 июня](#030624) стали доступны в модели `general`.

В `general:rc` улучшено качество распознавания узбекского языка.

## Релиз 03.06.24 {#030624}

По запросам пользователей в `general:rc` для русского языка улучшено качество распознавания аббревиатур и медицинских терминов.

## Релиз 23.04.24 {#230424}

Изменения `general:rc` от [9 апреля](#090424) стали доступны в модели `general`.

## Релиз 09.04.24 {#090424}

В `general:rc` изменился формат [классификаторов](stt/analysis.md). Теперь классификаторы `formal_greeting`, `informal_greeting`, `formal_farewell`, `informal_farewell`, `insult` и `profanity` возвращают результат в виде вероятности срабатывания. Классификаторы `answerphone` и `negative` теперь возвращают только вероятность срабатывания вместо вероятностей принадлежности двум классам.

## Релиз 27.03.24 {#273224}

Все изменения `general:rc` [от 28 февраля](#280224) стали доступны в модели `general`.

В `general:rc` доступны следующие обновления:

* Улучшено качество распознавания речи на узбекском языке.
* Улучшено качество [определения дикторов](stt/speaker-labeling.md) в результатах распознавания.

## Релиз 28.02.24 {#280224}

В `general:rc` доступны следующие обновления:

* Улучшено качество распознавания для узбекского языка.
* По запросам пользователей для русского языка улучшено качество распознавания лекарств, марок автомобилей и табачной продукции.

## Релиз 27.02.24 {#270224}

Все изменения `general:rc` стали доступны в модели `general`.

## Релиз 12.01.24 {#120124}

В `general:rc` добавлена поддержка [определения дикторов](stt/speaker-labeling.md) в результатах распознавания.

## Релиз 12.01.24 {#120124}

В `general:rc` улучшено качество распознавания речи на узбекском языке.

## Релиз 29.12.23 {#291223}

В `general:rc` доступны следующие обновления:

1. Исправлены ошибки [нормализации](stt/normalization.md) некоторых форм чисел (например, полторы тысячи ⟶ 1500).
1. Появилась поддержка следующих [классификаторов](stt/analysis.md#classifier):

   * классификатор пола `gender`. Результатом классификации являются вероятности для классов `male` и `female`.
   * классификатор негатива `negative`. Результат классификации — вероятности для классов `negative` и `not_negative`.
   * классификатор автоответчика `answerphone`. В результате классификации возвращаются вероятности для классов `answerphone` и `not_answerphone`.

1. Добавлено срабатывание классификаторов в частичных ответах распознавания (событие `ON_PARTIAL`).

## Релиз 22.11.23 {#221123}

Все изменения `general:rc` стали доступны в модели `general`.

## Релиз 10.11.23 {#101123}

В `general:rc` доступны следующие изменения:

* Обновлена модель распознавания речи на русском языке.
* По запросам пользователей улучшено качество распознавания названий городов Республики Казахстан.
* Улучшено качество нормализации результатов распознавания речи на казахском языке.
* Исправлены внутренние ошибки сервера, возникающие при работе с маленькими фрагментами аудио.

## Релиз 06.09.23 {#060923}

В `general:rc` доступны следующие изменения:

* Исправлена проблема с появлением английских слов в распознавании русскоязычной модели.
* Улучшено общее качество распознавания на русском языке.
* Улучшено качество распознавания русской модели по запросам от пользователей.
* Улучшено общее качество распознавания на узбекском языке.

[Классификаторы аудио](stt/analysis.md), появившиеся в `general:rc` в [релизе от 15.08.23](#150823), стали доступны в `general`.

## Релиз 15.08.23 {#150823}

В `general:rc` появилась поддержка [классификаторов аудио](stt/analysis.md).

## Релиз 20.07.23 {#200723}

Исправление ресемплинга и новые метрики диалога доступны в модели `general`.

## Релиз 07.07.23 {#070723}

В `general:rc` доступны следующие изменения:

* Исправлена ошибка ресемплинга двухканальных аудио в [API v3](stt-v3/api-ref/grpc/).
* Добавлена возможность подсчета метрик диалога для речевой аналитики. Подсчет метрик настраивается с помощью опции `speech_analysis` в сообщении [`StreamingOptions`](stt-v3/api-ref/grpc/Recognizer/recognizeStreaming#speechkit.stt.v3.StreamingOptions).

## Релиз 13.06.23 {#130623}

В `general:rc` исправлены переключения на английский язык при распознавании русского языка.

## Релиз 07.06.23 {#070623}

В `general:rc` доступны следующие изменения:

* Улучшено качество распознавания на узбекском, немецком, французском, голландском, итальянском, польском языках и иврите.
* Добавлена [нормализация](stt/normalization.md) чисел для узбекского языка.
* В режиме FullData появилась поддержка [разделения текста на фразы](stt/eou.md) сообщением [`eou_update`](stt-v3/api-ref/grpc/Recognizer/recognizeStreaming.md#speechkit.stt.v3.StreamingResponse).

## Релиз 25.05.23 {#250523}

Изменения [релиза от 17 мая](#170523) стали доступны в модели `general`.

## Релиз 17.05.23 {#170523}

В `general:rc` доступны следующие изменения:

* Улучшено общее качество распознавания на русском языке.
* Улучшено качество распознавания русской модели по запросам от пользователей.
* Улучшено качество распознавания на иностранных языках (узбекский, немецкий, французский, нидерландский, итальянский, польский).
* Доступен [новый язык распознавания](stt/models.md) иврит `he-HE`.

## Релиз 14.04.23 {#140423}

Для модели `general:rc` улучшено качество распознавания русской модели на клиентских сценариях в части распознавания аббревиатур.

## Релиз 16.03.23 {#070323}

Изменения [релиза от 7 марта](#070323) стали доступны в модели `general`.

## Релиз 07.03.23 {#070323}

Для модели `general:rc`:

1. Улучшено качество распознавания на узбекском языке.
1. Добавлена нормализация чисел при распознавании речи на английском, немецком, французском, итальянском, испанском и турецком языках. Также в тестовом режиме добавлена нормализация чисел для речи на казахском языке.

## Релиз 08.02.23 {#080223}

1. В модели `general:rc` для всех версий API доступна первая версия распознавания на узбекском языке. Пока при некоторых акустических условиях узбекский язык может быть распознан как казахский. Эта проблема будет исправлена в следующих релизах модели.
1. Теперь чтобы обратиться к модели `general:rc` в [API v3](./stt-v3/api-ref/grpc/), передавайте это значение в [параметре `model`](./stt-v3/api-ref/grpc/Recognizer/recognizeStreaming#speechkit.stt.v3.RecognitionModelOptions).

## Релиз 20.12.22 {#201222}

В модели `general:rc`:
1. По обращениям пользователей улучшено качество распознавания названий лекарственных препаратов, фамилий, имен и отчеств.
1. Немного улучшено качество распознавания на казахском и турецком языках.

## Релиз 20.10.22 {#201022}

В модели `general:rc`:
1. Появилось распознавание бразильского варианта португальского языка, языковой код `pt-BR`.
1. Улучшено качество распознавания речи на всех языках в автоматическом режиме распознавания.
1. Немного улучшено качество распознавания на русском и казахском языке.

## Релиз 05.10.22 {#051022}

Изменения [релиза от 20 сентября](#200922) доступны в модели `general`.

## Релиз 20.09.22 {#200922}

В модели `general:rc`:
* для русского языка улучшено качество распознавания районов Москвы и лекарственных препаратов.
* добавлена классификация языка в автоматическом режиме распознавания.

Исправления доступны для тестирования.

## Релиз 29.06.22 {#290622}

1. Многоязычная модель стала доступна в версии `general`.
1. В версиях `general:rc` и `general` многоязычная модель может принимать подсказки, какие языки присутствуют в речи.
1. В модели `general` для русского языка стали доступны изменения в `general:rc` [от 7 июня](#070622).

## Релиз 07.06.22 {#070622}

1. В модели `general:rc` улучшено качество расстановки пунктуации и распознавание фамилий.
1. Изменения [релиза от 25 апреля](#250422) доступны в модели `general`.

## Релиз 25.04.22 {#250422}

Изменения в модели `general:rc`:

1. Улучшено распознавание слов газификация и догазификация.
1. Добавлена обратная связь сервиса при обработке формата OGG-OPUS. Если поток не является корректным аудио в формате OPUS, сервис возвращает `Invalid_Argument`.

## Релиз 19.04.22 {#190422}

1. В многоязычную модель распознавания речи добавлена поддержка турецкого языка.
1. [Новая версия API](v3/api-ref/grpc/) доступна для потокового распознавания {{ speechkit-full-name }}. Старый интерфейс также будет поддерживаться, однако все новые возможности будут доступны только в API v3.

## Релиз 14.03.22 {#140322}

[Версия `general:rc`](#020322) от 2 марта 2022 года доступна по тегу `general`.

## Релиз 02.03.22 {#020322}

Улучшенное распознавание имен, адресов и терминов, а также расстановка пунктуации в длинных предложениях и текстах, содержащих цифры, стало доступно в модели `general`.

В модель `general:rc` внесены дальнейшие изменения на основе данных пользователей.

## Релиз 17.02.22 {#170222}

В текущем релизе улучшено качество русскоязычной модели `general:rc` в следующих направлениях:

1. Распознавание фамилий, имен, отчеств и адресов.
1. Распознавание специфичных для клиентов терминов. В модель внесены данные по запросу пользователя от 1 февраля 2022 года, добавлены исправления по данным пользователя от 9 ноября 2021 года.
1. Расстановка пунктуации в длинных предложениях и текстах, содержащих цифры.

## Релиз 3.02.22 {#030222}

1. В модели `general:rc` доступен универсальный режим (язык `"auto"`). В этом режиме модель способна распознавать речь на одном из языков:
   * русский
   * казахский
   * английский
   * немецкий
   * французский
   * финский
   * шведский
   * нидерландский
   * польский
   * португальский
   * итальянский
   * испанский

1. Новые языки также доступны под своими кодами. Модель `general:rc` использует указание как подсказку для распознавания языка. При явном указании языка модель будет использовать его как подсказку для улучшения качества распознавания. В данный момент подсказка влияет только на качество распознавания русского языка.


При работе с `general:rc` рекомендуем включить автотюнинг.


_Известные проблемы_: в универсальном режиме качество распознавания может деградировать при распознавании непрерывной речи без пауз.

## Релиз 26.01.22 {#260122}

1. Модель распознавания `general` и `general:rc` для казахского языка доступна в режимах потокового и отложенного распознавания.

1. В модели `general:rc` в режимах потокового и отложенного распознавания появился пунктуатор.

1. В режиме отложенного распознавания появилась поддержка работы с форматом [MP3](https://ru.wikipedia.org/wiki/MP3).

