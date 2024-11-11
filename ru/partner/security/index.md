# Управление доступом

## Какие роли действуют в сервисе {#roles-list}

### Сервисные роли {#service-roles}

#### billing.accounts.owner {#billing-accounts-owner}

{% include [billing.accounts.owner](../../_roles/billing/accounts/owner.md) %}

#### billing.accounts.viewer {#billing-accounts-viewer}

{% include [billing.accounts.viewer](../../_roles/billing/accounts/viewer.md) %}

#### billing.accounts.accountant {#billing-accounts-accountant}

{% include [billing.accounts.accountant](../../_roles/billing/accounts/accountant.md) %}

#### billing.accounts.editor {#billing-accounts-editor}

{% include [billing.accounts.editor](../../_roles/billing/accounts/editor.md) %}

#### billing.accounts.varWithoutDiscounts {#billing-accounts-var-without-discounts}

{% include [billing.accounts.varwithoutdiscounts](../../_roles/billing/accounts/varWithoutDiscounts.md) %}

#### billing.accounts.admin {#billing-accounts-admin}

{% include [billing.accounts.admin](../../_roles/billing/accounts/admin.md) %}

### Примитивные роли {#primitive-roles}

Примитивные роли — роли агрегаторы, определяющие разрешения пользователя для доступа к сервисам. В {{ billing-name }} эти роли соответствуют ролям `billing.accounts.*`:

* `auditor` — аналогична роли `billing.accounts.viewer` с ограничениями.
* `viewer` — аналогична роли `billing.accounts.viewer`.
* `editor` — аналогична роли `billing.accounts.editor`.
* `admin` — аналогична роли `billing.accounts.admin`.

Примитивные роли могут назначаться только пользователям, добавленным в список **{{ ui-key.yacloud_components.notify-subs.label_users }}**.

## Доступные операции для партнерского аккаунта {#partner-available-operations}

Список доступных операций для ролей каждого типа представлен в таблицах ниже. Полные названия ролей из таблицы начинаются с `billing.accounts.*`.

### Общие операции для аккаунта {#general}

#|
|| **Операции** |
`owner` |
`viewer` |
`accountant` |
`editor` |
`admin` |
`varWithoutDiscounts` ||
|| Посмотреть данные платежного аккаунта |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) ||
|| Привязать облако к аккаунту |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/no.svg) |
![image](../../_assets/common/no.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) ||
|| Управлять правами доступа к платежному аккаунту |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/no.svg) |
![image](../../_assets/common/no.svg) |
![image](../../_assets/common/no.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) ||
|| Активировать платную версию |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/no.svg) |
![image](../../_assets/common/no.svg) |
![image](../../_assets/common/no.svg) |
![image](../../_assets/common/no.svg) |
![image](../../_assets/common/no.svg) ||
|| Удалить платежный аккаунт |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/no.svg) |
![image](../../_assets/common/no.svg) |
![image](../../_assets/common/no.svg) |
![image](../../_assets/common/no.svg) |
![image](../../_assets/common/no.svg) ||
|#

### Операции для программы прямого партнерства {#var-operations}

#|
|| **Операции** |
`owner` |
`viewer` |
`accountant` |
`editor` |
`admin` |
`varWithoutDiscounts` ||
|| Просмотреть список клиентов (сабаккаунтов) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) ||
|| Посмотреть потребление сервисов клиентом |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) ||
|| Посмотреть историю начисления рибейта |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/no.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) ||
|| Вывести рибейт |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/no.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) ||
|| Создать запись о клиенте (сабаккаунт) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/no.svg) |
![image](../../_assets/common/no.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) ||
|| Приостановить сабаккаунт |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/no.svg) |
![image](../../_assets/common/no.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) ||
|| Возобновить работу сабаккаунта |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/no.svg) |
![image](../../_assets/common/no.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) ||
|| Посмотреть присвоенные специализации |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/no.svg) ||
|| Посмотреть список партнерских премий и информацию о них |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/no.svg) ||
|| Обновить данные записи о клиенте (сабаккаунта) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/no.svg) |
![image](../../_assets/common/no.svg) |
![image](../../_assets/common/no.svg) |
![image](../../_assets/common/no.svg) |
![image](../../_assets/common/no.svg) ||
|#

## Операции для реферальной программы {#referral-operations}

#|
|| **Операции** |
`owner` |
`viewer` |
`accountant` |
`editor` |
`admin` |
`varWithoutDiscounts` ||
|| Посмотреть историю начисления вознаграждений по реферальной программе |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) ||
|| Посмотреть статус расчетов с компанией-реферером: начислено, выведено, и остаток |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) ||
|| Создать реферальную ссылку |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/no.svg) |
![image](../../_assets/common/no.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) ||
|| Изменить реферальную ссылку |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/no.svg) |
![image](../../_assets/common/no.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) ||
|| Активировать реферальную ссылку |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/no.svg) |
![image](../../_assets/common/no.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) ||
|| Вывести вознаграждение по реферальной программе |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/no.svg) |
![image](../../_assets/common/no.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) ||
|| Посмотреть список реферальных ссылок |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/no.svg) ||
|#

## Доступные операции для сабаккаунта {#subaccount-available-operations}

Список доступных операций для ролей каждого типа представлен в таблице ниже. Полные названия ролей из таблицы начинаются с `billing.accounts.*`.

#|
|| **Операции** |
`customer`&nbsp;^1^ |
`owner` |
`viewer` |
`accountant` |
`editor` |
`admin` |
`varWithoutDiscounts` ||
|| Посмотреть данные сабаккаунта |
![image](../../_assets/common/no.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) ||
|| Привязать облако к аккаунту |
![image](../../_assets/common/no.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/no.svg) |
![image](../../_assets/common/no.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) ||
|| Активировать сабаккаунт |
![image](../../_assets/common/no.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/no.svg) |
![image](../../_assets/common/no.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) ||
|| Управлять правами доступа к сабаккаунту |
![image](../../_assets/common/no.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/no.svg) |
![image](../../_assets/common/no.svg) |
![image](../../_assets/common/no.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/yes.svg) ||
|| Удалить сабаккаунт (до подтверждения клиентом) |
![image](../../_assets/common/no.svg) |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/no.svg) |
![image](../../_assets/common/no.svg) |
![image](../../_assets/common/no.svg) |
![image](../../_assets/common/no.svg) |
![image](../../_assets/common/no.svg) ||
|| Принять приглашение от партнера |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/no.svg) |
![image](../../_assets/common/no.svg) |
![image](../../_assets/common/no.svg) |
![image](../../_assets/common/no.svg) |
![image](../../_assets/common/no.svg) |
![image](../../_assets/common/no.svg) ||
|| Отклонить приглашение от партнера |
![image](../../_assets/common/yes.svg) |
![image](../../_assets/common/no.svg) |
![image](../../_assets/common/no.svg) |
![image](../../_assets/common/no.svg) |
![image](../../_assets/common/no.svg) |
![image](../../_assets/common/no.svg) |
![image](../../_assets/common/no.svg) ||
|#

^1^ Роль `billing.accounts.customer` выдается пользователю автоматически при создании сабаккаунта. Назначить ее вручную нельзя.
