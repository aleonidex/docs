---
title: Инструкции по работе с {{ managed-k8s-full-name }}
description: Из статьи вы узнаете, как работать с управляемым сервисом {{ k8s }} в {{ yandex-cloud }}. Рассказываем, как работать с постоянными томами, управлять группой узлов и кластером {{ k8s }}.
---

# Пошаговые инструкции для {{ managed-k8s-name }}

* [{#T}](node-connect-ssh.md)
* [{#T}](node-connect-oslogin.md)
* [{#T}](update-kubernetes.md)
* [{#T}](autoscale.md)
* [{#T}](apply-tf-provider.md)

## Подключение к кластеру {#cluster-connect}

* [{#T}](connect/index.md)
* [{#T}](connect/security-groups.md)
* [{#T}](connect/create-static-conf.md)

## Установка приложений из {{ marketplace-full-name }} {#marketplace}

* [{#T}](applications/marketplace.md)
* [{#T}](applications/argo-cd.md)
* [{#T}](applications/chaos-mesh.md)
* [{#T}](applications/cert-manager-cloud-dns.md)
* [{#T}](applications/csi-s3.md)
* [{#T}](applications/crossplane.md)
* [{#T}](applications/external-secrets-operator.md)
* [{#T}](applications/externaldns.md)
* [{#T}](applications/falco.md)
* [{#T}](applications/filebeat-oss.md)
* [{#T}](applications/fluentbit.md)
* [{#T}](applications/gatekeeper.md)
* [{#T}](applications/gateway-api.md)
* [{#T}](applications/gitlab-agent.md)
* [{#T}](applications/gitlab-runner.md)
* [{#T}](applications/hashicorp-vault.md)
* [{#T}](applications/ingress-nginx.md)
* [{#T}](applications/alb-ingress-controller.md)
* [{#T}](applications/upgrade-alb-ingress-controller.md)
* [{#T}](applications/istio.md)
* [{#T}](applications/jaeger.md)
* [{#T}](applications/kruise.md)
* [{#T}](applications/kyverno.md)
* [{#T}](applications/loki.md)
* [{#T}](applications/metrics-provider.md)
* [{#T}](applications/node-local-dns.md)
* [{#T}](applications/policy-reporter.md)
* [{#T}](applications/prometheus-operator.md)
* [{#T}](applications/thumbor.md)
* [{#T}](applications/velero-yc-csi.md)

## Сетевые сценарии {#network-scenarios}

* [{#T}](create-load-balancer.md)
* [{#T}](calico.md)
* [{#T}](cilium.md)
* [{#T}](cilium-node-local-dns.md)
* [{#T}](create-load-balancer-with-ingress-nginx.md)

## Работа с постоянными томами {#persistent-volumes}

* [{#T}](volumes/dynamic-create-pv.md)
* [{#T}](volumes/static-create-pv.md)
* [{#T}](volumes/manage-storage-class.md)
* [{#T}](volumes/volume-expansion.md)
* [{#T}](volumes/statefulset-pvc-expansion.md)
* [{#T}](volumes/mode-block.md)
* [{#T}](volumes/s3-csi-integration.md)

## Управление кластером {{ managed-k8s-name }} {#kubernetes-cluster}

* [{#T}](kubernetes-cluster/kubernetes-cluster-list.md)
* [{#T}](kubernetes-cluster/kubernetes-cluster-create.md)
* [{#T}](kubernetes-cluster/kubernetes-cluster-update.md)
* [{#T}](kubernetes-cluster/kubernetes-cluster-namespace-create.md)
* [{#T}](kubernetes-cluster/kubernetes-cluster-delete.md)

## Управление группой узлов {#node-group}

* [{#T}](node-group/node-group-list.md)
* [{#T}](node-group/node-group-create.md)
* [{#T}](node-connect-ssh.md)
* [{#T}](autoscale.md)
* [{#T}](node-group/node-group-update.md)
* [{#T}](node-group/node-label-management.md)
* [{#T}](node-group/node-group-delete.md)


## Работа с внешними узлами {#external-nodes}

* [{#T}](external-nodes-connect.md)

## Получение дополнительной информации {#get-additional-info}

* [{#T}](kubernetes-cluster/kubernetes-cluster-operation-logs.md)