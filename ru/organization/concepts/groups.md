---
title: Группы пользователей
description: Вы можете объединять пользователей организации в группы, чтобы назначить одинаковую роль нескольким пользователям одновременно.
---

# Группы пользователей

Для [организаций](organization.md), в которых много [участников](membership.md), одинаковые права доступа к ресурсам {{ yandex-cloud }} могут потребоваться сразу нескольким пользователям. В этом случае роли и доступы удобнее выдавать не персонально, а для группы. Вы можете [настроить доступ](../operations/access-group.md) участников группы к организациям, облакам, каталогам и сервисным аккаунтам {{ yandex-cloud }}.

Другие пользователи смогут [управлять группой](../operations/access-manage-group.md), если вы назначите им соответствующие [роли](../security/index.md#service-roles). Например, `organization-manager.groups.memberAdmin` для просмотра информации и добавления участников.

Кроме групп, создаваемых администратором, в {{ yandex-cloud }} существуют [системные группы](../../iam/concepts/access-control/system-group.md) `All users in organization X` и `All users in federation N`, а также [публичные группы](../../iam/concepts/access-control/public-group.md) `All authenticated users` и `All users`.

Группы поддерживают только одноуровневую структуру. Нельзя создать группу внутри группы, а членство в группе дает равные права всем участникам.

Если вы используете группы пользователей в вашем поставщике удостоверений при работе с [федерациями](add-federation.md), вы можете [сопоставить группы](add-federation.md#group-mapping) в поставщике удостоверений и в {{ org-name }}.

## Примеры использования {#examples}

* [{#T}](../tutorials/user-group-access-control.md)
* [{#T}](../tutorials/federations/group-mapping/adfs.md)
* [{#T}](../tutorials/federations/group-mapping/entra-id.md)
* [{#T}](../tutorials/federations/group-mapping/keycloak.md)
* [{#T}](../../tutorials/security/integration-azure.md)

#### Что дальше {#what-is-next}

* [Управление группами пользователей](../operations/manage-groups.md).
* [Квоты и лимиты](limits.md).
