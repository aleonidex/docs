---
title: Installing ExternalDNS with a plugin for {{ dns-full-name }}
description: Follow this guide to install ExternalDNS with a {{ dns-name }} plugin.
---

# Installing ExternalDNS with a plugin for {{ dns-full-name }}


[ExternalDNS](https://github.com/kubernetes-sigs/external-dns) allows you to create [DNS records](../../../dns/concepts/resource-record.md) in [{{ dns-full-name }}](../../../dns/) automatically. ExternalDNS makes {{ k8s }} resources discoverable through public DNS servers.

## Getting started {#before-you-begin}

1. {% include [cli-install](../../../_includes/cli-install.md) %}

   {% include [default-catalogue](../../../_includes/default-catalogue.md) %}

1. [Create a service account](../../../iam/operations/sa/create.md) for {{ dns-name }}.
1. [Assign](../../../iam/operations/sa/assign-role-for-sa.md) it the `dns.editor` role.
1. [Create an authorized key](../../../iam/operations/authentication/manage-authorized-keys.md#create-authorized-key) for the [service account](../../../iam/concepts/users/service-accounts.md) in JSON format and save it to the `key.json` file:

   ```bash
   yc iam key create \
     --service-account-name <service_account_name> \
     --format json \
     --output key.json
   ```

1. {% include [check-sg-prerequsites](../../../_includes/managed-kubernetes/security-groups/check-sg-prerequsites-lvl3.md) %}

    {% include [sg-common-warning](../../../_includes/managed-kubernetes/security-groups/sg-common-warning.md) %}

## Installation using {{ marketplace-full-name }} {#marketplace-install}

1. Navigate to the [folder dashboard]({{ link-console-main }}) and select **{{ ui-key.yacloud.iam.folder.dashboard.label_managed-kubernetes }}**.
1. Click the name of the [{{ managed-k8s-name }} cluster](../../concepts/index.md#kubernetes-cluster) you need and select the ![image](../../../_assets/console-icons/shopping-cart.svg) **{{ ui-key.yacloud.k8s.cluster.switch_marketplace }}** tab.
1. Under **{{ ui-key.yacloud.marketplace-v2.label_available-products }}**, select [ExternalDNS with a plugin for {{ dns-full-name }}](/marketplace/products/yc/externaldns) and click **{{ ui-key.yacloud.marketplace-v2.button_k8s-product-use }}**.
1. Configure the application:
   * **Namespace**: Create a new [namespace](../../concepts/index.md#namespace), e.g., `externaldns-space`. If you leave the default namespace, ExternalDNS may work incorrectly.
   * **Application name**: Specify the application name.
   * **Service account key**: Paste the contents of the `key.json` file or create a new authorized key for the service account. The service account must have the `dns.editor` [role](../../../iam/concepts/access-control/roles.md).
   * **Folder ID**: Specify the [folder](../../../resource-manager/concepts/resources-hierarchy.md#folder) hosting the [{{ dns-name }} zone](../../../dns/concepts/dns-zone.md).
1. Click **{{ ui-key.yacloud.k8s.cluster.marketplace.button_install }}**.
1. Wait for the application to change its status to `Deployed`.

## Installation using a Helm chart {#helm-install}

1. {% include [helm-install](../../../_includes/managed-kubernetes/helm-install.md) %}

1. {% include [Install and configure kubectl](../../../_includes/managed-kubernetes/kubectl-install.md) %}

1. To install a [Helm chart](https://helm.sh/docs/topics/charts/) with ExternalDNS, run the following command:

   ```bash
   helm pull oci://{{ mkt-k8s-key.yc_externaldns.helmChart.name }} \
     --version {{ mkt-k8s-key.yc_externaldns.helmChart.tag }} \
     --untar && \
   helm install \
     --namespace <namespace> \
     --create-namespace \
     --set config.folder_id=<ID_of_folder_with_DNS_zone> \
     --set-file config.auth.json=<path_to_file_with_service_account_authorized_key> \
     externaldns ./externaldns/
    ```

    If you set `namespace` to the default namespace, ExternalDNS may work incorrectly. We recommend you to specify a value different from all existing namespaces (e.g., `externaldns-space`).

    {% include [Support OCI](../../../_includes/managed-kubernetes/note-helm-experimental-oci.md) %}

## Operation specifics {#features}

To automatically create DNS records using [ExternalDNS with a plugin for {{ dns-full-name }}](/marketplace/products/yc/externaldns):
* If you have an [Ingress controller](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/) installed, no additional setup is required.
* For [LoadBalancer](https://kubernetes.io/docs/concepts/services-networking/service/#loadbalancer) type services, use this annotation: `"external-dns.alpha.kubernetes.io/hostname=<your_domain>"`.

  To set the TTL of a DNS record, use this annotation: `"external-dns.alpha.kubernetes.io/ttl=<TTL_in_seconds>"`.

## Use cases {#examples}

* [{#T}](../../tutorials/alb-ingress-controller-log-options.md).
* [{#T}](../../tutorials/load-testing-grpc-autoscaling.md).

## See also {#see-also}

* [ExternalDNS documentation](https://github.com/kubernetes-sigs/external-dns/)
