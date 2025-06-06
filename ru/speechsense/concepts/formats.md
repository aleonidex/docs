---
title: Поддерживаемые форматы аудио
description: 'Из статьи вы узнаете про поддерживаемые форматы аудио: LPCM, WAV, MP3 и OggOpus.'
---

# Поддерживаемые форматы аудио

{{ speechsense-name }} позволяет [загружать аудио](../operations/data/upload-data.md) в форматах:

* [LPCM](#lpcm);
* [WAV](#WAV);
* [MP3](#MP3) — предпочтительный формат;
* [OggOpus](#OggOpus).

{% note info %}

{% include notitle [max-duration](../../_includes/speechsense/data/max-duration.md) %}

{% endnote %}

## LPCM {#lpcm}

[Linear pulse-code modulation](https://ru.wikipedia.org/wiki/Импульсно-кодовая_модуляция) (линейная импульсно-кодовая модуляция) — формат кодирования звука без сжатия.

Требования к аудио в формате LPCM:

* частота дискретизации — в диапазоне 8–48 кГц;
* разрядность квантования — 16 бит;
* порядок байтов — обратный (little-endian);
* аудиоданные хранятся как знаковые числа (signed integer).

## WAV {#WAV}

[WAV](https://ru.wikipedia.org/wiki/WAV) — данные закодированы с помощью LPCM и упакованы в контейнер WAV.

К аудио в формате WAV предъявляются те же требования, что и к LPCM. {{ speechsense-name }} не распознает WAV, если данные закодированы в формате, отличном от LPCM, или аудио не соответствует требованиям, предъявляемым к этому формату.

## MP3 {#MP3}

[MP3](https://ru.wikipedia.org/wiki/MP3) — данные закодированы с помощью аудиокодека MPEG-1/2/2.5 Layer III и упакованы в контейнер MP3.

{{ speechsense-name }} распознает MP3 без ограничений на качество и заголовки аудиофайлов.

## OggOpus {#OggOpus}

[OggOpus](https://wiki.xiph.org/OggOpus) — данные закодированы с помощью аудиокодека OPUS и упакованы в контейнер OGG.

{{ speechsense-name }} распознает OggOpus без ограничений на качество и заголовки аудиофайлов.