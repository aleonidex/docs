---
title: Что такое датасет в {{ foundation-models-full-name }}
description: Из этой статьи вы узнаете, что такое датасеты в {{ foundation-models-name }} и как они используются для дообучения моделей.
---

# Датасеты

Датасеты в {{ foundation-models-full-name }} хранят наборы данных для [дообучения](../tuning/index.md) и запуска моделей в пакетном режиме работы. Датасеты можно [создавать](../../operations/resources/create-dataset.md) в консоли управления, через API и [{{ ml-sdk-full-name }}](../../sdk/index.md).

Все датасеты создаются на основе файлов в формате [JSON Lines](https://jsonlines.org/) в кодировке [UTF-8](https://{{ lang }}.wikipedia.org/wiki/UTF-8). Структура содержимого датасета зависит от его типа. Вы можете создать датасеты следующих типов:

{% include [dataset-types](../../../_includes/foundation-models/dataset-types.md) %}

Актуальный список доступных типов датасетов вы можете получить, выполнив запрос:

```bash
grpcurl \
  -H "Authorization: Bearer <IAM-токен>" \
  llm.api.cloud.yandex.net:443 yandex.cloud.ai.dataset.v1.DatasetService.ListTypes 
```

## Датасеты для генерации текста {#generating}

{{ foundation-models-name }} позволяет создать два типа датасетов для генерации текста.

### Запросы для генерации текста {#text-requests}

Датасеты, содержащие только тексты запросов, подойдут для запуска моделей генерации текста в [пакетном режиме](../index.md#working-mode). Каждая строка содержит отдельный запрос, инициирующий генерацию текста, в формате JSON. В запросе может содержаться как только одно сообщение с ролью `user`, так и диалог с ролями `user` и `assistant`, а также (опционально) инструкция для модели:

```json
{"request": [{"role": "user", "text": "<Вопрос>"}]}
{"request": [{"role": "system", "text": "<инструкция>"}, {"role": "user", "text": "<Вопрос>"}]}
{"request": [{"role": "system", "text": "<инструкция>"}, {"role": "user", "text": "<Реплика_1>"}, {"role": "assistant", "text": "<Реплика_2>"}, {"role": "user", "text": "<Реплика_3>"}, {"role": "assistant", "text": "<Реплика_4>"}]}
```

### Запросы и ответы генерации текста {#text-complitions-requests}

Датасеты с вопросами и ответами используются для дообучения моделей. Также в этом формате возвращается ответ при обращении к моделям в пакетном режиме. Каждая строка содержит отдельный пример в формате JSON:

```json
{"request": [{"role": "user", "text": "<Вопрос>"}], "response": "<Ответ>"}
{"request": [{"role": "system", "text": "<инструкция>"}, {"role": "user", "text": "<Вопрос>"}], "response": "<Ответ>"}
{"request": [{"role": "system", "text": "<инструкция>"}, {"role": "user", "text": "<Реплика_1>"}, {"role": "assistant", "text": "<Реплика_2>"}, {"role": "user", "text": "<Реплика_3>"}, {"role": "assistant", "text": "<Реплика_4>"}], "response": "<Ответ>"}
```

{% note tip %}

При дообучении модели указывайте одну и ту же инструкцию для каждого обучающего примера и используйте ее при обращении к дообученной модели. Это повысит эффективность дообучения.

{% endnote %}

Датасет для дообучения должен содержать примеры как минимум 10 запросов и эталонных ответов. Максимальная длина запроса — 8 000 [токенов](../yandexgpt/tokens.md), максимальная длина эталонного ответа — 2 000 токенов. Длина запроса и ответа не должна превышать 8 000 токенов.

Пример наполнения датасета для дообучения модели генерации текста:

{% include [fine-tuning-file-example](../../../_includes/datasphere/fine-tuning-file-requirements.md) %}

## Датасеты для классификации текста {#classifier}

При создании датасетов для классификации текста старайтесь давать классам значащие имена. 

При дообучении модели классификатора на базе {{ yagpt-name }} действуют следующие ограничения:

| Вид ограничения | Минимальное значение | Максимальное значение |
|---|---|---|
| Количество примеров в датасете | 100 | 50 000 |
| Количество классов | 1 | 100 |
| Количество примеров для одного класса в датасете| 1 | — |
| Длина имени класса в символах | — | 100 |
| Количество [токенов](../yandexgpt/tokens.md) в тексте классифицируемого запроса | — | 8 000 |

Рекомендуется дообучать модели на датасетах, содержащих не менее 1 000 примеров и не менее 100 примеров для каждого класса.

### Бинарная и многоклассовая классификация {#multiclass}

Датасеты для многоклассовой и бинарной классификации должны содержать примеры текстов и их принадлежности к классам. Каждая строка содержит отдельный пример в формате JSON. Каждый пример может быть отнесен только к одному классу.

```json
{"text":"<текст_1>","<класс_1>":0,"<класс_2>":0,"<класс_3>":1}
{"text":"<текст_2>","<класс_1>":1,"<класс_2>":0,"<класс_3>":0}
{"text":"<текст_3>","<класс_1>":0,"<класс_2>":1,"<класс_3>":0}
{"text":"<текст_4>","<класс_1>":0,"<класс_2>":0,"<класс_3>":1}
```

{% include [binary](../../../_includes/foundation-models/classifier/binary-example.md) %}

{% include [multiclass](../../../_includes/foundation-models/classifier/multiclass-example.md) %}

### Классификация с несколькими метками {#multilable}

Датасеты для классификации с несколькими метками должны содержать примеры текстов и их принадлежности к классам, при этом каждый текст может относиться одновременно к нескольким классам. Каждая строка датасета содержит отдельный пример в формате JSON.

```json
{"text":"<текст_1>","<класс_1>":0,"<класс_2>":0,"<класс_3>":1}
{"text":"<текст_2>","<класс_1>":1,"<класс_2>":0,"<класс_3>":1}
{"text":"<текст_3>","<класс_1>":1,"<класс_2>":1,"<класс_3>":0}
```

{% include [multiclass](../../../_includes/foundation-models/classifier/multilable-example.md) %}

## Датасеты для эмбеддингов {#embeddings}

Датасеты для дообучения эмбеддингов могут сдержать пары текстов, близких по значению, или триплеты с текстом, близким по смыслу текстом и текстом, не имеющим отношения к основному. Каждая строка содержит отдельный пример в формате JSON:

* Структура датасета с парами: {#pairs}

  ```json
  {"anchor":"<основной_текст_1>","positive":"<близкий_текст_1>"}
  {"anchor":"<основной_текст_2>","positive":"<близкий_текст_2>"}
  ```

* Структура датасета с триплетами: {#triplets}

  ```json
  {"anchor":"<основной_текст_1>","positive":"<близкий_текст_1>","negative":"<негативный_пример_1>"}
  {"anchor":"<основной_текст_2>","positive":"<близкий_текст_2>","negative":"<негативный_пример_2>"}
  ```

  **Пример наполнения датасета с триплетами для дообучения эмбеддингов:**

  {% include [triplets](../../../_untranslatable/foundation-models/embedding-example.md) %}

  Где:

  * `anchor` — основной текст.
  * `positive` — текст, близкий по смыслу основному.
  * `negative` — текст, не имеющий отношения к основному.

 При дообучении моделей эмбеддингов под поисковые сценарии рекомендуется явно указывать, чем является текст: текстом документа или текстом запроса:

  ```json
  {"anchor": "[ЗАПРОС] город остин","positive": "[ДОКУМЕНТ] Остин расположен в центральной части штата Техас, к северо-востоку от Сан-Антонио. Через город проходит одна межштатная магистраль — I-35. Высота колеблется от 88 до 405 метров над уровнем моря.","negative": "[ДОКУМЕНТ] Биографической информации о жизни Джейн Остин мало, за исключением нескольких сохранившихся писем и биографических заметок, написанных членами её семьи. За свою жизнь Остин, возможно, написала до 3000 писем, но сохранилось только 161 письмо. Многие из писем были написаны старшей сестре Остин, Кассандре."}
  ```

## Визуально-текстовые датасеты {#vlm-datasets}

Визуально-текстовые датасеты понадобятся при работе с [мультимодальными моделями](../multimodal/index.md) в пакетном режиме. {{ foundation-models-name }} поддерживает два типа датасетов.

### Датасеты запросов {#vlm-requests}

Датасеты запросов к визуально-текстовым моделям содержат тексты запросов и изображения в кодировке Base64. Каждая строка содержит отдельный пример в формате JSON.

```json
{"request": [{"role": "user", "content": [{"type": "text", "text": "Вопрос"}, { "type": "image", "image": "Base64 код картинки"}]}]}
```

### Датасеты ответов {#vlm-response}

Датасеты ответов визуально-текстовых моделям содержат тексты запросов, изображения в кодировке Base64 и сгенерированный ответ для каждого запроса. Каждая строка содержит отдельный пример в формате JSON.

```json
{"request": [{"role": "user", "content": [{"type": "text", "text": "Вопрос"}, {"type": "image", "image": "Base64 код картинки"}]}], "response": "тут нарисован робот"}
```

#### Примеры использования {#examples}

* [{#T}](../../operations/tuning/create.md)
* [{#T}](../../operations/tuning/tune-classifiers.md)
* [{#T}](../../tutorials/models-fine-tuning.md)
* [{#T}](../../operations/tuning/tune-classifiers.md)
* [{#T}](../../operations/resources/create-dataset-gpt.md)
* [{#T}](../../operations/resources/create-dataset-classifier.md)