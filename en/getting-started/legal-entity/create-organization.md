# Creating an organization

Before getting started with {{ org-full-name }}, [log in]({{ link-passport-login }}) to your Yandex account.

{% list tabs group=instructions %}

- {{ cloud-center }} interface {#cloud-center}

  {% include [create-new-org](../../_includes/organization/create-new-org.md) %}

{% endlist %}

After registering, you will become the organization owner. You will be able to manage employee accounts, connect and disconnect services.

## Edit organization data {#change}

{% list tabs group=instructions %}

- {{ cloud-center }} interface {#cloud-center}

  1. Log in to [{{ cloud-center }}]({{ cloud-center-link }}) with an administrator or organization owner account.

      On the {{ cloud-center }} home page that opens, you can view general info about your organization.
  
  1. To update your organization data, click ![pencil](../../_assets/console-icons/pencil.svg) **{{ ui-key.yacloud_org.dashboard.organization.action.edit-button }}** in the box with its name in the center of the screen.

  1. In the window that opens, edit the organization's name, technical name, and description or specify employee count.

  1. Click **{{ ui-key.yacloud_org.forms.action.save }}**.

{% endlist %}

## Switch to another organization {#another-organization}

If you are an administrator or user of multiple organizations, you can switch between them:

{% list tabs group=instructions %}

- {{ cloud-center }} interface {#cloud-center}

  1. Go to [{{ cloud-center }}]({{ cloud-center-link }}).

  1. In the {{ cloud-center }} home page that opens, click ![chevron-down](../../_assets/console-icons/chevron-down.svg) next to the name of the current organization in the top-left corner of the screen and select another one.
  
  This will switch you to a different organization.

- Management console {#console}

  1. Go to the [management console]({{ link-console-main }}) and click your account picture in the left-hand panel.
  1. Select an organization or federation from the list of those available in your account. Available organizations and federations are marked with ![case](../../_assets/console-icons/briefcase.svg).

      Alternatively, you can select an organization or federation belonging to a different Yandex account you previously logged in to. To do this, click the account name in **Login history**.

      If the account you need is not on the list, log in to it by clicking ![image](../../_assets/console-icons/plus.svg) **Logging in to your account**.

  The name of the selected organization is displayed in the top-left corner of the {{ cloud-center }} window, and in the section with the list of clouds and folders.

{% endlist %}

## Creating an additional organization {#create-additional-org}

{% list tabs group=instructions %}

- {{ cloud-center }} interface {#cloud-center}

  1. Log in to [{{ cloud-center }}]({{ cloud-center-link }}) under the account you are going to use to create a new organization.

  1. In the top-left corner, click ![chevron-down](../../_assets/console-icons/chevron-down.svg) next to the name of the current organization and select ![circle-plus](../../_assets/console-icons/circle-plus.svg) **{{ ui-key.yacloud_org.form.organization.create.action.create }}**.

  1. In the window that opens, enter a name and description for your organization.

  1. Click **{{ ui-key.yacloud_org.form.organization.select.action.create }}**.

{% endlist %}