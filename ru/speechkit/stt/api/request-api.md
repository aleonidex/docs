---
title: API синхронного распознавания в {{ speechkit-full-name }}
description: Следуя данной инструкции, вы сможете использовать API синхронного распознавания.
---

# API синхронного распознавания


С помощью [API синхронного распознавания](../request.md) можно распознавать подготовленные аудиофайлы с такими характеристиками:

* максимальный размер файла — {{ stt-short-fileSize }};
* максимальная длительность — {{ stt-short-audioLength }};
* максимальное количество аудиоканалов — {{ stt-short-channelsCount }}.

Сервис синхронного распознавания располагается по адресу: `stt.{{ api-host }}/speech/v1/stt:recognize`

## Query-параметры {#query_params}

Параметр | Описание
----- | -----
lang | **string**<br>Язык, для которого будет выполнено распознавание.<br/>Допустимые значения см. в [описании модели](../../stt/models.md). Значение по умолчанию — `ru-RU`  — русский язык.
topic | **string**<br>Языковая модель, которую следует использовать при распознавании.<br/>Чем точнее выбрана модель, тем лучше результат распознавания. В одном запросе можно указать только одну модель.<br/>[Допустимые значения](../../stt/models.md) зависят от выбранного языка. Значение параметра по умолчанию: `general`.
profanityFilter | **boolean**<br>Параметр, регулирующий работу фильтра ненормативной лексики в распознанной речи.<br>Допустимые значения:<ul><li>`false` (по умолчанию) — ненормативная лексика не будет исключена из результатов распознавания;</li><li>`true` — ненормативная лексика будет исключена из результатов распознавания.</li></ul>
rawResults |  **boolean** <br>Флаг, указывающий, как писать числа. `true` — писать прописью, `false` (по умолчанию) — писать цифрами.
format | **string**<br>[Формат](../../formats.md) передаваемого аудио.<br/>Допустимые значения:<ul><li>`lpcm` — [LPCM без WAV-заголовка](../../formats.md#lpcm).</li><li>`oggopus` (по умолчанию) — [OggOpus](../../formats.md#oggopus).</li></ul>
sampleRateHertz | **string**<br>Частота дискретизации передаваемого аудио.<br/>Применяется, если значение `format` равно `lpcm`. Допустимые значения:<ul><li>`48000` (по умолчанию) — частота дискретизации 48 кГц;</li><li>`16000` — частота дискретизации 16 кГц;</li><li>`8000` — частота дискретизации 8 кГц.</li></ul>
folderId | **string**<br><p>[Идентификатор каталога](../../../resource-manager/operations/folder/get-id.md), к которому у вас есть доступ. Требуется для авторизации с пользовательским аккаунтом (см. ресурс [{#T}](../../concepts/auth.md)). Не используйте это поле, если вы делаете запрос от имени сервисного аккаунта.</p> <p>Максимальная длина строки в символах — 50.</p>

## Параметры в теле запроса {#body_params}

В теле необходимо передать двоичное содержимое аудиофайла.

## Ответ {#response}

Распознанный текст возвращается в ответе в поле `result`.

```json
{
  "result": <распознанный_текст>
}
```

Подробнее о формате и кодах ответов см. на странице [{#T}](../../concepts/response.md).

## Примеры использования {#examples}

* [{#T}](request-examples.md).
