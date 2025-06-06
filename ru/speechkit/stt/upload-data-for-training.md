# Загрузка данных для дообучения модели распознавания речи


Чтобы загрузить данные для [дообучения модели распознавания речи](additional-training.md#advanced-training), соберите их в архив и передайте его команде {{ speechkit-name }}. Дообучение доступно только для распознавания на русском языке.

## Перед началом работы {#before-you-begin}

1. Подготовьте файлы TSV с [терминами и примерами](additional-training.md#advanced-training).
1. [Создайте сообщество](../../datasphere/operations/community/create.md) в {{ ml-platform-full-name }}. В нем вы будете работать.
1. [Привяжите к сообществу платежный аккаунт](../../datasphere/operations/community/link-ba.md).

## Подготовка архива с данными для дообучения {#prepare}

1. На главной странице {{ ml-platform-name }} [создайте проект](../../datasphere/operations/projects/create.md).
1. [Откройте ноутбук]({{ link-datasphere-main }}/import-ipynb?path=https://{{ s3-storage-host }}/doc-files/SpeechKit-prepare-templates.ipynb), содержащий функцию проверки данных. Данные будут упакованы в архив, который нужно будет передать команде {{ speechkit-name }}, чтобы дообучить модель.

1. Нажмите кнопку **Запустить в {{ ml-platform-full-name }}**.
1. Выберите проект, созданный ранее, и нажмите кнопку **Добавить**.

   Ноутбук откроется в {{ jlab }}Lab в выбранном проекте.

1. Перетащите файлы с терминами и примерами в папку проекта в {{ jlab }}Lab.
1. В ноутбуке вызовите функцию для проверки данных. Будет сформирован архив для загрузки в {{ speechkit-name }}.

   Чтобы вызвать функцию, в ноутбуке выделите ячейку, указанную ниже, и нажмите кнопку ![run](../../_assets/datasphere/jupyterlab/run.svg):

   ```python
   prepare_stt_templates(
      templates_path="<имя_файла_с_терминами_и_примерами>.tsv",
      output_path="<имя_файла_с_результатами>_tar.gz",
   )
   ```

1. Скачайте полученный архив. Для этого кликните на него правой кнопкой мыши и нажмите ![download](../../_assets/datasphere/jupyterlab/download.svg) **Download**.
1. [Обратитесь в техническую поддержку]({{ link-console-support }}) и передайте ей собранный архив.
