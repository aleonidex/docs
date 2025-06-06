---
title: Спецификации Docker-контейнера в {{ cos-full-name }}
description: Описать конфигурацию запуска Docker-контейнера можно с помощью COI- или Docker Compose-спецификации.
---

# Спецификации Docker-контейнера

Для описания конфигурации запуска [Docker-контейнера](/blog/posts/2022/03/docker-containers) существует два способа:
* [COI-спецификация](#coi-spec), позволяет запустить только один Docker-контейнер.
* С помощью [Docker Compose-спецификации](#compose-spec) можно задать более гибкую конфигурацию, например запуск нескольких Docker-контейнеров и необходимых служб.

{% note warning %}

Одновременно может использоваться только одна из спецификаций: COI или Docker Compose.

{% endnote %}

## COI-спецификация {#coi-spec}

Docker-контейнер в {{ coi }} описывается в спецификации (YAML-файле), основанной на [спецификации подов {{ k8s }}](https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/pod-v1/).

При [создании виртуальной машины на базе {{ coi }}](../tutorials/vm-create.md) из консоли управления или CLI спецификация генерируется автоматически на основе указанных данных. Для [создания группы ВМ на базе {{ coi }}](../tutorials/ig-create.md) спецификацию необходимо составить вручную. Пример спецификации и необходимые ключи описаны ниже.

### Пример COI-спецификации Docker-контейнера {#coi-spec-example}

COI-спецификация – это YAML-файл со следующим содержанием:

```yaml
spec:
  containers:
  - command:
    - sleep
    args:
    - 100000
    env:
    - name: MYENV
      value: myvalue
    image: {{ registry }}/mirror/ubuntu:16.04
    name: my-container
    securityContext:
      privileged: false
    stdin: false
    tty: false
    volumeMounts:
      - mountPath: /home/yc-user/cache
        name: cache-volume
      - mountPath: /home/yc-user/data
        name: data-volume
  restartPolicy: Always
  volumes:
    - name: cache-volume
      emptyDir:
        medium: Memory
    - name: data-volume
      hostPath:
        path: /data
```

Где:
* `command` – команда, выполняемая при запуске Docker-контейнера.
* `args` – аргументы, передаваемые запускаемой в Docker-контейнере команде.
* `env` – переменные окружения, доступные внутри Docker-контейнера.
* `image` – имя Docker-образа, на основе которого будет создан и запущен Docker-контейнер.
* `name` – имя запускаемого Docker-контейнера.
* `securityContext` – настройки безопасности и контроля доступа внутри Docker-контейнера. Поддерживается только возможность запуска привилегированного Docker-контейнера.
* `privileged` – запуск Docker-контейнера в привилегированном режиме. Процессы в привилегированных Docker-контейнерах получают доступ ко всем системным устройствам и эквивалентны использованию root-прав на ВМ. Значение по умолчанию — false.
* `stdin` – буфер для потока ввода во время выполнения Docker-контейнера. Поток ввода будет связан с запущенным Docker-контейнером. Значение по умолчанию — false.
* `tty` – выделение TTY для Docker-контейнера. Значение по умолчанию - false.
* `restartPolicy` – [политика перезапуска](restart-policy.md) Docker-контейнера.
* `volumeMounts` – список томов для монтирования внутри Docker-контейнера.
* `mountPath` – путь в Docker-контейнере, по которому будет смонтирован указанный том.
* `volumes` – описание томов, используемых в спецификации.
* `emptyDir` – пустая директория во временной файловой системе `tmpfs`, которая создается в оперативной памяти ВМ. Содержимое этой директории удаляется при остановке Docker-контейнера, в который она смонтирована, а также при перезагрузке ВМ. Для использования `tmpfs` необходимо указать параметр `medium: Memory`. Размер тома ограничен количеством оперативной памяти, выделенной для ВМ.
* `hostPath` – директория из файловой системы ВМ, которая будет смонтирована в Docker-контейнер.
* `path` – путь к директории `hostPath`.

### Примеры использования {#examples-coi-spec}

* [{#T}](../tutorials/coi-with-terraform.md)
* [{#T}](../tutorials/serial-port.md)

## Docker Compose-спецификация {#compose-spec}

Инструкции для запуска Docker-контейнеров и конфигурации сервисов указываются в файле спецификации `docker-compose.yaml`, согласно [документации Docker](https://docs.docker.com/compose/compose-file/).

Подробнее о запуске нескольких Docker-контейнеров читайте в разделе [{#T}](../tutorials/docker-compose.md).

### Пример Docker Compose-спецификации {#compose-spec-example}

Docker Compose-спецификация – это YAML-файл `docker-compose.yaml` со следующим содержанием:

```yaml
version: '3.7'
services:
  app1:
    container_name: nginx
    image: "nginx"
    ports:
      - "80:80"
    restart: always
  app2:
    container_name: redis
    image: "redis"
    restart: always
    volumes:
      - /mnt/logs:/logs
      - /mnt/data:/data

x-yc-disks:
  - device_name: compute-disk-data
    fs_type: ext4
    host_path: /mnt/data
    partition: 1
  - device_name: compute-disk-data
    fs_type: ext4
    host_path: /mnt/logs
    partition: 2
```

Где:
* `version` – тег версии спецификации, с которого должен начинаться файл.
* `services` – раздел, в котором описываются сервисы.
* `container_name` – имя запускаемого Docker-контейнера.
* `image` – имя Docker-образа, на основе которого будет создан и запущен Docker-контейнер.
* `ports` – используется для перенаправления портов сервиса. Указывается в виде: `<порт_компьютера>:<порт_контейнера>`.
* `restart` – настройка политики перезапуска Docker-контейнера.
* `volumes` – описание томов, используемых в Docker-контейнере.
* `x-yc-disks` – раздел, в котором описываются подключаемые [диски](../../compute/concepts/disk.md). Представляет собой [расширение спецификации Docker Compose](https://docs.docker.com/compose/compose-file/#extension-fields). Используется при подготовке к запуску Docker-контейнеров, перед запуском Docker Compose. Docker Compose пропускает этот раздел.
* `device_name` – имя устройства, отличается от имени диска. Используется на ВМ для поиска диска по дереву `/dev/disk/by-id/virtio-<device_name>`. Указывается в параметрах и командах CLI по добавлению дисков на ВМ при помощи параметра `device-name` или в консоли управления при [подключении](../../compute/operations/vm-control/vm-attach-disk#attach) диска к ВМ.
* `fs_type` – тип файловой системы. Поддерживаются файловые системы ext4 и xfs.
* `host_path` – директория, в которую монтируется диск.
* `partition` – используемый раздел диска.

### Примеры использования {#examples-compose-spec}

* [{#T}](../tutorials/docker-compose.md)
* [{#T}](../tutorials/vm-create-with-second-disk.md)
* [{#T}](../tutorials/coi-fluent-bit-logging.md)
