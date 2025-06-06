# Подписать пользователя на уведомления

По умолчанию технические уведомления для организации получают пользователи с ролью `organization-manager.organizations.owner`.

Вы также можете подписать других пользователей организации на получение технических уведомлений. Для этого:

{% list tabs group=instructions %}

- Интерфейс {{ cloud-center }} {#cloud-center}

  1. Войдите в сервис [{{ cloud-center }}]({{ cloud-center-link }}).
  
      Чтобы подписать пользователя на получение технических уведомлений, необходима [роль](../../iam/roles-reference.md#editor) `editor` или выше на организацию.

      {% include [switch-org-note](../../_includes/organization/switch-org-note.md) %}

      На открывшейся главной странице сервиса {{ cloud-center }} приведены основные сведения о вашей организации.

  1. Пролистайте страницу вниз до блока **{{ ui-key.yacloud_org.dashboard.notify.title.notify }}** и нажмите ![PersonPlus](../../_assets/console-icons/person-plus.svg) **{{ ui-key.yacloud_org.dashboard.notify.placeholder_add-user-action }}**. В открывшемся окне:

      1. Выберите пользователя, которого хотите подписать на получение уведомлений, или воспользуйтесь поиском.

          Вы можете подписать на уведомления одновременно нескольких пользователей.
      1. Нажмите кнопку **{{ ui-key.yacloud.common.add }}**.

  Добавленные пользователи появятся в блоке **{{ ui-key.yacloud_org.dashboard.notify.title.notify }}**.

{% endlist %}

Пользователи, которых вы подписали на уведомления, будут получать технические уведомления организации. Настроить типы и каналы получения уведомлений каждый подписанный пользователь может самостоятельно. Подробнее см. в разделе [{#T}](../../resource-manager/concepts/notify.md#notification-settings).