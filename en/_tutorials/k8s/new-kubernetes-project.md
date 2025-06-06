To launch an app:

1. [Create service accounts](#create-sa).
1. [Create security groups](#create-sg).
1. [Prepare {{ k8s }} resources](#create-k8s-res).
1. [Connect to the {{ managed-k8s-name }} cluster](#cluster-connect).
1. [Prepare {{ container-registry-name }} resources](#create-cr-res).
1. [Install {{ alb-name }}](#setup-alb).
1. [Create a load balancer](#create-ingress).

If you no longer need the resources you created, [delete them](#clear-out).


## Required paid resources {#paid-resources}

The support cost includes:

* Fee for a DNS zone and DNS requests (see [{{ dns-name }} pricing](../../dns/pricing.md)).
* {{ managed-k8s-name }} cluster fee: using the master and outgoing traffic (see [{{ managed-k8s-name }} pricing](../../managed-kubernetes/pricing.md)).
* Cluster nodes (VM) fee: using computing resources, operating system, and storage (see [{{ compute-name }} pricing](../../compute/pricing.md)).
* Fee for {{ container-registry-name }}: using the storage and outgoing traffic (see [{{ container-registry-name }} pricing](../../container-registry/pricing)).
* Fee for using the computing resources of the L7 load balancer (see [{{ alb-name }} pricing](../../application-load-balancer/pricing.md)).
* Fee for a public IP address for an L7 load balancer (see [{{ vpc-name }} pricing](../../vpc/pricing.md#prices-public-ip)).


## Getting started {#before-you-begin}

1. {% include [cli-install](../../_includes/cli-install.md) %}

   {% include [default-catalogue](../../_includes/default-catalogue.md) %}

1. Install the `jq` [JSON stream processor](https://stedolan.github.io/jq/).

   ```bash
   sudo apt update && sudo apt install jq
   ```

### Create a network with a subnet {#create-network}

1. [Create a network](../../vpc/operations/network-create.md) named `yc-auto-network`:

   ```bash
   yc vpc network create --name yc-auto-network
   ```

1. [Create a subnet](../../vpc/operations/subnet-create.md) in the `{{ region-id }}-a` [availability zone](../../overview/concepts/geo-scope.md):

   ```bash
   yc vpc subnet create \
     --name yc-auto-subnet-0 \
     --network-name yc-auto-network \
     --range 192.168.1.0/24 \
     --zone {{ region-id }}-a
   ```

### Register a domain zone and add a certificate {#register-domain}

1. [Register a public domain zone and delegate your domain](../../dns/operations/zone-create-public.md).
1. If you already have a [certificate](../../certificate-manager/concepts/index.md#types) for the [domain zone](../../dns/concepts/dns-zone.md), [add its details](../../certificate-manager/operations/import/cert-create.md) to [{{ certificate-manager-full-name }}](../../certificate-manager/).

   If you have no certificate, issue a new Let's Encrypt® certificate and [add](../../certificate-manager/operations/managed/cert-create.md) it to {{ certificate-manager-name }}.
1. Get the certificate ID:

   ```bash
   yc certificate-manager certificate list
   ```

   Result:

   ```text
   +-----------------+-------+----------------+---------------------+----------+--------+
   |       ID        | NAME  |    DOMAINS     |      NOT AFTER      |   TYPE   | STATUS |
   +-----------------+-------+----------------+---------------------+----------+--------+
   | <ID> | <name> | <domain_name> | 2022-04-06 17:19:37 | IMPORTED | ISSUED |
   +-----------------+-------+----------------+---------------------+----------+--------+
   ```

## Create service accounts {#create-sa}

For a {{ managed-k8s-name }} cluster and [load balancer](../../application-load-balancer/concepts/application-load-balancer.md) to run, the following [service accounts](../../iam/concepts/users/service-accounts.md) are required:
* Service account with the `k8s.clusters.agent` and `vpc.publicAdmin` [roles](../../resource-manager/concepts/resources-hierarchy.md#folder) for the [folder](../../managed-kubernetes/security/index.md#yc-api) where the {{ managed-k8s-name }} cluster is created. This service account will be used to create the resources required for the {{ managed-k8s-name }} cluster.
* Service account with the [{{ roles-cr-puller }}](../../container-registry/security/index.md#container-registry-images-puller) role for the folder containing the [Docker image](../../container-registry/concepts/registry.md) [registry](../../container-registry/concepts/docker-image.md). [Nodes](../../managed-kubernetes/concepts/index.md#node-group) will pull the required Docker images from the registry on behalf of this service account.
* For the {{ alb-name }} Ingress controller to run, you need service accounts with the following roles:
  * [alb.editor](../../application-load-balancer/security/index.md#alb-editor): To create the required resources.
  * [vpc.publicAdmin](../../vpc/security/index.md#vpc-public-admin): To manage [external connectivity](../../vpc/security/index.md#roles-list).
  * [certificate-manager.certificates.downloader](../../certificate-manager/security/index.md#certificate-manager-certificates-downloader): To use certificates registered in [{{ certificate-manager-name }}](../../certificate-manager/).
  * [compute.viewer](../../compute/security/index.md#compute-viewer): To use {{ managed-k8s-name }} cluster nodes in the load balancer [target groups](../../application-load-balancer/concepts/target-group.md).

### Service account for resources {#res-sa}

To create a service account which will create the resources for the {{ managed-k8s-name }} cluster:
1. Write the folder ID from your {{ yandex-cloud }} CLI profile configuration to the variable:

   {% list tabs group=programming_language %}

   - Bash {#bash}

     ```bash
     FOLDER_ID=$(yc config get folder-id)
     ```

   - PowerShell {#powershell}

     ```shell script
     $FOLDER_ID = yc config get folder-id
     ```

   {% endlist %}

1. Create a service account:

   {% list tabs group=programming_language %}

   - Bash {#bash}

     ```bash
     yc iam service-account create --name k8s-res-sa-$FOLDER_ID
     ```

   - PowerShell {#powershell}

     ```shell script
     yc iam service-account create --name k8s-res-sa-$FOLDER_ID
     ```

   {% endlist %}

1. Write the service account ID to the variable:

   {% list tabs group=programming_language %}

   - Bash {#bash}

     ```bash
     RES_SA_ID=$(yc iam service-account get --name k8s-res-sa-$FOLDER_ID --format json | jq .id -r)
     ```

   - PowerShell {#powershell}

     ```shell script
     $RES_SA_ID = (yc iam service-account get --name k8s-res-sa-$FOLDER_ID --format json | ConvertFrom-Json).id
     ```

   {% endlist %}

1. Assign the service account the [{{ roles-editor }}](../../iam/roles-reference.md#editor) role for the folder:

   ```bash
   yc resource-manager folder add-access-binding \
     --id $FOLDER_ID \
     --role editor \
     --subject serviceAccount:$RES_SA_ID
   ```

### Service account for nodes {#node-sa}

To create a service account that lets nodes download the necessary Docker images from the registry:
1. Write the folder ID from your {{ yandex-cloud }} CLI profile configuration to the variable:

   {% list tabs group=programming_language %}

   - Bash {#bash}

     ```bash
     FOLDER_ID=$(yc config get folder-id)
     ```

   - PowerShell {#powershell}

     ```shell script
     $FOLDER_ID = yc config get folder-id
     ```

   {% endlist %}

1. Create a service account:

   {% list tabs group=programming_language %}

   - Bash {#bash}

     ```bash
     yc iam service-account create --name k8s-node-sa-$FOLDER_ID
     ```

   - PowerShell {#powershell}

     ```shell script
     yc iam service-account create --name k8s-node-sa-$FOLDER_ID
     ```

   {% endlist %}

1. Write the service account ID to the variable:

   {% list tabs group=programming_language %}

   - Bash {#bash}

     ```bash
     NODE_SA_ID=$(yc iam service-account get --name k8s-node-sa-$FOLDER_ID --format json | jq .id -r)
     ```

   - PowerShell {#powershell}

     ```shell script
     $NODE_SA_ID = (yc iam service-account get --name k8s-node-sa-$FOLDER_ID --format json | ConvertFrom-Json).id
     ```

   {% endlist %}

1. Assign the service account the [{{ roles-cr-puller }}](../../container-registry/security/index.md#container-registry-images-puller) role for the folder:

   ```bash
   yc resource-manager folder add-access-binding \
     --id $FOLDER_ID \
     --role container-registry.images.puller \
     --subject serviceAccount:$NODE_SA_ID
   ```

### Service account required for the {{ alb-name }} Ingress controller {#ic-sa}

1. Write the folder ID from your {{ yandex-cloud }} CLI profile configuration to the variable:

   {% list tabs group=programming_language %}

   - Bash {#bash}

     ```bash
     FOLDER_ID=$(yc config get folder-id)
     ```

   - PowerShell {#powershell}

     ```shell script
     $FOLDER_ID = yc config get folder-id
     ```

   {% endlist %}

1. Create a service account:

   {% list tabs group=programming_language %}

   - Bash {#bash}

     ```bash
     yc iam service-account create --name k8s-ic-sa-$FOLDER_ID
     ```

   - PowerShell {#powershell}

     ```shell script
     yc iam service-account create --name k8s-ic-sa-$FOLDER_ID
     ```

   {% endlist %}

1. Write the service account ID to the variable:

   {% list tabs group=programming_language %}

   - Bash {#bash}

     ```bash
     IC_SA_ID=$(yc iam service-account get --name k8s-ic-sa-$FOLDER_ID --format json | jq .id -r)
     ```

   - PowerShell {#powershell}

     ```shell script
     $RES_SA_ID = (yc iam service-account get --name k8s-ic-sa-$FOLDER_ID --format json | ConvertFrom-Json).id
     ```

   {% endlist %}

1. Assign the service account the following roles for the folder:
   * [{{ roles-alb-editor }}](../../application-load-balancer/security/index.md#alb-editor).
   * [{{ roles-vpc-public-admin }}](../../vpc/security/index.md#vpc-public-admin).
   * [certificate-manager.certificates.downloader](../../certificate-manager/security/index.md#certificate-manager-certificates-downloader).
   * [compute.viewer](../../compute/security/index.md#compute-viewer).

   ```bash
   yc resource-manager folder add-access-binding \
     --id $FOLDER_ID \
     --role alb.editor \
     --role vpc.publicAdmin \
     --role certificate-manager.certificates.downloader \
     --role compute.viewer \
     --subject serviceAccount:$IC_SA_ID
   ```

1. Create an [authorized key](../../iam/concepts/authorization/access-key.md) for the service account and save it to a file named `sa-key.json`:

   ```bash
   yc iam key create \
     --service-account-id $IC_SA_ID \
     --output sa-key.json
   ```

## Create security groups {#create-sg}

{% include [configure-sg-manual](../../_includes/managed-kubernetes/security-groups/configure-sg-manual-lvl3.md) %}

{% include [configure-sg-alb-manual](../../_includes/managed-kubernetes/security-groups/configure-sg-alb-manual.md) %}

{% include [sg-common-warning](../../_includes/managed-kubernetes/security-groups/sg-common-warning.md) %}

## Create {{ k8s }} resources {#create-k8s-res}

{% include notitle [create-k8s-res](../../_includes/managed-kubernetes/create-k8s-res.md) %}

## Connect to the {{ managed-k8s-name }} cluster {#cluster-connect}

{% include [Install and configure kubectl](../../_includes/managed-kubernetes/kubectl-install.md) %}

## Create {{ container-registry-name }} resources {#create-cr-res}

### Create a registry {#registry-create}

Create a container registry:

```bash
yc container registry create --name yc-auto-cr
```

### Configure Docker credential helper {#config-ch}

To facilitate authentication in {{ container-registry-name }}, configure a [Docker credential helper](../../container-registry/operations/authentication.md#cred-helper). It enables you to use private {{ yandex-cloud }} registries without running the `docker login` command.

To configure a credential helper, run the following command:

```bash
yc container registry configure-docker
```

### Prepare a Docker image {#docker-image}

Build a Docker image and push it to the registry.
1. Create a Dockerfile named `hello.dockerfile` and add the following lines to it:

   ```docker
   FROM nginx
   CMD echo "Hi, I'm inside"
   ```

1. Assemble the Docker image.
   1. Get the ID of the [previously created](#registry-create) registry and write it to the variable:

      {% list tabs group=programming_language %}

      - Bash {#bash}

        ```bash
        REGISTRY_ID=$(yc container registry get --name yc-auto-cr --format json | jq .id -r)
        ```

      - PowerShell {#powershell}

        ```shell script
        $REGISTRY_ID = (yc container registry get --name yc-auto-cr --format json | ConvertFrom-Json).id
        ```

      {% endlist %}

   1. Build the Docker image:

      ```bash
      docker build . -f hello.dockerfile -t {{ registry }}/$REGISTRY_ID/nginx:hello
      ```

   1. Push the Docker image to the registry:

      ```bash
      docker push {{ registry }}/$REGISTRY_ID/nginx:hello
      ```

1. Make sure the Docker image was pushed to the registry:

   ```bash
   yc container image list
   ```

   Result:

   ```text
   +----------------------+---------------------+----------------------------+-------+-----------------+
   |          ID          |       CREATED       |            NAME            | TAGS  | COMPRESSED SIZE |
   +----------------------+---------------------+----------------------------+-------+-----------------+
   | crpa2mf008mp******** | 2019-11-20 11:52:17 | crp71hkgiolp********/nginx | hello | 27.5 MB         |
   +----------------------+---------------------+----------------------------+-------+-----------------+
   ```

### Run the test app {#test-app}

Create a [pod](../../managed-kubernetes/concepts/index.md#pod) with the app from the Docker image and make sure that no additional authentication in {{ container-registry-name }} was required to push the Docker image.
1. Run the pod with the app from the Docker image:

   ```bash
   kubectl run --attach hello-nginx --image {{ registry }}/$REGISTRY_ID/nginx:hello
   ```

1. Make sure the pod has entered the `Running` state and learn its full name.

   ```bash
   kubectl get pods
   ```

   Result:

   ```text
   NAME                          READY  STATUS   RESTARTS  AGE
   hello-nginx-5847fb96**-*****  1/1    Running  0         1h
   ```

1. Check the logs of the container running on this pod:

   ```bash
   kubectl logs hello-nginx-5847fb96**-*****
   ```

   Result:

   ```text
   Hi, I'm inside
   ```

   The pod pushed the Docker image with no additional authentication on the {{ container-registry-name }} side.

## Install {{ alb-name }} {#setup-alb}

To install [{{ alb-name }}](/marketplace/products/yc/alb-ingress-controller), [follow this guide](../../managed-kubernetes/operations/applications/alb-ingress-controller.md).

## Create a load balancer {#create-ingress}

1. Create a load balancer for [{{ k8s }} services](../../managed-kubernetes/concepts/index.md#service).
   1. Create the `ingress.yaml` file with the Ingress controller manifest:

      ```yaml
      ---
      apiVersion: networking.k8s.io/v1
      kind: Ingress
      metadata:
        name: alb-demo-tls
        annotations:
          ingress.alb.yc.io/subnets: <list_of_subnet_IDs>
          ingress.alb.yc.io/security-groups: <list_of_security_group_IDs>
          ingress.alb.yc.io/external-ipv4-address: <IP_address_assignment_method>
          ingress.alb.yc.io/group-name: <Ingress_resource_group_name>
      spec:
        tls:
          - hosts:
              - <domain_name>
            secretName: yc-certmgr-cert-id-<TLS_certificate_ID>
        rules:
          - host: <domain_name>
            http:
              paths:
                - pathType: Prefix
                  path: "/"
                  backend:
                    service:
                      name: service-hello
                      port:
                        name: http
      ---
      apiVersion: v1
      kind: Service
      metadata:
        name: service-hello
      spec:
        selector:
          run: hello-nginx
        type: NodePort
        ports:
          - name: http
            port: 80
            targetPort: 80
            protocol: TCP
            nodePort: 30080
      ```

      Where:
      * `ingress.alb.yc.io/subnets`: Specify one or more [subnets](../../vpc/concepts/network.md#subnet) that {{ alb-name }} will work with.
      * `ingress.alb.yc.io/security-groups`: Specify one or more [security groups](../../application-load-balancer/concepts/application-load-balancer.md#security-groups) for {{ alb-name }}. If you skip this parameter, the default security group will be used. At least one of the security groups must allow an outgoing TCP connection to port 10501 in the node group subnet or its security group.
      * `ingress.alb.yc.io/external-ipv4-address`: To get a new IP address or provide public access to {{ alb-name }} from the internet, specify the [previously obtained IP address](../../vpc/operations/get-static-ip.md) or set the value to `auto`.

        With `auto` setting, deleting the Ingress controller will also remove its associated IP address from the cloud. To avoid this, use a reserved IP address.
      * `ingress.alb.yc.io/group-name`: Specify the group name. It groups {{ k8s }} Ingress resources served by a separate {{ alb-name }} instance.
   1. Create a load balancer:

      ```bash
      kubectl apply -f ingress.yaml
      ```

   1. Wait until the load balancer is created and gets a public IP address. This may take several minutes:

      ```bash
      kubectl get ingress alb-demo-tls
      ```

      The expected result is a non-empty value in the `ADDRESS` field for the new load balancer:

      ```bash
      NAME          CLASS   HOSTS           ADDRESS     PORTS    AGE
      alb-demo-tls  <none>  <domain_name>  <IP_address>  80,443  15h
      ```

      Based on the load balancer configuration, an [L7 load balancer](../../application-load-balancer/concepts/application-load-balancer.md) will be automatically deployed.
1. Follow the `https://<domain_name>` link and make sure that your application is successfully published.

    {% include [check-sg-if-url-unavailable-lvl3](../../_includes/managed-kubernetes/security-groups/check-sg-if-url-unavailable-lvl3.md) %}

## Delete the resources you created {#clear-out}

Some resources are not free of charge. To avoid paying for them, delete the resources you no longer need:

1. [Delete the {{ managed-k8s-name }} cluster](../../managed-kubernetes/operations/kubernetes-cluster/kubernetes-cluster-delete.md):

   ```bash
   yc managed-kubernetes cluster delete --name k8s-demo
   ```

1. Delete the service accounts.
   * Delete the service account created for resources:

     ```bash
     yc iam service-account delete --id $RES_SA_ID
     ```

   * Delete the service account created for nodes:

     ```bash
     yc iam service-account delete --id $NODE_SA_ID
     ```

   * Delete the service account created for the load balancer:

     ```bash
     yc iam service-account delete --id $IC_SA_ID
     ```

1. Delete resources {{ container-registry-name }}.
   1. Find out the ID of the Docker image pushed to the registry:

      {% list tabs group=programming_language %}

      - Bash {#bash}

        ```bash
        IMAGE_ID=$(yc container image list --format json | jq .[0].id -r)
        ```

      - PowerShell {#powershell}

        ```powershell
        $IMAGE_ID = (yc container image list --format json | ConvertFrom-Json).id
        ```

      {% endlist %}

   1. [Delete the Docker image](../../container-registry/operations/docker-image/docker-image-delete.md):

      ```bash
      yc container image delete --id $IMAGE_ID
      ```

   1. [Delete the registry](../../container-registry/operations/registry/registry-delete.md):

      ```bash
      yc container registry delete --name yc-auto-cr
      ```

1. Delete the [L7 load balancer](../../application-load-balancer/operations/application-load-balancer-delete.md).
