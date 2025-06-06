---
title: Резервное копирование в {{ compute-full-name }}
description: Резервное копирование данных в образах, снимках и на дисках, не подключенных к виртуальным машинам, надежно хранятся и реплицируются в инфраструктуре {{ yandex-cloud }}. Чтобы делать резервные копии данных с дисков, подключенных к ВМ, периодически создавайте снимки нужных дисков.
keywords:
  - бэкап
  - бекап
  - backup
  - резервное копирование
  - резервное копирование данных
---

# Резервное копирование в {{ compute-name }}


Данные в образах, снимках и на дисках, не подключенных к виртуальным машинам, надежно хранятся и реплицируются в инфраструктуре {{ yandex-cloud }}.

Чтобы делать резервные копии данных с дисков, подключенных к ВМ, периодически создавайте [снимки](../operations/disk-control/create-snapshot.md) нужных дисков — вручную или автоматически, [по расписаниям](snapshot-schedule.md).

Восстановить ВМ с состоянием нужного диска на момент создания снимка можно следующими способами:
* Создать новый диск из снимка, затем [подключить созданный диск](../operations/vm-control/vm-attach-disk.md) к ВМ.
* Создать новую ВМ [из набора снимков](../operations/vm-create/create-from-snapshots.md) — в параметрах операции можно указать снимки как загрузочного, так и обычных дисков.

{% include [boot-disk-recover](../../_includes/compute/boot-disk-recover.md) %}


## Сравнение с резервным копированием в {{ backup-name }} {#backup-comparison}

В сервисе {{ backup-name }} вы можете создавать [полные и инкрементальные копии](../../backup/concepts/backup.md#types) виртуальных машин вместе с загрузочными дисками и восстанавливать ВМ из таких копий. Копии ВМ создаются по принципу application-consistent: сохраняются не только данные на дисках, но и данные, которые были отправлены в запись на диск и еще не успели записаться.

{% include [av-note](../../_includes/backup/av-note.md) %}

С помощью {{ backup-name }} из резервной копии можно [восстановить](../../backup/operations/backup-vm/recover-file-by-file.md) отдельные файлы и директории.

Чтобы включить на ВМ резервное копирование, [подключите](../../backup/concepts/vm-connection.md) эту ВМ к сервису {{ backup-name }} и [привяжите](../../backup/operations/policy-vm/update.md#update-vm-list) ее к [политике](../../backup/concepts/policy.md) резервного копирования. Операционная система ВМ должна [поддерживаться](https://docs.cyberprotect.ru/ru-RU/CyberBackupCloud/21.06/user/#supported-operating-systems-and-environments.html) провайдером.

{% include [backup-image-snapshot-comparison](../../_includes/backup-image-snapshot-comparison.md) %}


## Примеры использования {#examples}

* [{#T}](../tutorials/hystax-backup.md)
* [{#T}](../../tutorials/archive/vm-with-backup-policy/index.md)