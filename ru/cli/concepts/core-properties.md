# Конфигурация CLI

Вы можете [задать конфигурационные параметры](../operations/profile/manage-properties.md) в профиле, чтобы быстро переключать контекст выполнения команд. Для разового запуска команды с определенным значением параметра используйте глобальные параметры.

{% note warning %}

Обратите внимание, что некоторые параметры не могут использоваться одновременно. Значение одного из параметров обнуляется после установки другого.

{% endnote %}

| Глобальный параметр | Параметр профиля | Описание |
|----|----|----|
| `--debug` | — | Вывести журнал отладки при выполнении команды. |
| `--debug-grpc` | — | Вывести журнал отладки gRPC при выполнении команды. Очень подробный, полезен для отладки проблем с подключением. |
| `--cloud-id` | `cloud-id` | [Идентификатор облака](../../resource-manager/operations/cloud/get-id.md), который будет использован при выполнении команды. Применяется только в команде [yc resource-manager folder list](../cli-ref/resource-manager/cli-ref/folder/list.md) или совместно с `folder-name`. В остальных случаях параметр игнорируется. |
| `--folder-id` | `folder-id` | Идентификатор каталога, который будет использован при выполнении команды. При установке данного параметра профиля обнуляется параметр `folder-name`. |
| `--folder-name` | `folder-name` | Имя каталога, который будет использован при выполнении команды. При установке данного параметра профиля обнуляется параметр `folder-id`. |
| `--format` | `format` | Формат, в котором будет представлен вывод: `text` (по умолчанию), `yaml`, `json`, `json-rest`. |
| `-h`, `--help` | — | Вывести справку о команде. |
| `--no-user-output` | — | Исключить вывод, предназначенный пользователю, из потока вывода ошибок `stderr`. |
| `--profile` | — | Имя профиля, параметры которого будут использованы при выполнении команды. |
| — | `service-account-key`| Авторизованный ключ [сервисного аккаунта](../../iam/concepts/users/service-accounts.md) в {{ yandex-cloud }}. Устанавливается из файла, в котором хранится ключ. При установке данного параметра профиля обнуляется параметр `token`. |
| `--token` | `token` | OAuth-токен, который используется для аутентификации в {{ yandex-cloud }}. При установке данного параметра профиля обнуляется параметр `service-account-key`. |
