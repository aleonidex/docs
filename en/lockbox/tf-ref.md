---
title: '{{ TF }} reference for {{ lockbox-full-name }}'
description: This page provides information about {{ TF }} resources and data sources supported by {{ lockbox-name }}.
---

# {{ TF }} reference for {{ lockbox-full-name }}

{% include [terraform-ref-intro](../_includes/terraform-ref-intro.md) %}

## Resources {#resources}

{{ lockbox-name }} supports the following {{ TF }} resources :

| **{{ TF }} resource** | **{{ yandex-cloud }} resource** |
| --- | --- |
| [yandex_lockbox_secret]({{ tf-provider-resources-link }}/lockbox_secret) | [Secret](./concepts/secret.md#secret) |
| [yandex_lockbox_secret_iam_binding]({{ tf-provider-resources-link }}/lockbox_secret_iam_binding) | [Assigning](../iam/concepts/access-control/index.md#access-bindings) access permissions for a secret. <br>We recommend using `yandex_lockbox_secret_iam_member` instead of `yandex_lockbox_secret_iam_binding`. |
| [yandex_lockbox_secret_iam_member]({{ tf-provider-resources-link }}/lockbox_secret_iam_member) | [Assigning](../iam/concepts/access-control/index.md#access-bindings) access permissions for a secret |
| [yandex_lockbox_secret_version]({{ tf-provider-resources-link }}/lockbox_secret_version) | [Secret version](./concepts/secret.md#version)<br> We recommend using `lockbox_secret_version_hashed` instead of `lockbox_secret_version`. |
| [yandex_lockbox_secret_version_hashed]({{ tf-provider-resources-link }}/lockbox_secret_version_hashed) | [Secret version](./concepts/secret.md#version), stores values in {{ TF }} state in hashed format. <br> Storing your data in like this is more secure than storing it openly. <br> May contain a maximum of 10 key-value pairs. |

## Data sources {#data-sources}

{{ lockbox-name }} supports the following {{ TF }} data sources:

| **{{ TF }} data source** | **Description** |
| --- | --- |
| [yandex_lockbox_secret]({{ tf-provider-datasources-link }}/lockbox_secret) | Information about a [secret](./concepts/secret.md#secret) |
| [yandex_lockbox_secret_version]({{ tf-provider-datasources-link }}/lockbox_secret_version) | Information about a [secret version](./concepts/secret.md#version) |