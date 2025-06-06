---
title: Установка Container Storage Interface для S3
description: Следуя данной инструкции, вы сможете установить Container Storage Interface для S3.
---

# Установка Container Storage Interface для S3


[Container Storage Interface для S3](/marketplace/products/yc/csi-s3) (_CSI_) позволяет динамически резервировать [бакеты](../../../storage/concepts/bucket.md) S3-совместимых хранилищ и монтировать их к [подам](../../concepts/index.md#pod) [кластера {{ managed-k8s-name }}](../../concepts/index.md#kubernetes-cluster) в виде [постоянных томов](../../concepts/volume.md#persistent-volume) (_PersistentVolume_). Подключение выполняется при помощи [FUSE](https://ru.wikipedia.org/wiki/FUSE_(модуль_ядра))-реализации файловой системы [GeeseFS](https://github.com/yandex-cloud/geesefs).

{% include [csi-s3-actual](../../../_includes/managed-kubernetes/csi-s3-actual.md) %}

Установить Container Storage Interface для S3 можно следующими способами:
* [В консоли управления с помощью {{ marketplace-name }}](#marketplace-install)
* [C помощью Helm-чарта из репозитория {{ marketplace-name }}](#helm-install)
* [С помощью Helm-чарта из репозитория на GitHub](#helm-github-install)

## Перед началом работы {#before-you-begin}

1. [Создайте](../../../iam/operations/sa/create.md) сервисный аккаунт с [ролью](../../../storage/security/index.md#storage-editor) `storage.editor`.
1. [Создайте](../../../iam/operations/authentication/manage-access-keys.md#create-access-key) статический ключ доступа для сервисного аккаунта. Сохраните идентификатор ключа и секретный ключ — они понадобятся при установке приложения.
1. (Опционально) Чтобы новые тома помещались в один бакет с разными префиксами, [создайте](../../../storage/operations/buckets/create.md) бакет {{ objstorage-full-name }}. Сохраните имя бакета — оно понадобится при установке приложения. Пропустите этот шаг, если для каждого тома требуется создавать отдельный бакет.

1. {% include [check-sg-prerequsites](../../../_includes/managed-kubernetes/security-groups/check-sg-prerequsites-lvl3.md) %}

    {% include [sg-common-warning](../../../_includes/managed-kubernetes/security-groups/sg-common-warning.md) %}

## Установка в консоли управления с помощью {{ marketplace-name }} {#marketplace-install}

1. Перейдите на [страницу каталога]({{ link-console-main }}) и выберите сервис **{{ ui-key.yacloud.iam.folder.dashboard.label_managed-kubernetes }}**.
1. Нажмите на имя нужного кластера {{ managed-k8s-name }} и выберите вкладку ![image](../../../_assets/console-icons/shopping-cart.svg) **{{ ui-key.yacloud.k8s.cluster.switch_marketplace }}**.
1. В разделе **{{ ui-key.yacloud.marketplace-v2.label_available-products }}** выберите [Container Storage Interface для S3](/marketplace/products/yc/csi-s3) и нажмите кнопку **{{ ui-key.yacloud.marketplace-v2.button_k8s-product-use }}**.
1. Задайте настройки приложения:
   * **Пространство имен** — выберите [пространство имен](../../concepts/index.md#namespace) `kube-system`.

      {% include [Namespace warning](../../../_includes/managed-kubernetes/kube-system-namespace-warning.md) %}

   * **Название приложения** — укажите название приложения, например `csi-s3`.
   * **Создать класс хранения** — выберите эту опцию, чтобы создать новый [класс хранилища](../volumes/manage-storage-class.md) при развертывании приложения.
   * **Создать секрет** — выберите эту опцию, чтобы создать новый секрет для класса хранилища при установке приложения.
   * **Идентификатор ключа S3** — скопируйте в это поле идентификатор ключа сервисного аккаунта.
   * **Секретный ключ S3** — скопируйте в это поле секретный ключ сервисного аккаунта.
   * **Общий бакет S3 для томов** — укажите имя общего бакета, в котором будут создаваться [динамически распределяемые тома](../../concepts/volume.md#dynamic-provisioning). Чтобы CSI создавал новый бакет для каждого тома, оставьте это поле пустым.
   * **Адрес S3-сервиса** — адрес S3-сервиса, который будет использовать приложение. По умолчанию `https://{{ s3-storage-host }}`.
   * **Опции монтирования GeeseFS** — опции монтирования для программы GeeseFS. Полный список опций см. в [документации GeeseFS](https://github.com/yandex-cloud/geesefs).
   * **Политика очистки томов** — выберите политику очистки томов PersistentVolume при удалении PersistentVolumeClaim:
     * **Retain** — сохранять том.
     * **Delete** — удалять том.
   * **Название класса хранения** — если вы ранее выбрали опцию **Создать класс хранения**, укажите имя нового класса хранилища.
   * **Название секрета** — если вы ранее выбрали опцию **Создать секрет**, укажите имя нового секрета, который будет создан для класса хранилища. В ином случае укажите имя существующего секрета, который будет использован для класса хранилища.
   * **Игнорировать все политики taint** — выберите эту опцию, чтобы драйвер CSI, который монтирует файловую систему на узлах, игнорировал все политики taint для узлов кластера {{ managed-k8s-name }}.
1. Нажмите кнопку **{{ ui-key.yacloud.k8s.cluster.marketplace.button_install }}**.
1. Дождитесь перехода приложения в статус `Deployed`.

## Установка с помощью Helm-чарта из репозитория {{ marketplace-name }} {#helm-install}

1. {% include [Установка Helm](../../../_includes/managed-kubernetes/helm-install.md) %}
1. {% include [Install and configure kubectl](../../../_includes/managed-kubernetes/kubectl-install.md) %}
1. Для установки [Helm-чарта](https://helm.sh/docs/topics/charts/) с CSI выполните команду, указав в ней параметры ключа, созданного [ранее](#before-you-begin):

   ```bash
   helm pull oci://{{ mkt-k8s-key.yc_csi-s3.helmChart.name }} \
     --version {{ mkt-k8s-key.yc_csi-s3.helmChart.tag }} \
     --untar && \
   helm install \
     --namespace kube-system \
     --set secret.accessKey=<идентификатор_ключа> \
     --set secret.secretKey=<секретный_ключ> \
     csi-s3 ./csi-s3/
   ```

   {% include [Support OCI](../../../_includes/managed-kubernetes/note-helm-experimental-oci.md) %}

   Также вы можете [задать дополнительные параметры](#installation-parameters) Container Storage Interface для S3.

## Установка с помощью Helm-чарта из репозитория на GitHub {#helm-github-install}

В [репозитории на GitHub](https://github.com/yandex-cloud/k8s-csi-s3) размещается наиболее актуальная версия Container Storage Interface для S3 с поддержкой {{ objstorage-name }}.

1. {% include [Установка Helm](../../../_includes/managed-kubernetes/helm-install.md) %}
1. {% include [Install and configure kubectl](../../../_includes/managed-kubernetes/kubectl-install.md) %}
1. Для установки [Helm-чарта](https://helm.sh/docs/topics/charts/) с CSI выполните команду:

    ```bash
    helm repo add yandex-s3 https://yandex-cloud.github.io/k8s-csi-s3/charts && \
    helm repo update && \
    helm pull yandex-s3/csi-s3 --untar && \
    helm install \
      --namespace kube-system \
      --set secret.accessKey=<идентификатор_ключа> \
      --set secret.secretKey=<секретный_ключ> \
      csi-s3 ./csi-s3/
    ```

    Также вы можете [задать дополнительные параметры](#installation-parameters) Container Storage Interface для S3.

## Параметры для установки с помощью Helm-чарта {#installation-parameters}

При установке приложения Container Storage Interface для S3 обязательными являются параметры `secret.accessKey` и `secret.secretKey`. Вы можете не указывать остальные параметры, либо переопределить их в команде установки с помощью ключа `--set <имя_параметра>=<новое_значение>`.

Список доступных для переопределения параметров и их значения по умолчанию приведены в таблице:

Имя параметра | Описание | Значение по умолчанию
--- | --- | ---
`storageClass.create` | Требуется ли создание нового класса хранилища | `true`
`storageClass.name` | Имя класса хранилища | `csi-s3`
`storageClass.singleBucket` | Использовать один бакет для всех PersistentVolumeClaim |
`storageClass.mountOptions` | Опции монтирования GeeseFS | `--memory-limit 1000 --dir-mode 0777 --file-mode 0666`
`storageClass.reclaimPolicy` | Политика очистки томов | `Delete`
`storageClass.annotations` | Описание для класса хранилища |
`secret.create` | Требуется ли создание нового секрета | `true`
`secret.name` | Имя секрета | `csi-s3-secret`
`secret.accessKey` | Идентификатор ключа |
`secret.secretKey` | Секретный ключ |
`secret.endpoint` | Адрес S3-сервиса | `https://{{ s3-storage-host }}`

## См. также {#see-also}

* [Спецификация CSI](https://github.com/container-storage-interface/spec/blob/master/spec.md)
* [Container Storage Interface для S3 с поддержкой {{ objstorage-name }} на GitHub](https://github.com/yandex-cloud/k8s-csi-s3)
* [Интеграция с {{ objstorage-name }}](../volumes/s3-csi-integration.md)
* [Примеры использования CSI](../volumes/s3-csi-integration.md#examples)
* [Работа с постоянными и динамическими томами в {{ k8s }}](../../concepts/volume.md)
