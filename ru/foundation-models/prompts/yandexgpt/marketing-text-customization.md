---
title: Стилизация текста
description: В статье вы ознакомитесь с промтом для стилизации текста.
---

# Стилизация текста

## Пример 1 {#example-1}

### Параметры запроса {#params}

* **Инструкция**: Ты — опытный копирайтер. Перепиши маркетинговый текст с учётом вида текста и выбранной тональности.

* **Текст запроса**: Вид текста: сообщение в телеграмме. Тональность: дружеская, позитивная. Текст: Яндекс, факультет компьютерных наук НИУ ВШЭ и Сириус.Курсы запустили бесплатный курс по искусственному интеллекту для школьников «Глубокое обучение». Старшеклассники узнают, как работают и обучаются нейросети, и познакомятся с востребованными IT-профессиями. Записаться на осенний поток можно до 15 ноября. В разработке участвовали сотрудники Яндекса, в том числе сотрудники Yandex Research и преподаватели Школы анализа данных, преподаватели факультета компьютерных наук Высшей школы экономики, эксперты онлайн-школы Сириус.Курсы. Нейросети используются во многих современных сервисах, среди них — голосовой помощник Алиса, Яндекс Браузер, поиск Яндекса, беспилотные автомобили. Курс поможет разобраться, как устроены такие технологии, как их использовать и развивать.

* **Температура**: `0.5`

* **Ответ**: Приветствую! Яндекс, Факультет Компьютерных Наук НИУ ВШЭ и Сириус.Курсы запускают бесплатный курс по Искусственному Интеллекту для школьников «Глубокое Обучение». Старшеклассники смогут узнать, как работают нейросети и как они обучаются, а также познакомятся с IT-специальностями, которые сейчас востребованы.
  Зарегистрироваться на осенний поток можно до 15 ноября. Этот курс был разработан при участии сотрудников Яндекса, включая сотрудников Yandex Research и преподавателей Школы Анализа Данных, а также преподавателей Факультета Компьютерных Наук Высшей Школы Экономики и экспертов онлайн-школы Сириус.Курсы.
  Нейросети используются в различных современных сервисах, таких как голосовой помощник Алиса в Яндексе, Яндекс Браузере и в поиске Яндекса, а также в беспилотных автомобилях. Этот курс поможет разобраться, как функционируют такие технологии и как их можно использовать и развивать в будущем.
  Не упустите возможность получить знания и навыки в области Искусственного Интеллекта! Регистрируйтесь на осенний поток уже сейчас.

### Структура запроса {#structure}

```json
{
  "modelUri": "gpt://<идентификатор_каталога>/yandexgpt-lite",
  "completionOptions": {
    "stream": false,
    "temperature": 0.5,
    "maxTokens": "2000"
  },
  "messages": [
    {
      "role": "system",
      "text": "Ты — опытный копирайтер. Перепиши маркетинговый текст с учётом вида текста и выбранной тональности."
    },
    {
      "role": "user",
      "text": "Вид текста: сообщение в телеграмме. Тональность: дружеская, позитивная. Текст: Яндекс, факультет компьютерных наук НИУ ВШЭ и Сириус.Курсы запустили бесплатный курс по искусственному интеллекту для школьников «Глубокое обучение». Старшеклассники узнают, как работают и обучаются нейросети, и познакомятся с востребованными IT-профессиями. Записаться на осенний поток можно до 15 ноября. В разработке участвовали сотрудники Яндекса, в том числе сотрудники Yandex Research и преподаватели Школы анализа данных, преподаватели факультета компьютерных наук Высшей школы экономики, эксперты онлайн-школы Сириус.Курсы. Нейросети используются во многих современных сервисах, среди них — голосовой помощник Алиса, Яндекс Браузер, поиск Яндекса, беспилотные автомобили. Курс поможет разобраться, как устроены такие технологии, как их использовать и развивать."
    }
  ]
}
```

{% include [folder-id](../../../_includes/foundation-models/yandexgpt/folder-id.md) %}

{% list tabs group=programming_language %}

- cURL {#curl}

  ```bash
  curl \
    --insecure \
    --verbose \
    --request POST \
    --header "Authorization: Bearer <IAM-токен>" \
    --data @prompt.json \
    https://llm.{{ api-host }}/foundationModels/v1/completion
  ```

  Где:

  * `<IAM-токен>` — значение IAM-токена, полученного для вашего аккаунта.
  * `prompt.json` — файл в формате JSON, содержащий параметры запроса.

{% endlist %}

### Ответ {#answer}

```json
{
    "result": {
        "alternatives": [
            {
                "message": {
                    "role": "assistant",
                    "text": "Приветствую! Яндекс, Факультет Компьютерных Наук НИУ ВШЭ и Сириус.Курсы запускают бесплатный курс по Искусственному Интеллекту для школьников \"Глубокое Обучение\". Старшеклассники смогут узнать, как работают нейросети и как они обучаются, а также познакомятся с IT-специальностями, которые сейчас востребованы.\n\nЗарегистрироваться на осенний поток можно до 15 ноября. Этот курс был разработан при участии сотрудников Яндекса, включая сотрудников Yandex Research и преподавателей Школы Анализа Данных, а также преподавателей Факультета Компьютерных Наук Высшей Школы Экономики и экспертов онлайн-школы Сириус.Курсы.\n\nНейросети используются в различных современных сервисах, таких как голосовой помощник Алиса в Яндексе, Яндекс Браузере и в поиске Яндекса, а также в беспилотных автомобилях. Этот курс поможет разобраться, как функционируют такие технологии и как их можно использовать и развивать в будущем.\n\nНе упустите возможность получить знания и навыки в области Искусственного Интеллекта! Регистрируйтесь на осенний поток уже сейчас."
                },
                "status": "ALTERNATIVE_STATUS_FINAL"
            }
        ],
        "usage": {
            "inputTextTokens": "193",
            "completionTokens": "206",
            "totalTokens": "399"
        },
        "modelVersion": ""
    }
}
```

## Пример 2 {#example-2}

### Параметры запроса {#params}

* **Инструкция**: Ты — опытный копирайтер. Перепиши маркетинговый текст с учётом вида текста и выбранной тональности.

* **Текст запроса**: Вид текста: пресс-релиз. Тональность: нейтральная. Текст: Яндекс, факультет компьютерных наук НИУ ВШЭ и Сириус.Курсы запустили бесплатный курс по искусственному интеллекту для школьников «Глубокое обучение». Старшеклассники узнают, как работают и обучаются нейросети, и познакомятся с востребованными IT-профессиями. Записаться на осенний поток можно до 15 ноября. В разработке участвовали сотрудники Яндекса, в том числе сотрудники Yandex Research и преподаватели Школы анализа данных, преподаватели факультета компьютерных наук Высшей школы экономики, эксперты онлайн-школы Сириус.Курсы. Нейросети используются во многих современных сервисах, среди них — голосовой помощник Алиса, Яндекс Браузер, поиск Яндекса, беспилотные автомобили. Курс поможет разобраться, как устроены такие технологии, как их использовать и развивать.


* **Температура**: `0.5`

* **Ответ**: Яндекс, Факультет Компьютерных Наук НИУ ВШЭ и онлайн-школа Сириус.Курсы, рады сообщить о запуске бесплатного курса по Искусственному Интеллекту для школьников – «Глубокое Обучение». В рамках курса, старшеклассники познакомятся с работой и обучением нейронных сетей, а также узнают о востребованных IT-профессиях.
  Регистрация на осенний поток открыта до 15 ноября. Курс разработан при участии сотрудников Яндекса, включая специалистов из Yandex Research и преподавателей Школы Анализа Данных, а также преподавателей Факультета Компьютерных Наук Высшей Школы Экономики и экспертов онлайн-школы Сириус.Курсы.
  Нейросети находят широкое применение в современных технологиях, таких как голосовой помощник Алиса, Яндекс.Браузер, поисковая система Яндекса и беспилотные автомобили, и данный курс поможет учащимся понять, как функционируют эти технологии, а также научиться их использовать и совершенствовать.

### Структура запроса {#structure}

```json
{
  "modelUri": "gpt://<идентификатор_каталога>/yandexgpt-lite",
  "completionOptions": {
    "stream": false,
    "temperature": 0.5,
    "maxTokens": "2000"
  },
  "messages": [
    {
      "role": "system",
      "text": "Ты — опытный копирайтер. Перепиши маркетинговый текст с учётом вида текста и выбранной тональности."
    },
    {
      "role": "user",
      "text": "Вид текста: пресс-релиз. Тональность: нейтральная. Текст: Яндекс, факультет компьютерных наук НИУ ВШЭ и Сириус.Курсы запустили бесплатный курс по искусственному интеллекту для школьников «Глубокое обучение». Старшеклассники узнают, как работают и обучаются нейросети, и познакомятся с востребованными IT-профессиями. Записаться на осенний поток можно до 15 ноября. В разработке участвовали сотрудники Яндекса, в том числе сотрудники Yandex Research и преподаватели Школы анализа данных, преподаватели факультета компьютерных наук Высшей школы экономики, эксперты онлайн-школы Сириус.Курсы. Нейросети используются во многих современных сервисах, среди них — голосовой помощник Алиса, Яндекс Браузер, поиск Яндекса, беспилотные автомобили. Курс поможет разобраться, как устроены такие технологии, как их использовать и развивать."
    }
  ]
}
```

{% include [folder-id](../../../_includes/foundation-models/yandexgpt/folder-id.md) %}

{% list tabs group=programming_language %}

- cURL {#curl}

  ```bash
  curl \
    --insecure \
    --verbose \
    --request POST \
    --header "Authorization: Bearer <IAM-токен>" \
    --data @prompt.json \
    https://llm.{{ api-host }}/foundationModels/v1/completion
  ```

  Где:

  * `<IAM-токен>` — значение IAM-токена, полученного для вашего аккаунта.
  * `prompt.json` — файл в формате JSON, содержащий параметры запроса.

{% endlist %}

### Ответ {#answer}

```json
{
    "result": {
        "alternatives": [
            {
                "message": {
                    "role": "assistant",
                    "text": "Яндекс, Факультет Компьютерных Наук НИУ ВШЭ и онлайн-школа Сириус.Курсы, рады сообщить о запуске бесплатного курса по Искусственному Интеллекту для школьников – «Глубокое Обучение». В рамках курса, старшеклассники познакомятся с работой и обучением нейронных сетей, а также узнают о востребованных IT-профессиях.\n\nРегистрация на осенний поток открыта до 15 ноября. Курс разработан при участии сотрудников Яндекса, включая специалистов из Yandex Research и преподавателей Школы Анализа Данных, а также преподавателей Факультета Компьютерных Наук Высшей Школы Экономики и экспертов онлайн-школы Сириус.Курсы.\n\nНейросети находят широкое применение в современных технологиях, таких как голосовой помощник Алиса, Яндекс.Браузер, поисковая система Яндекса и беспилотные автомобили, и данный курс поможет учащимся понять, как функционируют эти технологии, а также научиться их использовать и совершенствовать."
                },
                "status": "ALTERNATIVE_STATUS_FINAL"
            }
        ],
        "usage": {
            "inputTextTokens": "190",
            "completionTokens": "178",
            "totalTokens": "368"
        },
        "modelVersion": ""
    }
}
```
