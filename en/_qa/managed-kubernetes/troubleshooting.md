# Troubleshooting in {{ managed-k8s-name }}

This section describes typical problems you may encounter while using {{ managed-k8s-name }} and gives troubleshooting recommendations.

#### Error creating a cluster in a different folder's cloud network {#neighbour-catalog-permission-denied}

Error message:

```text
Permission denied
```

The error occurs when the [resource service account](../../managed-kubernetes/security/index.md#sa-annotation) has no required [roles](../../iam/concepts/access-control/roles.md) in the [folder](../../resource-manager/concepts/resources-hierarchy.md#folder) whose [cloud network](../../vpc/concepts/network.md#network) is selected when creating a cluster.

To create a [{{ managed-k8s-name }} cluster](../../managed-kubernetes/concepts/index.md#kubernetes-cluster) in a cloud network of another folder, [assign](../../iam/operations/sa/assign-role-for-sa.md) the resource [service account](../../iam/concepts/users/service-accounts.md) the following roles in this folder:
* [{{ roles-vpc-private-admin }}](../../vpc/security/index.md#vpc-private-admin)
* [{{ roles-vpc-user }}](../../vpc/security/index.md#vpc-user)

To use a [public IP address](../../vpc/concepts/address.md#public-addresses), also [assign](../../iam/operations/sa/assign-role-for-sa.md) the [{{ roles-vpc-public-admin }}](../../vpc/security/index.md#vpc-public-admin) role.

#### A namespace has been deleted but its status is _Terminating_ and its deletion is not completed {#namespace-terminating}

This happens when a [namespace](../../managed-kubernetes/concepts/index.md#namespace) has stuck resources that cannot be deleted by the namespace controller.

To fix the issue, delete the stuck resources manually.

{% list tabs %}

- CLI

  {% include [cli-install](../../_includes/cli-install.md) %}

  {% include [default-catalogue](../../_includes/default-catalogue.md) %}

  1. [Connect to the {{ managed-k8s-name }} cluster](../../managed-kubernetes/operations/connect/index.md).
  1. Get a list of resources that remain within the namespace:

     ```bash
     kubectl api-resources --verbs=list --namespaced --output=name \
       | xargs --max-args=1 kubectl get --show-kind \
       --ignore-not-found --namespace=<namespace>
     ```

  1. Delete the resources found:

     ```bash
     kubectl delete <resource_type> <resource_name> --namespace=<namespace>
     ```

  That being done, if the namespace is still in the `Terminating` status and cannot be deleted, delete it forcibly using `finalizer`:
  1. Enable {{ k8s }} API proxy to your local computer:

     ```bash
     kubectl proxy
     ```

  1. Delete the namespace:

     ```bash
     kubectl get namespace <namespace> --output=json \
       | jq '.spec = {"finalizers":[]}' > temp.json && \
     curl --insecure --header "Content-Type: application/json" \
       --request PUT --data-binary @temp.json \
       127.0.0.1:8001/api/v1/namespaces/<namespace>/finalize
     ```

    We do not recommend deleting the namespace with the `Terminating` status using `finalizer` right away, as this may cause the stuck resources to remain in your {{ managed-k8s-name }} cluster.

{% endlist %}

#### I am using {{ network-load-balancer-full-name }} alongside an Ingress controller. Why are some of my cluster's nodes UNHEALTHY? {#nlb-ingress}

This is normal behavior for a [load balancer](../../network-load-balancer/concepts/index.md) with `External Traffic Policy: Local` enabled. Only the [{{ managed-k8s-name }} nodes](../../managed-kubernetes/concepts/index.md#node-group) whose [pods](../../managed-kubernetes/concepts/index.md#pod) are ready to accept user traffic get the `HEALTHY` status. The rest of the nodes are labeled as `UNHEALTHY`.

To find out the policy type of a load balancer created using a `LoadBalancer` type service, run this command:

```bash
kubectl describe svc <LoadBalancer_type_service_name> \
| grep 'External Traffic Policy'
```

For more information, see [Parameters of a LoadBalancer service](../../managed-kubernetes/operations/create-load-balancer.md#advanced).

#### Why is a created PersistentVolumeClaim still pending? {#pvc-pending}

This is normal for a [PersistentVolumeClaim](../../managed-kubernetes/concepts/volume.md#persistent-volume) (PVC). The created PVC remains in the **Pending** status until you create a pod that must use it.

To change the PVC status to **Running**:
1. View details of the PVC:

   ```bash
   kubectl describe pvc <PVC_name> \
     --namespace=<namespace_PVC_resides_in>
   ```

   A message saying `waiting for first consumer to be created before binding` means that the PVC is waiting for a pod to be created.
1. [Create a pod](../../managed-kubernetes/operations/volumes/dynamic-create-pv.md#create-pod) for this PVC.

#### Why does my {{ managed-k8s-name }} cluster fail to run after I change its node configuration? {#not-starting}

Make sure the new configuration of {{ managed-k8s-name }} nodes is within the [quota](../../managed-kubernetes/concepts/limits.md):

{% list tabs %}

- CLI

  {% include [cli-install](../../_includes/cli-install.md) %}

  {% include [default-catalogue](../../_includes/default-catalogue.md) %}

  To run diagnostics for your {{ managed-k8s-name }} cluster nodes:
  1. [Connect to the {{ managed-k8s-name }} cluster](../../managed-kubernetes/operations/connect/index.md).
  1. Check the health of {{ managed-k8s-name }} nodes:

     ```bash
     yc managed-kubernetes cluster list-nodes <cluster_ID>
     ```

     A message saying that the allowed amount of {{ managed-k8s-name }} cluster resources has been exceeded is displayed in the first column of the command output. For example:

     ```text
     +--------------------------------+-----------------+------------------+-------------+--------------+
     |         CLOUD INSTANCE         | KUBERNETES NODE |     RESOURCES    |     DISK    |    STATUS    |
     +--------------------------------+-----------------+------------------+-------------+--------------+
     | fhmil14sdienhr5uh89no          |                 | 2 100% core(s),  | 64.0 GB hdd | PROVISIONING |
     | CREATING_INSTANCE              |                 | 4.0 GB of memory |             |              |
     | [RESOURCE_EXHAUSTED] The limit |                 |                  |             |              |
     | on total size of network-hdd   |                 |                  |             |              |
     | disks has exceeded.,           |                 |                  |             |              |
     | [RESOURCE_EXHAUSTED] The limit |                 |                  |             |              |
     | on total size of network-hdd   |                 |                  |             |              |
     | disks has exceeded.            |                 |                  |             |              |
     +--------------------------------+-----------------+------------------+-------------+--------------+
     ```

{% endlist %}

To run your {{ managed-k8s-name }} cluster, [increase the quotas](../../managed-kubernetes/concepts/limits.md).

#### An error occurs when renewing an Ingress controller certificate {#ingress-certificate}

Error message:

```text
ERROR controller-runtime.manager.controller.ingressgroup Reconciler error
{"name": "some-prod", "namespace": , "error": "rpc error: code = InvalidArgument
desc = Validation error:\nlistener_specs[1].tls.sni_handlers[2].handler.certificate_ids:
Number of elements must be less than or equal to 1"}
```

The error occurs if different certificates are specified for the same Ingress controller listener.

**Solution**: Edit and apply the Ingress controller specifications making sure that only one certificate is specified in each listener's description.

#### Why is DNS name resolution not working in my cluster? {#not-resolve-dns}

There may be no name resolution for internal and external DNS queries in a {{ managed-k8s-name }} cluster for several reasons. To fix the issue:
1. [Check the version of your {{ managed-k8s-name }} cluster and node groups](#check-version).
1. Make sure that [CoreDNS](#check-coredns) is up and running.
1. [Make sure the {{ managed-k8s-name }} cluster has enough CPU resources available](#check-cpu).
1. [Set up autoscaling](#dns-autoscaler).
1. [Set up local DNS caching](#node-local-dns).

##### Check the version of your cluster and node groups {#check-version}

1. Get a list of current {{ k8s }} versions:

   ```bash
   yc managed-kubernetes list-versions
   ```

1. Find out the {{ managed-k8s-name }} cluster version:

   ```bash
   yc managed-kubernetes cluster get <cluster_name_or_ID> | grep version:
   ```

   You can get the {{ managed-k8s-name }} cluster ID and name with a [list of clusters in the folder](../../managed-kubernetes/operations/kubernetes-cluster/kubernetes-cluster-list.md#list).
1. Find out the {{ managed-k8s-name }} node group version:

   ```bash
   yc managed-kubernetes node-group get <node_group_name_or_ID> | grep version:
   ```

   You can get the ID and name of the {{ managed-k8s-name }} node group with a [list of node groups in your cluster](../../managed-kubernetes/operations/node-group/node-group-list.md#list).
1. If the versions of your {{ managed-k8s-name }} cluster and node groups are not on the list of current {{ k8s }} versions, [upgrade them](../../managed-kubernetes/operations/update-kubernetes.md).

##### Make sure that CoreDNS is up and running {#check-coredns}

Get a list of CoreDNS pods and their statuses:

```bash
kubectl get pods -n kube-system -l k8s-app=kube-dns -o wide
```

Make sure all the pods have the `Running` status.

##### Make sure the cluster has enough CPU resources available {#check-cpu}

1. Navigate to the [folder dashboard]({{ link-console-main }}) and select **{{ ui-key.yacloud.iam.folder.dashboard.label_managed-kubernetes }}**.
1. Click the name of the {{ managed-k8s-name }} cluster you need and select the **{{ ui-key.yacloud.k8s.cluster.switch_nodes-manager }}** tab.
1. Go to the **{{ ui-key.yacloud.k8s.nodes.label_nodes }}** tab and click the name of any {{ managed-k8s-name }} node.
1. Go to the **{{ ui-key.yacloud.k8s.node.overview.label_monitoring }}** tab.
1. Make sure that, in the **CPU, [cores]** chart, the `used` CPU values have not reached the `total` available CPU values. Check this for each {{ managed-k8s-name }} cluster node.

##### Set up autoscaling {#dns-autoscaler}

Set up [automatic DNS scaling by {{ managed-k8s-name }} cluster size](../../managed-kubernetes/tutorials/dns-autoscaler.md).

##### Set up local DNS caching {#node-local-dns}

[Set up NodeLocal DNS Cache](../../managed-kubernetes/tutorials/node-local-dns.md). To make sure that the settings are optimal, [install NodeLocal DNS Cache from {{ marketplace-full-name }}](../../managed-kubernetes/operations/applications/node-local-dns.md#marketplace-install).

#### When creating a node group via the CLI, a parameter conflict occurs. How do I fix that? {#conflicting-flags}

Check whether the `--location`, `--network-interface`, and `--public-ip` parameters are specified in the same command. If you provide these parameters together, the following errors occur:
* For the `--location` and `--public-ip` or `--location` and `--network-interface` pairs:

  ```text
  ERROR: rpc error: code = InvalidArgument desc = Validation error:
  allocation_policy.locations[0].subnet_id: can't use "allocation_policy.locations[0].subnet_id" together with "node_template.network_interface_specs"
  ```

* For the `--network-interface` and `--public-ip` pair:

  ```text
  ERROR: flag --public-ip cannot be used together with --network-interface. Use '--network-interface' option 'nat' to get public address
  ```

Make sure you only provide one of the three parameters in a command. It is enough to specify the location of a {{ managed-k8s-name }} node group either in `--location` or `--network-interface`.

{% include [assign-public-ip-addresses](../../_includes/managed-kubernetes/assign-public-ip-addresses.md) %}

#### Error connecting to a cluster using kubectl {#connect-to-cluster}

Error message:

```text
ERROR: cluster has empty endpoint
```

The error occurs if you try to [connect to a cluster](../../managed-kubernetes/operations/connect/index.md#kubectl-connect) with no public IP address and get `kubectl` credentials for a public IP address using this command:

```bash
{{ yc-k8s }} cluster \
   get-credentials <cluster_name_or_ID> \
   --external
```

To connect to the cluster's private IP address from a VM located in the same network, get `kubectl` credentials using this command:

```bash
{{ yc-k8s }} cluster \
   get-credentials <cluster_name_or_ID> \
   --internal
```

If you need to connect to a cluster from the internet, [recreate the cluster and assign](../../managed-kubernetes/operations/kubernetes-cluster/kubernetes-cluster-create.md) it a public IP address.

#### Errors occur when connecting to a node over SSH {#node-connect}

Error messages:

```text
Permission denied (publickey,password)
```

```text
Too many authentication failures
```

Errors occur [when connecting to a {{ managed-k8s-name }} node](../../managed-kubernetes/operations/node-connect-ssh.md) in the following cases:
* No public SSH key is added to the {{ managed-k8s-name }} node group metadata.

  **Solution**: [Update the {{ managed-k8s-name }}  node group keys](../../managed-kubernetes/operations/node-connect-ssh.md#node-add-metadata).
* An invalid public SSH key is added to the {{ managed-k8s-name }} node group metadata.

  **Solution**: [Change the format of the public key file to the appropriate one](../../managed-kubernetes/operations/node-connect-ssh.md#key-format) and [update the {{ managed-k8s-name }} node group keys](../../managed-kubernetes/operations/node-connect-ssh.md#node-add-metadata).
* No private SSH key is added to an authentication agent (ssh-agent).

  **Solution**: Add a private key by running the following command: `ssh-add <path_to_private_key_file>`.

#### How do I grant internet access to {{ managed-k8s-name }} cluster nodes? {#internet}

If {{ managed-k8s-name }} cluster nodes have no access to the internet, the following error occurs when trying to connect to the internet:

```text
Failed to pull image "{{ registry }}/***": rpc error: code = Unknown desc = Error response from daemon: Gethttps://{{ registry }}/v2/: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
```

There are several ways to grant internet access to {{ managed-k8s-name }} cluster nodes:
* Create and configure a [NAT gateway](../../vpc/operations/create-nat-gateway.md) or [NAT instance](../../vpc/tutorials/nat-instance/index.md). As a result, through [static routing](../../vpc/concepts/routing.md), traffic will be routed via the gateway or a separate [VM instance](../../compute/concepts/vm.md) with NAT features.
* [Assign a public IP address to a {{ managed-k8s-name }} node group](../../managed-kubernetes/operations/node-group/node-group-update.md#update-settings).

{% note info %}

If you assigned public IP addresses to the cluster nodes and then configured the NAT gateway or NAT instance, internet access via the public IP addresses will be disabled. For more information, see the [{{ vpc-full-name }} documentation](../../vpc/concepts/routing.md#internet-routes).

{% endnote %}

#### Why cannot I choose Docker as the container runtime environment? {#docker-runtime}

There is no support for Docker as a container runtime environment in clusters with {{ k8s }} version 1.24 or higher. Only [containerd](https://containerd.io/) is available.

#### Error when connecting a {{ GL }} repository to Argo CD {#argo-cd}

Error message:

```text
FATA[0000] rpc error: code = Unknown desc = error testing repository connectivity: authorization failed
```

This error occurs if access to {{ GL }} over HTTP(S) is disabled.

**Solution**: Enable access. To do this:

  1. In {{ GL }}, on the left-hand panel, select **Admin → Settings → General**.
  1. Under **Visibility and access controls**, find the **Enabled Git access protocols** setting.
  1. In the list, select the item which allows access over HTTP(S).

  [For more information, see the {{ GL }} documentation](https://docs.gitlab.com/administration/settings/visibility_and_access_controls/#configure-enabled-git-access-protocols).