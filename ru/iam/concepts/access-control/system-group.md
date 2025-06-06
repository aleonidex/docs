---
title: Системные группы
description: Из статьи вы узнаете, что такое системные группы и каких они бывают типов.
---

# Системные группы


_Системная группа_ — группа пользователей (субъектов), на которую можно назначать [роли](./roles.md). В {{ yandex-cloud }} существует два типа системных групп: `All users in organization X` и `All users in federation N`. Эти группы позволяют предоставить доступ к вашим ресурсам для заданной группы пользователей, но только на те операции, которые позволяет выполнять назначенная роль. В системные группы не входят [сервисные аккаунты](../users/service-accounts.md).

Системные группы являются динамическими: при добавлении в организацию или федерацию нового пользователя он автоматически получает все права, назначенные на соответствующую организацию или федерацию. При удалении пользователя из организации или федерации, пользователь автоматически такие права теряет.

Небезопасно назначать системным группам роли с широким набором разрешений, такие как `editor` или `admin`.

### All users in organization X {#allOrganizationUsers}

Системная группа `All users in organization X` — это все пользователи [организации](../../../organization/quickstart.md) `X`, к которой относится группа.

При назначении роли системной группе `All users in organization X` с помощью [CLI](../../../cli/quickstart.md), [{{ TF }}]({{ tf-provider-link }}) и [API](../../../api-design-guide/concepts/general.md) используйте идентификатор [субъекта](./index.md#subject) `group:organization:<идентификатор_организации>:users`, где `<идентификатор_организации>` — уникальный идентификатор, присвоенный [организации](../../../organization/quickstart.md) `X`.

### All users in federation N {#allFederationUsers}

Системная группа `All users in federation N` — это все пользователи [федерации удостоверений](../../../organization/concepts/add-federation.md) `N`, к которой относится группа.

При назначении роли системной группе `All users in federation N` с помощью [CLI](../../../cli/quickstart.md), [{{ TF }}]({{ tf-provider-link }}) и [API](../../../api-design-guide/concepts/general.md) используйте идентификатор [субъекта](./index.md#subject) `group:federation:<идентификатор_федерации>:users`, где `<идентификатор_федерации>` — уникальный идентификатор, присвоенный [федерации удостоверений](../../../organization/concepts/add-federation.md) `N`.

## Примеры использования {#examples}

* [{#T}](../../../tutorials/security/integration-adfs.md)
* [{#T}](../../../tutorials/security/integration-azure.md)