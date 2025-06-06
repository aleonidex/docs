# Enabling a blue-green and canary deployment of web service versions


Configure your web service architecture to switch between versions using the two common deployment models: a [blue-green deployment](https://martinfowler.com/bliki/BlueGreenDeployment.html) and a [canary deployment](https://martinfowler.com/bliki/CanaryRelease.html). 

Both models use two backends: a blue and a green one. First, you deploy a stable version generally available to users on one backend, e.g., the blue one. Then you use the other backend, the green one, to test the next version. When the testing is complete, the backends will switch roles: 

* In a blue-green deployment, all user traffic switches from one backend to the other right away.
* In a canary deployment, the traffic switches gradually, initially for some of the users.

After that, the green backend becomes the primary one, and you can use the blue backend to test your next version. As long as your previous version runs on the blue backend, you can roll the service back to it by switching back the roles.

In this tutorial, we will use {{ objstorage-full-name }} buckets as backends with a {{ alb-full-name }}-enabled L7 load balancer switching traffic between them. To reduce the content delivery time, the load balancer receives user requests via the {{ cdn-full-name }} content delivery network.

In our examples, we are going to use the `cdn.yandexcloud.example` and `cdn-staging.yandexcloud.example` domain names.

You can use various [supported tools](#supported-tools) to perform these steps.

To build an architecture for a blue-green and canary deployment:

1. [Get your cloud ready](#before-you-begin).
1. [Add a certificate to {{ certificate-manager-name }}](#add-certificate).
1. [Create a cloud network and subnets](#create-network).
1. [Create buckets in {{ objstorage-name }}](#create-buckets).
1. [Upload the files of your service to the buckets](#upload-files).
1. [Create {{ alb-name }} backend groups](#create-l7backend).
1. [Create an HTTP router and virtual hosts](#create-route-hosts).
1. [Create an L7 load balancer](#create-balancer).
1. [Create a CDN](#create-cdn-resource).
1. [Configure DNS for the service](#configure-dns).
1. [Test the service and version switching](#check).

If you no longer need the resources you created, [delete them](#clear-out).

## Supported tools {#supported-tools}

You can complete most of the steps in this tutorial using any standard tool, such as the [management console]({{ link-console-main }}), [{{ yandex-cloud }}](../../cli/) and [AWS](../../storage/tools/aws-cli.md) CLIs, {{ TF }}, and the [{{ yandex-cloud }}](../../api-design-guide) API. Each step lists its respective supported tools.

Some steps do not support certain tools:

* Currently, you cannot use CLIs and {{ TF }} to:
  * [Create an {{ alb-name }} backend group with buckets as backends](#create-l7backend).
  * Get the domain name of a CDN load balancer when [configuring DNS for the service](#configure-dns).
  * Disable and enable caching of a CDN when [running a health check and testing version switching](#check).
* Currently, you cannot get the domain name of a CDN load balancer through the API when [configuring DNS for the service](#configure-dns).

## Get your cloud ready {#before-you-begin}

{% include [before-you-begin](../_tutorials_includes/before-you-begin.md) %}

We will use a folder named `example-folder` in our example.


### Required paid resources {#paid-resources}

The infrastructure support costs include:

* Fee for data storage in {{ objstorage-name }}, data operations, and outbound traffic (see [{{ objstorage-name }} pricing](../../storage/pricing.md)).
* Fee for using the computing resources of the L7 load balancer (see [{{ alb-name }} pricing](../../application-load-balancer/pricing.md)).
* Fee for outgoing traffic from CDN servers (see [{{ cdn-name }} pricing](../../cdn/pricing.md)).
* Fee for public DNS queries and DNS zones if using {{ dns-full-name }} (see [{{ dns-name }} pricing](../../dns/pricing.md)).

## Add a certificate to {{ certificate-manager-name }} {#add-certificate}

{% include [certificate-usage](../../_includes/cdn/certificate-usage.md) %}

For a Let's Encrypt® certificate, have your [rights checked](../../certificate-manager/operations/managed/cert-validate.md) for the domain specified in the certificate.


## Create a cloud network and subnets {#create-network}

All resources you create will belong to the same [cloud network](../../vpc/concepts/network.md).

{% list tabs group=instructions %}

- Management console {#console}

    1. In the [management console]({{ link-console-main }}), select `example-folder`.
    1. From the list of services, select **{{ ui-key.yacloud.iam.folder.dashboard.label_vpc }}**.
    1. At the top right, click **{{ ui-key.yacloud.vpc.networks.button_create }}**.
    1. In the **{{ ui-key.yacloud.vpc.networks.create.field_name }}** field, specify `canary-network`.
    1. In the **{{ ui-key.yacloud.vpc.networks.create.field_advanced }}** field, select **{{ ui-key.yacloud.vpc.networks.create.field_is-default }}**.
    1. Click **{{ ui-key.yacloud.vpc.networks.button_create }}**.

- {{ yandex-cloud }} CLI {#cli}

  {% include [cli-install](../../_includes/cli-install.md) %}

  {% include [default-catalogue](../../_includes/default-catalogue.md) %}

  1. Create a network named `canary-network`:

     ```bash
     yc vpc network create canary-network
     ```

     Result:

     ```
     id: enptrcle5q3d********
     folder_id: b1g9hv2loamq********
     created_at: "2021-11-03T09:25:03Z"
     name: canary-network
     default_security_group_id: enpbsnnop4ak********
     ```

     For more information about the `yc vpc network create` command, see the [CLI reference](../../cli/cli-ref/vpc/cli-ref/network/create.md).

  1. Create subnets in all availability zones:

     * `{{ region-id }}-a`:

       ```bash
       yc vpc subnet create canary-subnet-{{ region-id }}-a \
         --zone {{ region-id }}-a \
         --network-name canary-network \
         --range 10.1.0.0/16
       ```

       Result:

       ``` 
       id: e9bnnssj8sc8********
       folder_id: b1g9hv2loamq********
       created_at: "2021-11-03T09:27:00Z"
       name: canary-subnet-{{ region-id }}-a
       network_id: enptrcle5q3d********
       zone_id: {{ region-id }}-a
       v4_cidr_blocks:
       - 10.1.0.0/16
       ```

     * `{{ region-id }}-b`:

       ```bash
       yc vpc subnet create canary-subnet-{{ region-id }}-b \
         --zone {{ region-id }}-b \
         --network-name canary-network \
         --range 10.2.0.0/16
       ```

       Result:

       ``` 
       id: e2lghukd9iqo********
       folder_id: b1g9hv2loamq********
       created_at: "2021-11-03T09:27:39Z"
       name: canary-subnet-{{ region-id }}-b
       network_id: enptrcle5q3d********
       zone_id: {{ region-id }}-b
       v4_cidr_blocks:
       - 10.2.0.0/16
       ```

     * `{{ region-id }}-d`:

       ```bash
       yc vpc subnet create canary-subnet-{{ region-id }}-d \
         --zone {{ region-id }}-d \
         --network-name canary-network \
         --range 10.3.0.0/16
       ```

       Result:

       ``` 
       id: b0c3pte4o2kn********
       folder_id: b1g9hv2loamq********
       created_at: "2021-11-03T09:28:08Z"
       name: canary-subnet-{{ region-id }}-d
       network_id: enptrcle5q3d********
       zone_id: {{ region-id }}-d
       v4_cidr_blocks:
       - 10.3.0.0/16
       ```

     For more information about the `yc vpc subnet create` command, see the [CLI reference](../../cli/cli-ref/vpc/cli-ref/subnet/create.md).

- {{ TF }} {#tf}

  {% include [terraform-install](../../_includes/terraform-install.md) %}

  1. In the configuration file, describe the parameters for `canary-network` and its `canary-subnet-{{ region-id }}-a`, `canary-subnet-{{ region-id }}-b`, and `canary-subnet-{{ region-id }}-d` subnets:

     ```hcl
     resource "yandex_vpc_network" "canary-network" {
       name = "canary-network"
     }

     resource "yandex_vpc_subnet" "canary-subnet-a" {
       name           = "canary-subnet-{{ region-id }}-a"
       zone           = "{{ region-id }}-a"
       network_id     = "${yandex_vpc_network.canary-network.id}"
       v4_cidr_blocks = ["10.1.0.0/16"]
     }

     resource "yandex_vpc_subnet" "canary-subnet-b" {
       name           = "canary-subnet-{{ region-id }}-b"
       zone           = "{{ region-id }}-b"
       network_id     = "${yandex_vpc_network.canary-network.id}"
       v4_cidr_blocks = ["10.2.0.0/16"]
     }

     resource "yandex_vpc_subnet" "canary-subnet-d" {
       name           = "canary-subnet-{{ region-id }}-d"
       zone           = "{{ region-id }}-d"
       network_id     = "${yandex_vpc_network.canary-network.id}"
       v4_cidr_blocks = ["10.3.0.0/16"]
     }
     ```

     For more information, see the descriptions of [yandex_vpc_network]({{ tf-provider-resources-link }}/vpc_network) and [yandex_vpc_subnet]({{ tf-provider-resources-link }}/vpc_subnet) in the {{ TF }} documentation.

  1. Make sure the configuration files are correct.

     1. In the command line, go to the directory where you created the configuration file.
     1. Run a check using this command:

        ```bash
        terraform plan
        ```

     If the configuration is correct, the terminal will display a list of new resources with their properties. If the configuration contains any errors, {{ TF }} will point them out. 

  1. Deploy the cloud resources.

     1. If the configuration does not contain any errors, run this command:

        ```bash
        terraform apply
        ```

     1. Confirm creating the resources.

- API {#api}

  1. Create `canary-network` using the [NetworkService/Create](../../vpc/api-ref/grpc/Network/create.md) gRPC API call or the [create](../../vpc/api-ref/Network/create.md) REST API method.
  1. Create `canary-subnet-{{ region-id }}-a`, `canary-subnet-{{ region-id }}-b`, and `canary-subnet-{{ region-id }}-d` in the three availability zones using the [SubnetService/Create](../../vpc/api-ref/grpc/Subnet/create.md) gRPC API call or the [create](../../vpc/api-ref/Subnet/create.md) REST API method.

{% endlist %}

## Create buckets in {{ objstorage-name }} {#create-buckets}

{% list tabs group=instructions %}

- Management console {#console}

  1. In the [management console]({{ link-console-main }}), select `example-folder`.
  1. From the list of services, select **{{ ui-key.yacloud.iam.folder.dashboard.label_storage }}**.
  1. Create a blue bucket for the backend stable version:

     1. At the top right, click **{{ ui-key.yacloud.storage.buckets.button_create }}**.
     1. In the **{{ ui-key.yacloud.storage.bucket.settings.field_name }}** field, enter a name for the bucket.
     1. In the **{{ ui-key.yacloud.storage.bucket.settings.field_access-read }}** and **{{ ui-key.yacloud.storage.bucket.settings.field_access-list }}** fields, select `{{ ui-key.yacloud.storage.bucket.settings.access_value_public }}`.
     1. Click **{{ ui-key.yacloud.storage.buckets.create.button_create }}**.

  1. Similarly, create a green bucket for the backend test version.

- AWS CLI {#cli}

  1. Create a blue bucket for the backend stable version:

     ```bash
     aws --endpoint-url https://{{ s3-storage-host }} \
       s3 mb s3://<blue_bucket_name>
     ```

     Result:

     ```
     make_bucket: s3://<blue_bucket_name>
     ```

  1. Enable public access to read objects and their list:

     ```bash
     aws --endpoint-url https://{{ s3-storage-host }} \
       s3api put-bucket-acl \
       --bucket <blue_bucket_name> \
       --acl public-read
     ```

  1. Similarly, create a green bucket for the backend test version and enable public access to it.

- {{ TF }} {#tf}

  {% include [terraform-role](../../_includes/storage/terraform-role.md) %}

  1. Describe the properties for creating a service account and access key in the configuration file:

     {% include [terraform-sa-key](../../_includes/storage/terraform-sa-key.md) %}

  1. Add the properties of the blue (backend stable version) and green (backend test version) buckets to the configuration file:

     ```hcl
     ...

     resource "yandex_storage_bucket" "canary-bucket-blue" {
       access_key = yandex_iam_service_account_static_access_key.sa-static-key.access_key
       secret_key = yandex_iam_service_account_static_access_key.sa-static-key.secret_key
       bucket     = "<blue_bucket_name>"
       acl        = "public-read"
     }

     resource "yandex_storage_bucket" "canary-bucket-green" {
       access_key = yandex_iam_service_account_static_access_key.sa-static-key.access_key
       secret_key = yandex_iam_service_account_static_access_key.sa-static-key.secret_key
       bucket     = "<green_bucket_name>"
       acl        = "public-read"
     }
     ```

     For more information about `yandex_storage_bucket`, see the [{{ TF }} documentation]({{ tf-provider-resources-link }}/storage_bucket).

  1. Make sure the configuration files are correct.

     1. In the command line, go to the directory where you created the configuration file.
     1. Run a check using this command:

        ```bash
        terraform plan
        ```

     If the configuration is correct, the terminal will display a list of new resources with their properties. If the configuration contains any errors, {{ TF }} will point them out. 

  1. Deploy the cloud resources.

     1. If the configuration does not contain any errors, run this command:

        ```bash
        terraform apply
        ```

     1. Confirm creating the resources.

- API {#api}

  Use the [create](../../storage/s3/api-ref/bucket/create.md) REST API method.

{% endlist %}

## Upload the files of your service to the buckets {#upload-files}

1. Create two files, both named `index.html`, for two service versions: one for version 1 and the other for version 2.

   {% cut "Example of the `index.html` file, version 1" %}

   ```html
   <!DOCTYPE html>
   <html>
     <head>
       <title>Version 1</title>
     </head>
     <body>
       <p>Version 1 is working</p>
     </body>
   </html>
   ```

   {% endcut %}

   {% cut "Example of the `index.html` file, version 2" %}

   ```html
   <!DOCTYPE html>
   <html>
     <head>
       <title>Version 2</title>
     </head>
     <body>
       <p>Version 2 is working</p>
     </body>
   </html>
   ```

   {% endcut %}

1. Upload the files to the buckets:

   {% list tabs group=instructions %}

   - Management console {#console}

     1. In the [management console]({{ link-console-main }}), select `example-folder`.
     1. From the list of services, select **{{ ui-key.yacloud.iam.folder.dashboard.label_storage }}**.
     1. Select the blue bucket.
     1. Click **{{ ui-key.yacloud.storage.bucket.button_upload }}** and select `index.html` version 1 for uploading.
     1. Similarly, upload `index.html` version 2 to the green bucket.

   - AWS CLI {#cli}

     1. Upload `index.html` version 1 to the blue bucket:

        ```bash
        aws --endpoint-url https://{{ s3-storage-host }} \
          s3 cp v1/index.html s3://<blue_bucket_name>/index.html
        ```

        Result:

        ```
        upload: v1/index.html to s3://<blue_bucket_name>/index.html
        ```

     1. Upload `index.html` version 2 to the green bucket:

        ```bash
        aws --endpoint-url https://{{ s3-storage-host }} \
          s3 cp v2/index.html s3://<green_bucket_name>/index.html
        ```

        Result:

        ```
        upload: v2/index.html to s3://<green_bucket_name>/index.html
        ```

   - {{ TF }} {#tf}

     1. Add to the configuration file the properties of the `v1/index.html` and `v2/index.html` files uploaded to the blue and green buckets, respectively:

        ```hcl
        ...

        resource "yandex_storage_object" "canary-bucket-blue-index" {
          bucket = "<blue_bucket_name>"
          key    = "index.html"
          source = "v1/index.html"
        }

        resource "yandex_storage_bucket" "canary-bucket-green-index" {
          bucket = "<green_bucket_name>"
          key    = "index.html"
          source = "v2/index.html"
        }
        ```

        For more information about `yandex_storage_object`, see the [{{ TF }} documentation]({{ tf-provider-resources-link }}/storage_object).

     1. Make sure the configuration files are correct.

        1. In the command line, go to the directory where you created the configuration file.
        1. Run a check using this command:

           ```bash
           terraform plan
           ```

        If the configuration is correct, the terminal will display a list of new resources with their properties. If the configuration contains any errors, {{ TF }} will point them out. 

     1. Deploy the cloud resources.

        1. If the configuration does not contain any errors, run this command:

           ```bash
           terraform apply
           ```

        1. Confirm creating the resources.

   - API {#api}

     Use the [upload](../../storage/s3/api-ref/object/upload.md) REST API method.

   {% endlist %}

## Create a security group {#create-security-group}

[Security groups](../../vpc/concepts/security-groups.md) contain rules that allow the L7 load balancer to receive incoming traffic and send it to backend buckets.

To create security groups:

{% list tabs group=instructions %}

- Management console {#console}

  1. In the [management console]({{ link-console-main }}), select **{{ ui-key.yacloud.iam.folder.dashboard.label_vpc }}**.
  1. In the left-hand panel, select ![image](../../_assets/console-icons/shield.svg) **{{ ui-key.yacloud.vpc.label_security-groups }}**.
  1. At the top right, click **{{ ui-key.yacloud.vpc.network.security-groups.button_create }}**.
  1. In the **{{ ui-key.yacloud.vpc.network.security-groups.forms.field_sg-name }}** field, specify `canary-sg`.
  1. In the **{{ ui-key.yacloud.vpc.network.security-groups.forms.field_sg-network }}** field, select `canary-network`.
  1. Under **{{ ui-key.yacloud.vpc.network.security-groups.forms.label_section-rules }}**, create the following rules using the instructions below the table:

      | Traffic<br/>direction | {{ ui-key.yacloud.vpc.network.security-groups.forms.field_sg-rule-description }} | {{ ui-key.yacloud.vpc.network.security-groups.forms.field_sg-rule-port-range }} | {{ ui-key.yacloud.vpc.network.security-groups.forms.field_sg-rule-protocol }} | Source /<br/>destination | {{ ui-key.yacloud.vpc.network.security-groups.forms.field_sg-rule-cidr-blocks }} |
      | --- | --- | --- | --- | --- | --- |
      | `Outgoing` | `any` | `All` | `{{ ui-key.yacloud.vpc.network.security-groups.forms.value_any }}` | `{{ ui-key.yacloud.vpc.network.security-groups.forms.value_sg-rule-destination-cidr }}` | `0.0.0.0/0` |
      | `Incoming` | `ext-http` | `80` | `{{ ui-key.yacloud.common.label_tcp }}` | `{{ ui-key.yacloud.vpc.network.security-groups.forms.value_sg-rule-destination-cidr }}` | `0.0.0.0/0` |
      | `Incoming` | `ext-https` | `443` | `{{ ui-key.yacloud.common.label_tcp }}` | `{{ ui-key.yacloud.vpc.network.security-groups.forms.value_sg-rule-destination-cidr }}` | `0.0.0.0/0` |
      | `Incoming` | `healthchecks` | `30080` | `{{ ui-key.yacloud.common.label_tcp }}` | `{{ ui-key.yacloud.vpc.network.security-groups.forms.value_sg-rule-sg-type-balancer }}` | — |

     1. Navigate to the **{{ ui-key.yacloud.vpc.network.security-groups.label_egress }}** or **{{ ui-key.yacloud.vpc.network.security-groups.label_ingress }}** tab.
     1. Click **{{ ui-key.yacloud.vpc.network.security-groups.button_add-rule }}**.
     1. In the **{{ ui-key.yacloud.vpc.network.security-groups.forms.field_sg-rule-port-range }}** field of the window that opens, specify a single port or a port range for traffic to come to or from.
     1. In the **{{ ui-key.yacloud.vpc.network.security-groups.forms.field_sg-rule-protocol }}** field, specify the required protocol or leave `{{ ui-key.yacloud.vpc.network.security-groups.forms.value_any }}`.
     1. In the **{{ ui-key.yacloud.vpc.network.security-groups.forms.field_sg-rule-destination }}** or **{{ ui-key.yacloud.vpc.network.security-groups.forms.field_sg-rule-source }}** field, select the rule purpose:

        * `{{ ui-key.yacloud.vpc.network.security-groups.forms.value_sg-rule-destination-cidr }}`: Rule will apply to the range of IP addresses. In the **{{ ui-key.yacloud.vpc.network.security-groups.forms.field_sg-rule-cidr-blocks }}** field, specify the CIDR and subnet masks that traffic will come to or from. To add multiple CIDRs, click **{{ ui-key.yacloud.vpc.subnetworks.create.button_add-cidr }}**.
        * `{{ ui-key.yacloud.vpc.network.security-groups.forms.value_sg-rule-sg-type-balancer }}`: Rule allowing a load balancer to health-check VMs.

     1. Click **{{ ui-key.yacloud.common.save }}**. Repeat these steps to create all rules from the table.

  1. Click **{{ ui-key.yacloud.common.save }}**.

- {{ yandex-cloud }} CLI {#cli}

  Run the following command:

  ```bash
  yc vpc security-group create canary-sg \
    --network-name canary-network \
    --rule direction=egress,port=any,protocol=any,v4-cidrs=[0.0.0.0/0] \
    --rule direction=ingress,port=80,protocol=tcp,v4-cidrs=[0.0.0.0/0] \
    --rule direction=ingress,port=443,protocol=tcp,v4-cidrs=[0.0.0.0/0] \
    --rule direction=ingress,port=30080,protocol=tcp,predefined=loadbalancer_healthchecks
  ```

  Result:

  ```
  id: enpd133ngcnr********
  folder_id: b1g9hv2loamq********
  created_at: "2021-11-03T10:26:16Z"
  name: canary-sg
  network_id: enptrcle5q3d********
  status: ACTIVE
  rules:
  - id: enpkgrpi2gsi********
    direction: EGRESS
    protocol_name: ANY
    protocol_number: "-1"
    cidr_blocks:
      v4_cidr_blocks:
      - 0.0.0.0/0
  - id: enpgssij0i16********
    direction: INGRESS
    ports:
      from_port: "80"
      to_port: "80"
    protocol_name: TCP
    protocol_number: "6"
    cidr_blocks:
      v4_cidr_blocks:
      - 0.0.0.0/0
  - id: enp0bft67j9l********
    direction: INGRESS
    ports:
      from_port: "443"
      to_port: "443"
    protocol_name: TCP
    protocol_number: "6"
    cidr_blocks:
      v4_cidr_blocks:
      - 0.0.0.0/0
  - id: enpmorcimu65********
    direction: INGRESS
    ports:
      from_port: "30080"
      to_port: "30080"
    protocol_name: TCP
    protocol_number: "6"
    predefined_target: loadbalancer_healthchecks
  ```

  For more information about the `yc vpc security-group create` command, see the [CLI reference](../../cli/cli-ref/vpc/cli-ref/security-group/create.md).

- {{ TF }} {#tf}

  1. Add the `canary-sg` security group properties to the configuration file:

     ```hcl
     resource "yandex_vpc_security_group" "canary-sg" {
       name       = "canary-sg"
       network_id = yandex_vpc_network.canary-network.id

       egress {
         protocol       = "ANY"
         port           = "ANY"
         v4_cidr_blocks = ["0.0.0.0/0"]
       }

       ingress {
         protocol       = "TCP"
         port           = 80
         v4_cidr_blocks = ["0.0.0.0/0"]
       }

       ingress {
         protocol       = "TCP"
         port           = 443
         v4_cidr_blocks = ["0.0.0.0/0"]
       }

       ingress {
         protocol          = "TCP"
         port              = 30080
         predefined_target = "loadbalancer_healthchecks"
       }
     }
     ```

     For more information about resource properties in {{ TF }}, see the [relevant {{ TF }} documentation]({{ tf-provider-resources-link }}/vpc_security_group).

  1. Make sure the configuration files are correct.

     1. In the command line, go to the directory where you created the configuration file.
     1. Run a check using this command:

        ```bash
        terraform plan
        ```

     If the configuration is correct, the terminal will display a list of new resources with their properties. If the configuration contains any errors, {{ TF }} will point them out. 

  1. Deploy the cloud resources.

     1. If the configuration does not contain any errors, run this command:

        ```bash
        terraform apply
        ```

     1. Confirm creating the resources.

- API {#api}

  Use the [SecurityGroupService/Create](../../vpc/api-ref/grpc/SecurityGroup/create.md) gRPC API call or the [create](../../vpc/api-ref/SecurityGroup/create.md) REST API method.

  To add a rule for load balancer health checks, use the `loadbalancer_healthchecks` parameter in the [SecurityGroupRuleSpec.target.predefined_target](../../vpc/api-ref/grpc/SecurityGroup/create.md#yandex.cloud.vpc.v1.SecurityGroupRuleSpec) field for the gRPC API or the [predefinedTarget](../../vpc/api-ref/SecurityGroup/create.md#yandex.cloud.vpc.v1.CreateSecurityGroupRequest) field for the REST API.

{% endlist %}

## Create {{ alb-name }} backend groups {#create-l7backend}

{% list tabs group=instructions %}

- Management console {#console}

  1. Create a backend group named `canary-bg-production` with `canary-backend-blue` and `canary-backend-green`:

     1. In the [management console]({{ link-console-main }}), select `example-folder`.
     1. From the list of services, select **{{ ui-key.yacloud.iam.folder.dashboard.label_application-load-balancer }}**.
     1. In the left-hand panel, select ![image](../../_assets/console-icons/cubes-3-overlap.svg) **{{ ui-key.yacloud.alb.label_backend-groups }}**.
     1. At the top right, click **{{ ui-key.yacloud.alb.button_backend-group-create }}**.
     1. In the **{{ ui-key.yacloud.common.name }}** field, specify `canary-bg-production`.
     1. Create a backend named `canary-backend-blue`:
         1. Under **{{ ui-key.yacloud.alb.label_backends }}**, click **{{ ui-key.yacloud.common.add }}**.
         1. In the **{{ ui-key.yacloud.common.name }}** field, specify `canary-backend-blue`.
         1. In the **{{ ui-key.yacloud.alb.label_backend-weight }}** field, specify `100`.
         1. In the **{{ ui-key.yacloud.common.type }}** field, select `{{ ui-key.yacloud.alb.label_bucket }}`.
         1. Select the blue bucket in the **{{ ui-key.yacloud.alb.label_bucket }}** field.
     1. Create a backend named `canary-backend-green`:
         1. Under **{{ ui-key.yacloud.alb.label_backends }}**, click **{{ ui-key.yacloud.common.add }}**.
         1. In the **{{ ui-key.yacloud.common.name }}** field, specify `canary-backend-green`.
         1. In the **{{ ui-key.yacloud.alb.label_backend-weight }}** field, specify `0`.
         1. In the **{{ ui-key.yacloud.common.type }}** field, select `{{ ui-key.yacloud.alb.label_bucket }}`.
         1. In the **{{ ui-key.yacloud.alb.label_bucket }}** field, select the green bucket.
     1. Click **{{ ui-key.yacloud.common.create }}**.

  1. Similarly, create a backend group named `canary-bg-staging`. Set the `canary-backend-blue` weight to `0` and the `canary-backend-green` weight to `100`.
  
  If you are going to complete the next steps in {{ TF }}, copy the IDs of the `canary-bg-production` and `canary-bg-staging` backend groups from the ![image](../../_assets/console-icons/cubes-3-overlap.svg) **{{ ui-key.yacloud.alb.label_backend-groups }}** tab.

- API {#api}

  Use the [BackendGroupService/Create](../../application-load-balancer/api-ref/grpc/BackendGroup/create.md) gRPC API call or the [create](../../application-load-balancer/api-ref/BackendGroup/create.md) REST API method.

{% endlist %} 

## Create an HTTP router and virtual hosts {#create-route-hosts}

{% list tabs group=instructions %}

- Management console {#console}

  1. In the [management console]({{ link-console-main }}), select `example-folder`.
  1. From the list of services, select **{{ ui-key.yacloud.iam.folder.dashboard.label_application-load-balancer }}**.
  1. In the left-hand panel, select ![image](../../_assets/console-icons/route.svg) **{{ ui-key.yacloud.alb.label_http-routers }}**.
  1. At the top right, click **{{ ui-key.yacloud.alb.button_http-router-create }}**.
  1. In the **{{ ui-key.yacloud.common.name }}** field, specify `canary-router`.
  1. Create a virtual host named `canary-vh-production`:

     1. Under **{{ ui-key.yacloud.alb.label_virtual-hosts }}**, click **{{ ui-key.yacloud.alb.button_virtual-host-add }}**.
     1. In the **{{ ui-key.yacloud.common.name }}** field, specify `canary-vh-production`.
     1. In the **{{ ui-key.yacloud.alb.label_authority }}** field, specify `cdn.yandexcloud.example`.
     1. Click **{{ ui-key.yacloud.alb.button_add-route }}**.
     1. In the **{{ ui-key.yacloud.common.name }}** field, specify `canary-route-production`.
     1. In the **{{ ui-key.yacloud.alb.label_path }}** field, select `{{ ui-key.yacloud.alb.label_match-prefix }}` and specify the `/` path.
     1. From the **{{ ui-key.yacloud.alb.label_http-methods }}** list, select `GET`.
     1. In the **{{ ui-key.yacloud.alb.label_route-action }}** field, keep `{{ ui-key.yacloud.alb.label_route-action-route }}`.
     1. From the **{{ ui-key.yacloud.alb.label_backend-group }}** list, select `canary-bg-production`.

  1. Create a virtual host named `canary-vh-staging`:

     * **{{ ui-key.yacloud.common.name }}**: `canary-vh-production`
     * **{{ ui-key.yacloud.alb.label_authority }}**: `cdn-staging.yandexcloud.example`
     * Route **{{ ui-key.yacloud.common.name }}**: `canary-route-staging`
     * **{{ ui-key.yacloud.alb.label_backend-group }}**: `canary-bg-staging`
     * The other parameters are the same as for `canary-vh-production`.

  1. Click **{{ ui-key.yacloud.common.create }}**.

- {{ yandex-cloud }} CLI {#cli}

  1. Create an HTTP router named `canary-router`:

     ```bash
     yc alb http-router create canary-router
     ```

     Result:

     ```
     id: ds7qd0vj01dj********
     name: canary-router
     folder_id: b1g9hv2loamq********
     created_at: "2021-11-03T10:31:41.027649223Z"
     ```

     For more information about the `yc alb http-router create` command, see the [CLI reference](../../cli/cli-ref/application-load-balancer/cli-ref/http-router/create.md).

  1. Create a virtual host named `canary-vh-production`:

     ```bash
     yc alb virtual-host create canary-vh-production \
       --http-router-name canary-router \
       --authority cdn.yandexcloud.example
     ```

     Result:

     ```
     done (1s)
     name: canary-vh-production
     authority:
     - cdn.yandexcloud.example
     ```

     For more information about the `yc alb virtual-host create` command, see the [CLI reference](../../cli/cli-ref/application-load-balancer/cli-ref/virtual-host/create.md).

  1. Create a route named `canary-route-production` in the `canary-vh-production` virtual host:

     ```bash
     yc alb virtual-host append-http-route canary-route-production \
       --http-router-name canary-router \
       --virtual-host-name canary-vh-production \
       --prefix-path-match "/" \
       --backend-group-name canary-bg-production
     ```

     Result:

     ```
     done (1s)
     name: canary-vh-production
     authority:
     - cdn.yandexcloud.example
     routes:
     - name: canary-route-production
       http:
         match:
           path:
             prefix_match: /
         route:
           backend_group_id: ds7pbm5fj2v0********
     ```

     For more information about the `yc alb virtual-host append-http-route` command, see the [CLI reference](../../cli/cli-ref/application-load-balancer/cli-ref/virtual-host/append-http-route.md).

  1. Create a virtual host named `canary-vh-staging`:

     ```bash
     yc alb virtual-host create canary-vh-staging \
       --http-router-name canary-router \
       --authority cdn-staging.yandexcloud.example
     ```

     Result:

     ```
     done (1s)
     name: canary-vh-staging
     authority:
     - cdn-staging.yandexcloud.example
     ```

  1. Create a route named `canary-route-staging` in the `canary-vh-staging` virtual host:

     ```bash
     yc alb virtual-host append-http-route canary-route-staging \
       --http-router-name canary-router \
       --virtual-host-name canary-vh-staging \
       --prefix-path-match "/" \
       --backend-group-name canary-bg-staging
     ```

     Result:

     ```
     done (1s)
     name: canary-vh-staging
     authority:
     - cdn-staging.yandexcloud.example
     routes:
     - name: canary-route-staging
       http:
         match:
           path:
             prefix_match: /
         route:
           backend_group_id: ds765atleota********
     ```

- {{ TF }} {#tf}

  1. Add to the configuration file the settings of the `canary-router` HTTP router, its virtual hosts, and routes:

     ```hcl
     ...

     resource "yandex_alb_http_router" "canary-router" {
       name = "canary-router"
     }

     resource "yandex_alb_virtual_host" "canary-vh-production" {
       name           = "canary-vh-production"
       http_router_id = ${yandex_alb_http_router.canary-router.id}
       authority      = "cdn.yandexcloud.example"

       route {
         name = "canary-route-production"
         http_route {
           http_route_action {
             backend_group_id = "<canary-bg-production_backend_group_ID>"
           }
         }
       }  
     }

     resource "yandex_alb_virtual_host" "canary-vh-staging" {
       name           = "canary-vh-staging"
       http_router_id = ${yandex_alb_http_router.canary-router.id}
       authority      = "cdn-staging.yandexcloud.example"

       route {
         name = "canary-route-staging"
         http_route {
           http_route_action {
             backend_group_id = "<canary-bg-staging_backend_group_ID>"
           }
         }
       }  
     }
     ```

     For more information, see the description of [yandex_alb_http_router]({{ tf-provider-resources-link }}/alb_http_router) and [yandex_alb_virtual_host]({{ tf-provider-resources-link }}/alb_virtual_host) in the {{ TF }} documentation.

  1. Make sure the configuration files are correct.

     1. In the command line, go to the directory where you created the configuration file.
     1. Run a check using this command:

        ```bash
        terraform plan
        ```

     If the configuration is correct, the terminal will display a list of new resources with their properties. If the configuration contains any errors, {{ TF }} will point them out. 

  1. Deploy the cloud resources.

     1. If the configuration does not contain any errors, run this command:

        ```bash
        terraform apply
        ```

     1. Confirm creating the resources.

- API {#api}

  1. Create an HTTP router named `canary-router` using the [HttpRouterService/Create](../../application-load-balancer/api-ref/grpc/HttpRouter/create.md) gRPC API call or the [create](../../application-load-balancer/api-ref/HttpRouter/create.md) REST API method.
  1. Create the `canary-vh-production` and `canary-vh-staging` virtual hosts bound to the router and their routes using the [VirtualHostService/Create](../../application-load-balancer/api-ref/grpc/VirtualHost/create.md) gRPC API call or the [create](../../application-load-balancer/api-ref/VirtualHost/create.md) REST API method.

{% endlist %}

## Create an L7 load balancer {#create-balancer}

{% list tabs group=instructions %}

- Management console {#console}

  1. In the [management console]({{ link-console-main }}), select `example-folder`.
  1. From the list of services, select **{{ ui-key.yacloud.iam.folder.dashboard.label_application-load-balancer }}**.
  1. At the top right, click **{{ ui-key.yacloud.alb.button_load-balancer-create }}**.
  1. In the **{{ ui-key.yacloud.common.name }}** field, specify `canary-balancer`.
  1. Under **{{ ui-key.yacloud.mdb.forms.section_network-settings }}**:

     1. In the **{{ ui-key.yacloud.mdb.forms.label_network }}** field, select `canary-network`.
     1. In the **{{ ui-key.yacloud.mdb.forms.field_security-group }}** field, select `canary-sg`. If you leave this field blank, any incoming and outgoing traffic will be allowed for the load balancer.

  1. Under **{{ ui-key.yacloud.alb.section_allocation-settings }}**, select the three subnets for the load balancer nodes (`canary-subnet-{{ region-id }}-a`, `canary-subnet-{{ region-id }}-b`, and `canary-subnet-{{ region-id }}-d`) and enable traffic to these subnets.
  1. Under **{{ ui-key.yacloud.alb.label_listeners }}**, click **{{ ui-key.yacloud.alb.button_add-listener }}** and set up the listener:

     1. In the **{{ ui-key.yacloud.common.name }}** field, specify `canary-listener`.
     1. Under **{{ ui-key.yacloud.alb.section_external-address-specs }}**:
        * In the **{{ ui-key.yacloud.alb.label_port }}** field, specify `80`.
        * In the **{{ ui-key.yacloud.common.type }}** field, select `{{ ui-key.yacloud.alb.label_address-auto }}`.

     1. In the **{{ ui-key.yacloud.alb.label_http-router }}** field, select `canary-router`.
  1. Click **{{ ui-key.yacloud.common.create }}**.

- {{ yandex-cloud }} CLI {#cli}

  1. Get the subnet IDs for `canary-network`:

     ```bash
     yc vpc network list-subnets canary-network
     ```

     Result:

     ```
     +----------------------+-----------------------------+----------------------+----------------------+----------------+---------------+---------------+
     |          ID          |            NAME             |      FOLDER ID       |      NETWORK ID      | ROUTE TABLE ID |     ZONE      |     RANGE     |
     +----------------------+-----------------------------+----------------------+----------------------+----------------+---------------+---------------+
     | e9bnnssj8sc8******** | canary-subnet-{{ region-id }}-d | b1g9hv2loamq******** | enptrcle5q3d******** |                | {{ region-id }}-d | [10.1.0.0/16] |
     | e2lghukd9iqo******** | canary-subnet-{{ region-id }}-b | b1g9hv2loamq******** | enptrcle5q3d******** |                | {{ region-id }}-b | [10.2.0.0/16] |
     | b0c3pte4o2kn******** | canary-subnet-{{ region-id }}-a | b1g9hv2loamq******** | enptrcle5q3d******** |                | {{ region-id }}-a | [10.3.0.0/16] |
     +----------------------+-----------------------------+----------------------+----------------------+----------------+---------------+---------------+
     ```

     For more information about the `yc vpc network list-subnets` command, see the [CLI reference](../../cli/cli-ref/vpc/cli-ref/network/list-subnets.md).

  1. Get the `canary-sg` security group ID:

     ```bash
     yc vpc security-group get canary-sg | grep "^id"
     ```

     Result:

     ```
     id: enpd133ngcnr********
     ```

     For more information about the `yc vpc security-group get` command, see the [CLI reference](../../cli/cli-ref/vpc/cli-ref/security-group/get.md).

  1. Create a load balancer named `canary-balancer`:

     ```bash
     yc alb load-balancer create canary-balancer \
       --network-name canary-network \
       --security-group-id <canary-sg_security_group_ID> \
       --location zone={{ region-id }}-a,subnet-id=<canary-subnet-{{ region-id }}-a_subnet_ID> \
       --location zone={{ region-id }}-b,subnet-id=<canary-subnet-{{ region-id }}-b_subnet_ID> \
       --location zone={{ region-id }}-d,subnet-id=<canary-subnet-{{ region-id }}-d_subnet_ID>
     ```

     Result:

     ```
     done (3m0s)
     id: ds77q7v39b4u********
     name: canary-balancer
     folder_id: b1g9hv2loamq********
     status: ACTIVE
     region_id: {{ region-id }}
     network_id: enptrcle5q3d********
     allocation_policy:
       locations:
       - zone_id: {{ region-id }}-d
         subnet_id: b0c3pte4o2kn********
       - zone_id: {{ region-id }}-b
         subnet_id: e2lghukd9iqo********
       - zone_id: {{ region-id }}-a
         subnet_id: e9bnnssj8sc8********
     log_group_id: ckg23vr4dlks********
     security_group_ids:
     - enpd133ngcnr********
     created_at: "2021-11-03T10:55:49.134935148Z"
     ```

     For more information about the `yc alb load-balancer create` command, see the [CLI reference](../../cli/cli-ref/application-load-balancer/cli-ref/load-balancer/create.md).

  1. Add a listener to the load balancer:

     ```bash
     yc alb load-balancer add-listener \
       --name canary-balancer \
       --listener-name canary-listener \
       --external-ipv4-endpoint port=80 \
       --http-router-name canary-router
     ```

     Result:

     ```
     done (43s)
     id: ds77q7v39b4u********
     name: canary-balancer
     folder_id: b1g9hv2loamq********
     status: ACTIVE
     region_id: {{ region-id }}
     network_id: enptrcle5q3d********
     listeners:
     - name: canary-listener
       endpoints:
       - addresses:
         - external_ipv4_address:
             address: 84.252.133.149
         ports:
         - "80"
       http:
         handler:
           http_router_id: ds7qd0vj01dj********
     allocation_policy:
       locations:
       - zone_id: {{ region-id }}-d
         subnet_id: b0c3pte4o2kn********
       - zone_id: {{ region-id }}-b
         subnet_id: e2lghukd9iqo********
       - zone_id: {{ region-id }}-a
         subnet_id: e9bnnssj8sc8********
     log_group_id: ckg23vr4dlks********
     security_group_ids:
     - enpd133ngcnr********
     created_at: "2021-11-03T10:55:49.134935148Z"
     ```

     For more information about the `yc alb load-balancer add-listener` command, see the [CLI reference](../../cli/cli-ref/application-load-balancer/cli-ref/load-balancer/add-listener.md).

- {{ TF }} {#tf}

  1. Add the settings of the `canary-balancer` L7 load balancer to the configuration file:

     ```hcl
     ...

     resource "yandex_alb_load_balancer" "canary-balancer" {
       name               = "canary-balancer"
       network_id         = ${yandex_vpc_network.canary-network.id}
       security_group_ids = [ ${yandex_vpc_security_group.canary-sg.id} ]

       allocation_policy {
         location {
           zone_id   = "{{ region-id }}-a"
           subnet_id = ${yandex_vpc_subnet.canary-subnet-{{ region-id }}-a.id}
         }

         location {
           zone_id   = "{{ region-id }}-b"
           subnet_id = ${yandex_vpc_subnet.canary-subnet-{{ region-id }}-b.id}
         }

         location {
           zone_id   = "{{ region-id }}-d"
           subnet_id = ${yandex_vpc_subnet.canary-subnet-{{ region-id }}-d.id}
         }
       }

       listener {
         name = "canary-listener"
         endpoint {
           address {
             external_ipv4_address {
             }
           }
           ports = [80]
         }
         http {
           handler {
             http_router_id = ${yandex_alb_http_router.canary-router.id}
           }
         }
       }
     }
     ```

     For more information about `yandex_alb_load_balancer`, see the [{{ TF }} documentation]({{ tf-provider-resources-link }}/alb_load_balancer).

  1. Make sure the configuration files are correct.

     1. In the command line, go to the directory where you created the configuration file.
     1. Run a check using this command:

        ```bash
        terraform plan
        ```

     If the configuration is correct, the terminal will display a list of new resources with their properties. If the configuration contains any errors, {{ TF }} will point them out. 

  1. Deploy the cloud resources.

     1. If the configuration does not contain any errors, run this command:

        ```bash
        terraform apply
        ```

     1. Confirm creating the resources.

- API {#api}

  Use the [LoadBalancerService/Create](../../application-load-balancer/api-ref/grpc/LoadBalancer/create.md) gRPC API call or the [create](../../application-load-balancer/api-ref/LoadBalancer/create.md) REST API method.

{% endlist %}

## Create a CDN {#create-cdn-resource}

{% list tabs group=instructions %}

- Management console {#console}

  1. In the [management console]({{ link-console-main }}), select `example-folder`.
  1. From the list of services, select **{{ ui-key.yacloud.iam.folder.dashboard.label_cdn }}**.
  1. {% include [activate-provider](../../_includes/cdn/activate-provider.md) %}
  1. Create a CDN:

     1. At the top right, click **{{ ui-key.yacloud.cdn.button_resource-create }}**.
     1. Configure the basic CDN settings:

        * **{{ ui-key.yacloud.cdn.label_content-query-type }}**: `{{ ui-key.yacloud.cdn.value_query-type-one-origin }}`.
        * **{{ ui-key.yacloud.cdn.label_source-type }}**: `{{ ui-key.yacloud.cdn.value_source-type-balancer }}`.
        * **{{ ui-key.yacloud.cdn.label_balancer }}**: `canary-balancer`.
        * **{{ ui-key.yacloud.cdn.label_ip-address }}**: IP address assigned to the load balancer (the only one in the list).
        * Under **{{ ui-key.yacloud.cdn.label_section-domain }}**:
           * In the **{{ ui-key.yacloud.cdn.label_personal-domain }}** field, specify `cdn.yandexcloud.example`.
           * Click **{{ ui-key.yacloud.cdn.button_add-domain }}** and specify `cdn-staging.yandexcloud.example`.

          {% note alert %}

          The first domain name, `cdn.yandexcloud.example`, will become the primary one, and you will not be able to edit it after you create a CDN resource.

          {% endnote %}

        * Under **{{ ui-key.yacloud.cdn.label_section-additional }}**:

          * In the **{{ ui-key.yacloud.cdn.label_protocol }}** field, select `{{ ui-key.yacloud.common.label_http }}`. 
          * In the **{{ ui-key.yacloud.cdn.label_redirect }}** field, select `{{ ui-key.yacloud.cdn.value_do-not-use }}`.
          * Select **{{ ui-key.yacloud.cdn.field_access }}**.
          * In the **{{ ui-key.yacloud.cdn.label_certificate-type }}** field, specify `{{ ui-key.yacloud.cdn.value_certificate-custom }}` and select a [certificate](#add-certificate) for the `cdn.yandexcloud.example` and `cdn-staging.yandexcloud.example` domain names.
          * In the **{{ ui-key.yacloud.cdn.label_host-header }}** field, select `{{ ui-key.yacloud.cdn.value_host-header-resend }}`.

     1. Click **{{ ui-key.yacloud.common.create }}**.

  1. Enable client redirects from HTTP to HTTPS:

     1. Select the resource you created earlier.
     1. Make sure the certificate status under **{{ ui-key.yacloud.cdn.label_additional }}** has changed to `{{ ui-key.yacloud.cdn.value_certificate-status-ready }}`.
     1. At the top right, click ![image](../../_assets/console-icons/pencil.svg) **{{ ui-key.yacloud.common.edit }}**.
     1. Under **{{ ui-key.yacloud.cdn.label_section-additional }}**, select `{{ ui-key.yacloud.cdn.value_redirect-http-to-https }}` in the **{{ ui-key.yacloud.cdn.label_redirect }}** field.
     1. Click **{{ ui-key.yacloud.common.save }}**.

  1. Enable caching on CDN servers for the resource:

     1. Select the resource you created earlier.
     1. Navigate to **{{ ui-key.yacloud.cdn.label_resource-cache }}**.
     1. At the top right, click ![image](../../_assets/console-icons/pencil.svg) **{{ ui-key.yacloud.common.edit }}**.
     1. Enable **{{ ui-key.yacloud.cdn.label_resource-cache-cdn-cache-enabled }}**.
     1. Click **{{ ui-key.yacloud.common.save }}**.

- {{ yandex-cloud }} CLI {#cli}

  1. If the CDN provider is not activated yet, run this command:
      
      ```bash
      yc cdn provider activate --folder-id <folder_ID> --type gcore
      ```

  1. Create an origin group named `canary-origin-group` and indicate the IP address of the load balancer:
      
      ```bash
      yc cdn origin-group create --name "canary-origin-group" \
        --origin source=<load_balancer_IP_address>:80,enabled=true
      ```

      Result:
      
      ```
      id: "90748"
      folder_id: b1geoelk7fld********
      name: canary-origin-group
      use_next: true
      origins:
      - id: "562449"
        origin_group_id: "90748"
        source: 51.250.10.216:80
        enabled: true
      ```

      For more information about the `yc cdn origin-group create` command, see the [CLI reference](../../cli/cli-ref/cdn/cli-ref/origin-group/create.md).


  1. Copy `origin_group_id` from the previous step and create a CDN by running this command:

      ```bash
      yc cdn resource create \
        --cname cdn.yandexcloud.example \
        --origin-group-id <origin_group_ID> \
        --secondary-hostnames cdn-staging.yandexcloud.example \
        --origin-protocol http \
        --cert-manager-ssl-cert-id <certificate_ID> \
        --forward-host-header
      ```

      Result:
      
      ```
      id: bc843k2yinvq********
      folder_id: b1ge1elk72ld********
      cname: cdn.yandexcloud.example
      ...
      active: true
      ...
      ...
      secondary_hostnames:
      - cdn-staging.yandexcloud.example
      ...
      ```

      For more information about the `yc cdn resource create` command, see the [CLI reference](../../cli/cli-ref/cdn/cli-ref/resource/create.md).

  1. Enable client redirects for the resource:

     ```bash
     yc cdn resource update <resource_ID> --redirect-http-to-https
     ```

- {{ TF }} {#tf}

  1. Add the CDN settings to the configuration file:
      
      ```hcl
      ...

      resource "yandex_cdn_origin_group" "my_group" {
        name     = "canary-origin-group"
        use_next = true
        origin {
         source = "<load_balancer_IP_address>:80"
         backup = false
        }
      }

      resource "yandex_cdn_resource" "my_resource" {

          cname               = "cdn.yandexcloud.example"
          active              = true
          origin_protocol     = "http"
          secondary_hostnames = ["cdn-staging.yandexcloud.example"]
          origin_group_id     = yandex_cdn_origin_group.my_group.id
          ssl_certificate {
            type                   = "certificate_manager"
            certificate_manager_id = "<certificate_ID>"
          }
          options {
              edge_cache_settings    = "345600"
              browser_cache_settings = "1800"
              ignore_cookie          = true
              ignore_query_params    = false
          }

      }
      ```

      For more information, see the description of [yandex_cdn_origin_group]({{ tf-provider-resources-link }}/cdn_origin_group) and [yandex_cdn_resource]({{ tf-provider-resources-link }}/cdn_resource) in the {{ TF }} documentation.

  1. Make sure the configuration files are correct.

     1. In the command line, go to the directory where you created the configuration file.
     1. Run a check using this command:

        ```bash
        terraform plan
        ```

     If the configuration is correct, the terminal will display a list of new resources with their properties. If the configuration contains any errors, {{ TF }} will point them out.

  1. Deploy the cloud resources.

     1. If the configuration does not contain any errors, run this command:

        ```bash
        terraform apply
        ```

     1. Confirm creating the resources: type `yes` in the terminal and press **Enter**.

     After that, your resources will appear in the specified folder. You can check the new resources and their settings using the [management console]({{ link-console-main }}).

  1. Enable client redirects for the resource. Add the following field at the top of the `options` section for the CDN:

      ```hcl
      ...
      options {
        redirect_https_to_http = true
      ...
      ```

  1. Run a check using this command:

      ```bash
      terraform plan
      ```

     If the configuration is correct, the terminal will display a list of resources to update with their properties. If the configuration contains any errors, {{ TF }} will point them out.
  
  1. If there are no errors, run this command:

      ```bash
      terraform apply
      ```

  1. Confirm the resource update: type `yes` in the terminal and press **Enter**.
      
  This enables redirects for the resource.

- API {#api}

  Use the [ResourceService/Create](../../cdn/api-ref/grpc/Resource/create.md) gRPC API call or the [create](../../cdn/api-ref/Resource/create.md) REST API method.

{% endlist %}

## Configure DNS for the service {#configure-dns}

The `cdn.yandexcloud.example` and `cdn-staging.yandexcloud.example` domain names must be mapped to the CDN resource using DNS records.

To configure DNS:

1. Get the domain name of the CDN load balancer:

   {% list tabs group=instructions %}

   - Management console {#console}

     1. In the [management console]({{ link-console-main }}), select `example-folder`.
     1. From the list of services, select **{{ ui-key.yacloud.iam.folder.dashboard.label_cdn }}**.
     1. From the list of CDN resources, select the one with `cdn.yandexcloud.example` as its primary domain name.
     1. From **{{ ui-key.yacloud.cdn.label_dns-settings_title }}** at the bottom of the page, copy the domain name in `cl-********.edgecdn.ru` format.

   {% endlist %}

1. On your DNS hosting provider’s website, navigate to the DNS settings.
1. Create or edit CNAME records for `cdn.yandexcloud.example` and `cdn-staging.yandexcloud.example` so they are pointing at the copied domain name:

   ```
   cdn CNAME cl-********.edgecdn.ru
   cdn-staging CNAME cl-********.edgecdn.ru 
   ```

   {% include [note-dns-aname](../../_includes/cdn/note-dns-aname.md) %} 

   If you use {{ dns-name }}, follow this guide to configure the record:

   {% cut "Configuring DNS records for {{ dns-name }}" %}

   {% list tabs group=instructions %}

   - Management console {#console}

     1. In the [management console]({{ link-console-main }}), select **{{ ui-key.yacloud.iam.folder.dashboard.label_dns }}**.
     1. If you do not have a public DNS zone, create one:

        1. Click **{{ ui-key.yacloud.dns.button_zone-create }}**.
        1. In the **{{ ui-key.yacloud.dns.label_zone }}** field, enter the website domain name with a trailing dot: `yandexcloud.example.`.
        1. In the **{{ ui-key.yacloud.common.type }}** field, select `{{ ui-key.yacloud.dns.label_public }}`.
        1. In the **{{ ui-key.yacloud.common.name }}** field, specify `canary-dns-zone`.
        1. Click **{{ ui-key.yacloud.common.create }}**.

     1. Create a CNAME record for `cdn.yandexcloud.example` in the zone:

        1. Select `canary-dns-zone`.
        1. Click **{{ ui-key.yacloud.dns.button_record-set-create }}**.
        1. In the **{{ ui-key.yacloud.common.name }}** field, specify `cdn`.
        1. In the **{{ ui-key.yacloud.common.type }}** field, specify `CNAME`.
        1. In the **{{ ui-key.yacloud.dns.label_records }}** field, paste the value you copied in `cl-********.edgecdn.ru` format.
        1. Click **{{ ui-key.yacloud.common.create }}**.

     1. Similarly, create a CNAME record for `cdn-staging.yandexcloud.example` in the same zone. In the **{{ ui-key.yacloud.common.name }}** field, specify `cdn-staging`.

   - {{ yandex-cloud }} CLI {#cli}

     1. If you do not have a public DNS zone, create one:

        ```bash
        yc dns zone create \
          --name canary-dns-zone \
          --zone yandexcloud.example. \
          --public-visibility
        ```

        Result:

        ```
        id: dns4rq4taddd********
        folder_id: b1g9hv2loamq********
        created_at: "2021-11-03T11:03:28.847Z"
        name: canary-dns-zone
        zone: yandexcloud.example.
        public_visibility: {}
        ```

        For more information about the `yc dns zone create` command, see the [CLI reference](../../cli/cli-ref/dns/cli-ref/zone/create.md).

     1. In the zone, create a CNAME record for `cdn.yandexcloud.example` and `cdn-staging.yandexcloud.example` with the copied value in `cl-********.edgecdn.ru` format:

        ```bash
        yc dns zone add-records \
          --name canary-dns-zone \
          --record "cdn CNAME cl-********.edgecdn.ru" \
          --record "cdn-staging CNAME cl-********.edgecdn.ru"
        ```

        For more information about the `yc dns zone add-records` command, see the [CLI reference](../../cli/cli-ref/dns/cli-ref/zone/add-records.md).

   - {{ TF }} {#tf}

     1. Add the `canary-dns-zone` settings and the zone’s CNAME records to the configuration file:

        ```
        ...

        resource "yandex_dns_zone" "canary-dns-zone" {
          zone   = "yandexcloud.example."
          name   = "canary-dns-zone"
          public = true
        }

        resource "yandex_dns_recordset" "canary-recordset-production" {
          zone_id = ${yandex_dns_zone.canary-dns-zone.id}
          name    = "cdn"
          type    = "CNAME"
          data    = ["<copied_value_in_cl-********.edgecdn.ru_format>"]
        }

        resource "yandex_dns_recordset" "canary-recordset-staging" {
          zone_id = ${yandex_dns_zone.canary-dns-zone.id}
          name    = "cdn-staging"
          type    = "CNAME"
          data    = ["<copied_value_in_cl-********.edgecdn.ru_format>"]
        }
        ```

        For more information, see the descriptions of [yandex_dns_zone]({{ tf-provider-resources-link }}/dns_zone) and [yandex_dns_recordset]({{ tf-provider-resources-link }}/dns_recordset) in the {{ TF }} documentation.

     1. Make sure the configuration files are correct.

        1. In the command line, go to the directory where you created the configuration file.
        1. Run a check using this command:

           ```bash
           terraform plan
           ```

        If the configuration is correct, the terminal will display a list of new resources with their properties. If the configuration contains any errors, {{ TF }} will point them out. 

     1. Deploy the cloud resources.

        1. If the configuration does not contain any errors, run this command:

           ```bash
           terraform apply
           ```

        1. Confirm creating the resources.

   - API {#api}

     1. Create a DNS zone named `canary-dns-zone` using the [DnsZoneService/Create](../../dns/api-ref/grpc/DnsZone/create.md) gRPC API call or the [create](../../dns/api-ref/DnsZone/create.md) REST API method.
     1. Add the `cdn` and `cdn-staging` CNAME records to the zone with the copied value in `cl-********.edgecdn.ru` format using the [DnsZoneService/UpdateRecordSets](../../dns/api-ref/grpc/DnsZone/updateRecordSets.md) gRPC API call or the [updateRecordSets](../../dns/api-ref/DnsZone/updateRecordSets.md) REST API method.

   {% endlist %}

   {% endcut %}

{% include [after-creation-tip-tutorials](../../_includes/cdn/after-creation-tip-tutorials.md) %}

## Test the service and version switching {#check}

### Check one {#check-one}

Check that `cdn.yandexcloud.example` is mapped to version 1 and `cdn-staging.yandexcloud.example`, to version 2:

1. Open `https://cdn.yandexcloud.example/index.html` in your browser. You should see a page indicating version 1.
1. Delete the `index.html` file from the CDN resource cache:

   {% list tabs group=instructions %}

   - Management console {#console}

     1. In the [management console]({{ link-console-main }}), select `example-folder`.
     1. From the list of services, select **{{ ui-key.yacloud.iam.folder.dashboard.label_cdn }}**.
     1. Select the created CDN resource (the list of resources will contain its primary domain name: `cdn.yandexcloud.example`).
     1. Navigate to the **{{ ui-key.yacloud.cdn.label_resource-content }}** tab.
     1. Click **{{ ui-key.yacloud.cdn.button_resource-content-purge-cache }}**.
     1. In the **{{ ui-key.yacloud.cdn.label_resource-content-purging-cache-type }}** field, select `{{ ui-key.yacloud.cdn.label_resource-content-purging-cache-type-selective }}`.
     1. Enter the path to the uploaded file: `/index.html`.
     1. Click **{{ ui-key.yacloud.cdn.button_resource-content-purge-cache }}**.

   - {{ yandex-cloud }} CLI {#cli}

     1. Get the ID of the CDN resource you created:

        ```bash
        yc cdn resource list
        ```

        Result:

        ```
        +----------------------+--------------------------+--------------------------------+--------------------------------+--------+-------------------------------------------+
        |          ID          |          CNAME           |           CREATED AT           |           UPDATED AT           | ACTIVE |                  OPTIONS                  |
        +----------------------+--------------------------+--------------------------------+--------------------------------+--------+-------------------------------------------+
        | bc837xptmpkh******** | cdn.yandexcloud.example  | seconds:1637235693             | seconds:1637235693             | true   | edge_cache_settings:{enabled:true         |
        |                      |                          | nanos:434085000                | nanos:434115000                |        | default_value:345600}                     |
        |                      |                          |                                |                                |        | cache_http_headers:{value:"accept-ranges" |
        |                      |                          |                                |                                |        | value:"cache-control" value:"connection"  |
        |                      |                          |                                |                                |        | value:"content-encoding"                  |
        |                      |                          |                                |                                |        | value:"content-length"                    |
        |                      |                          |                                |                                |        | value:"content-type"                      |
        |                      |                          |                                |                                |        | value:"date" value:"etag"                 |
        |                      |                          |                                |                                |        | value:"expires" value:"keep-alive"        |
        |                      |                          |                                |                                |        | value:"last-modified" value:"server"      |
        |                      |                          |                                |                                |        | value:"vary"} stale:{enabled:true         |
        |                      |                          |                                |                                |        | value:"error" value:"updating"}           |
        |                      |                          |                                |                                |        | allowed_http_methods:{value:"GET"         |
        |                      |                          |                                |                                |        | value:"POST" value:"HEAD"                 |
        |                      |                          |                                |                                |        | value:"OPTIONS"}                          |
        +----------------------+--------------------------+--------------------------------+--------------------------------+--------+-------------------------------------------+
        ```

        For more information about the `yc cdn resource list` command, see the [CLI reference](../../cli/cli-ref/cdn/cli-ref/resource/list.md).

     1. Delete the file from the cache:

        ```bash
        yc cdn cache purge \
          --resource-id <CDN_resource_ID> \
          --path "/index.html"
        ```

        For more information about the `yc cdn cache purge` command, see the [CLI reference](../../cli/cli-ref/cdn/cli-ref/cache/purge.md).

   - API {#api}

     1. Get the ID of the CDN resource you created using the [ResourceService/List](../../cdn/api-ref/grpc/Resource/list.md) gRPC API call or the [list](../../cdn/api-ref/Resource/list.md) REST API method.
     1. Delete `index.html` from the cache using the [CacheService/Purge](../../cdn/api-ref/grpc/Cache/purge.md) gRPC API call or the [purge](../../cdn/api-ref/Cache/purge.md) method. 

   {% endlist %}

1. Open `https://cdn-staging.yandexcloud.example/index.html` in your browser. You should see a page indicating version 2.

### Canary deployment of version 2 {#canary-v2}

1. Disable caching of the CDN resource and delete `index.html` from the cache:

   {% list tabs group=instructions %}

   - Management console {#console}

     1. In the [management console]({{ link-console-main }}), select `example-folder`.
     1. From the list of services, select **{{ ui-key.yacloud.iam.folder.dashboard.label_cdn }}**.
     1. Select the created CDN resource (the list of resources will contain its primary domain name: `cdn.yandexcloud.example`).
     1. Navigate to the **{{ ui-key.yacloud.cdn.label_resource-cache }}** tab.
     1. Click **{{ ui-key.yacloud.common.edit }}**.
     1. Disable the **{{ ui-key.yacloud.cdn.label_resource-cache-cdn-cache-enabled }}** option.
     1. Click **{{ ui-key.yacloud.common.save }}**.

   - API {#api}

     1. Get the ID of the CDN resource you created using the [ResourceService/List](../../cdn/api-ref/grpc/Resource/list.md) gRPC API call or the [list](../../cdn/api-ref/Resource/list.md) REST API method.
     1. Disable caching using the [ResourceService/Update](../../cdn/api-ref/grpc/Resource/update.md) gRPC API call or the [list](../../cdn/api-ref/Resource/update.md) REST API method. 

   {% endlist %}

1. Delete `index.html` from the cache:

   {% list tabs group=instructions %}

   - Management console {#console}

     1. In the [management console]({{ link-console-main }}), select `example-folder`.
     1. From the list of services, select **{{ ui-key.yacloud.iam.folder.dashboard.label_cdn }}**.
     1. Select the created CDN resource (the list of resources will contain its primary domain name: `cdn.yandexcloud.example`).
     1. Navigate to the **{{ ui-key.yacloud.cdn.label_resource-content }}** tab.
     1. Click **{{ ui-key.yacloud.cdn.button_resource-content-purge-cache }}**.
     1. In the **{{ ui-key.yacloud.cdn.label_resource-content-purging-cache-type }}** field, select `{{ ui-key.yacloud.cdn.label_resource-content-purging-cache-type-selective }}`.
     1. Enter the path to the uploaded file: `/index.html`.
     1. Click **{{ ui-key.yacloud.cdn.button_resource-content-purge-cache }}**.

   - {{ yandex-cloud }} CLI {#cli}

     1. Get the ID of the CDN resource you created:

        ```bash
        yc cdn resource list
        ```

        Result:

        ```
        +----------------------+--------------------------+--------------------------------+--------------------------------+--------+-------------------------------------------+
        |          ID          |          CNAME           |           CREATED AT           |           UPDATED AT           | ACTIVE |                  OPTIONS                  |
        +----------------------+--------------------------+--------------------------------+--------------------------------+--------+-------------------------------------------+
        | bc837xptmpkh******** | cdn.yandexcloud.example  | seconds:1637235693             | seconds:1637235693             | true   | edge_cache_settings:{enabled:true         |
        |                      |                          | nanos:434085000                | nanos:434115000                |        | default_value:345600}                     |
        |                      |                          |                                |                                |        | cache_http_headers:{value:"accept-ranges" |
        |                      |                          |                                |                                |        | value:"cache-control" value:"connection"  |
        |                      |                          |                                |                                |        | value:"content-encoding"                  |
        |                      |                          |                                |                                |        | value:"content-length"                    |
        |                      |                          |                                |                                |        | value:"content-type"                      |
        |                      |                          |                                |                                |        | value:"date" value:"etag"                 |
        |                      |                          |                                |                                |        | value:"expires" value:"keep-alive"        |
        |                      |                          |                                |                                |        | value:"last-modified" value:"server"      |
        |                      |                          |                                |                                |        | value:"vary"} stale:{enabled:true         |
        |                      |                          |                                |                                |        | value:"error" value:"updating"}           |
        |                      |                          |                                |                                |        | allowed_http_methods:{value:"GET"         |
        |                      |                          |                                |                                |        | value:"POST" value:"HEAD"                 |
        |                      |                          |                                |                                |        | value:"OPTIONS"}                          |
        +----------------------+--------------------------+--------------------------------+--------------------------------+--------+-------------------------------------------+
        ```

     1. Delete the file from the cache:

        ```bash
        yc cdn cache purge \
          --resource-id <CDN_resource_ID> \
          --path "/index.html"
        ```

   - API {#api}

     1. Get the ID of the CDN resource you created using the [ResourceService/List](../../cdn/api-ref/grpc/Resource/list.md) gRPC API call or the [list](../../cdn/api-ref/Resource/list.md) REST API method.
     1. Delete `index.html` from the cache using the [CacheService/Purge](../../cdn/api-ref/grpc/Cache/purge.md) gRPC API call or the [purge](../../cdn/api-ref/Cache/purge.md) method. 

   {% endlist %}

1. Configure the `canary-bg-production` backend group so that `canary-backend-green` running version 2 processes 20% of the traffic coming to `cdn.yandexcloud.example`:

   {% list tabs group=instructions %}

   - Management console {#console}

     1. In the [management console]({{ link-console-main }}), select `example-folder`.
     1. From the list of services, select **{{ ui-key.yacloud.iam.folder.dashboard.label_application-load-balancer }}**.
     1. In the left-hand panel, select ![image](../../_assets/console-icons/cubes-3-overlap.svg) **{{ ui-key.yacloud.alb.label_backend-groups }}**.
     1. Select `canary-bg-production`.
     1. Set the `canary-backend-blue` weight to 80 instead of 100:

        1. Under **{{ ui-key.yacloud.alb.label_backends }}**, find `canary-backend-blue`, click ![image](../../_assets/console-icons/ellipsis.svg), and select **{{ ui-key.yacloud.common.edit }}**.
        1. In the **{{ ui-key.yacloud.alb.label_backend-weight }}** field, specify `80`.
        1. Click **{{ ui-key.yacloud.common.save }}**.

     1. Similarly, set the `canary-backend-green` weight to 20 instead of 0.
     1. Click **{{ ui-key.yacloud.common.save }}**.

   - {{ yandex-cloud }} CLI {#cli}

     1. Set the `canary-backend-blue` weight to 80 instead of 100:

        ```bash
        yc alb backend-group update-http-backend \
          --backend-group-name canary-bg-production \
          --name canary-backend-blue \
          --weight 80
        ```

        Result:

        ```
        done (1s)
        id: ds7l9puc18c9********
        name: canary-bg-production
        folder_id: b1g9hv2loamq********
        http:
          backends:
          - name: canary-backend-blue
            backend_weight: "80"
            storage_bucket:
              bucket: <blue_bucket_name>
        created_at: "2021-11-03T10:28:47.680825561Z"
        ```

        For more information about the `yc alb backend-group update-http-backend` command, see the [CLI reference](../../cli/cli-ref/application-load-balancer/cli-ref/backend-group/update-http-backend.md).

     1. Set the `canary-backend-green` weight to 20 instead of 0:

        ```bash
        yc alb backend-group update-http-backend \
          --backend-group-name canary-bg-production \
          --name canary-backend-green \
          --weight 20
        ```

        Result:

        ```
        done (1s)
        id: ds7l9puc18c9********
        name: canary-bg-production
        folder_id: b1g9hv2loamq********
        http:
          backends:
          - name: canary-backend-green
            backend_weight: "20"
            storage_bucket:
              bucket: <green_bucket_name>
        created_at: "2021-11-03T10:28:47.680825561Z"
        ```

   - API {#api}

     Use the [BackendGroupService/UpdateBackend](../../application-load-balancer/api-ref/grpc/BackendGroup/updateBackend.md) gRPC API call or the [updateBackend](../../application-load-balancer/api-ref/BackendGroup/updateBackend.md) REST API method.

   {% endlist %}

1. Open `https://cdn.yandexcloud.example/index.html` in your browser several times. In about 20% of cases, you should see a page indicating version 2, and in the other cases, version 1.
1. Same as at steps 1-2, configure and check the following traffic allocations between the backends:

   1. In the `canary-bg-production` backend group: 50% of traffic to each of the two backends.
   1. In the `canary-bg-production` backend group: All traffic goes to `canary-backend-green`.
   1. In the `canary-bg-staging` backend group (the `cdn-staging.yandexcloud.example` domain name): All traffic goes to `canary-backend-blue`.

1. Re-enable caching:

   {% list tabs group=instructions %}

   - Management console {#console}

     1. In the [management console]({{ link-console-main }}), select `example-folder`.
     1. From the list of services, select **{{ ui-key.yacloud.iam.folder.dashboard.label_cdn }}**.
     1. Select the created CDN resource (the list of resources will contain its primary domain name: `cdn.yandexcloud.example`).
     1. Navigate to the **{{ ui-key.yacloud.cdn.label_resource-cache }}** tab.
     1. Click **{{ ui-key.yacloud.common.edit }}**.
     1. Enable **{{ ui-key.yacloud.cdn.label_resource-cache-cdn-cache-enabled }}**.
     1. Click **{{ ui-key.yacloud.common.save }}**.

   - API {#api}

     1. Get the ID of the CDN resource you created using the [ResourceService/List](../../cdn/api-ref/grpc/Resource/list.md) gRPC API call or the [list](../../cdn/api-ref/Resource/list.md) REST API method.
     1. Enable caching using the [ResourceService/Update](../../cdn/api-ref/grpc/Resource/update.md) gRPC API call or the [list](../../cdn/api-ref/Resource/update.md) REST API method.

   {% endlist %}

### Blue-green deployment for rolling back to version 1 {#blue-green-v1-rollback}

1. Disable caching of the CDN resource and delete `index.html` from the cache:

   {% list tabs group=instructions %}

   - Management console {#console}

     1. In the [management console]({{ link-console-main }}), select `example-folder`.
     1. From the list of services, select **{{ ui-key.yacloud.iam.folder.dashboard.label_cdn }}**.
     1. Select the created CDN resource (the list of resources will contain its primary domain name: `cdn.yandexcloud.example`).
     1. Navigate to the **{{ ui-key.yacloud.cdn.label_resource-cache }}** tab.
     1. Click **{{ ui-key.yacloud.common.edit }}**.
     1. Disable the **{{ ui-key.yacloud.cdn.label_resource-cache-cdn-cache-enabled }}** option.
     1. Click **{{ ui-key.yacloud.common.save }}**.

   - API {#api}

     1. Get the ID of the CDN resource you created using the [ResourceService/List](../../cdn/api-ref/grpc/Resource/list.md) gRPC API call or the [list](../../cdn/api-ref/Resource/list.md) REST API method.
     1. Disable caching using the [ResourceService/Update](../../cdn/api-ref/grpc/Resource/update.md) gRPC API call or the [list](../../cdn/api-ref/Resource/update.md) REST API method. 

   {% endlist %}

1. Delete `index.html` from the cache:

   {% list tabs group=instructions %}

   - Management console {#console}

     1. In the [management console]({{ link-console-main }}), select `example-folder`.
     1. From the list of services, select **{{ ui-key.yacloud.iam.folder.dashboard.label_cdn }}**.
     1. Select the created CDN resource (the list of resources will contain its primary domain name: `cdn.yandexcloud.example`).
     1. Navigate to the **{{ ui-key.yacloud.cdn.label_resource-content }}** tab.
     1. Click **{{ ui-key.yacloud.cdn.button_resource-content-purge-cache }}**.
     1. In the **{{ ui-key.yacloud.cdn.label_resource-content-purging-cache-type }}** field, select `{{ ui-key.yacloud.cdn.label_resource-content-purging-cache-type-selective }}`.
     1. Enter the path to the uploaded file: `/index.html`.
     1. Click **{{ ui-key.yacloud.cdn.button_resource-content-purge-cache }}**.

   - {{ yandex-cloud }} CLI {#cli}

     1. Get the ID of the CDN resource you created:

        ```bash
        yc cdn resource list
        ```

        Result:

        ```
        +----------------------+--------------------------+--------------------------------+--------------------------------+--------+-------------------------------------------+
        |          ID          |          CNAME           |           CREATED AT           |           UPDATED AT           | ACTIVE |                  OPTIONS                  |
        +----------------------+--------------------------+--------------------------------+--------------------------------+--------+-------------------------------------------+
        | bc837xptmpkh******** | cdn.yandexcloud.example  | seconds:1637235693             | seconds:1637235693             | true   | edge_cache_settings:{enabled:true         |
        |                      |                          | nanos:434085000                | nanos:434115000                |        | default_value:345600}                     |
        |                      |                          |                                |                                |        | cache_http_headers:{value:"accept-ranges" |
        |                      |                          |                                |                                |        | value:"cache-control" value:"connection"  |
        |                      |                          |                                |                                |        | value:"content-encoding"                  |
        |                      |                          |                                |                                |        | value:"content-length"                    |
        |                      |                          |                                |                                |        | value:"content-type"                      |
        |                      |                          |                                |                                |        | value:"date" value:"etag"                 |
        |                      |                          |                                |                                |        | value:"expires" value:"keep-alive"        |
        |                      |                          |                                |                                |        | value:"last-modified" value:"server"      |
        |                      |                          |                                |                                |        | value:"vary"} stale:{enabled:true         |
        |                      |                          |                                |                                |        | value:"error" value:"updating"}           |
        |                      |                          |                                |                                |        | allowed_http_methods:{value:"GET"         |
        |                      |                          |                                |                                |        | value:"POST" value:"HEAD"                 |
        |                      |                          |                                |                                |        | value:"OPTIONS"}                          |
        +----------------------+--------------------------+--------------------------------+--------------------------------+--------+-------------------------------------------+
        ```

     1. Delete the file from the cache:

        ```bash
        yc cdn cache purge \
          --resource-id <CDN_resource_ID> \
          --path "/index.html"
        ```

   - API {#api}

     1. Get the ID of the CDN resource you created using the [ResourceService/List](../../cdn/api-ref/grpc/Resource/list.md) gRPC API call or the [list](../../cdn/api-ref/Resource/list.md) REST API method.
     1. Delete `index.html` from the cache using the [CacheService/Purge](../../cdn/api-ref/grpc/Cache/purge.md) gRPC API call or the [purge](../../cdn/api-ref/Cache/purge.md) method. 

   {% endlist %}

1. Re-route all traffic from the `cdn.yandexcloud.example` domain name back to `canary-backend-blue` running version 1:

   {% list tabs group=instructions %}

   - Management console {#console}

     1. In the [management console]({{ link-console-main }}), select `example-folder`.
     1. From the list of services, select **{{ ui-key.yacloud.iam.folder.dashboard.label_application-load-balancer }}**.
     1. In the left-hand panel, select ![image](../../_assets/console-icons/cubes-3-overlap.svg) **{{ ui-key.yacloud.alb.label_backend-groups }}**.
     1. Select `canary-bg-production`.
     1. Set the `canary-backend-blue` weight to 100 instead of 0:

        1. Under **{{ ui-key.yacloud.alb.label_backends }}**, find `canary-backend-blue`, click ![image](../../_assets/console-icons/ellipsis.svg), and select **{{ ui-key.yacloud.common.edit }}**.
        1. In the **{{ ui-key.yacloud.alb.label_backend-weight }}** field, specify `100`.
        1. Click **{{ ui-key.yacloud.common.save }}**.

     1. Similarly, set the `canary-backend-green` weight to 0 instead of 100.
     1. Click **{{ ui-key.yacloud.common.save }}**.

   - {{ yandex-cloud }} CLI {#cli}

     1. Set the `canary-backend-blue` weight to 100 instead of 0:

        ```bash
        yc alb backend-group update-http-backend \
          --backend-group-name canary-bg-production \
          --name canary-backend-blue \
          --weight 100
        ```

        Result:

        ```
        done (1s)
        id: ds7l9puc18c9********
        name: canary-bg-production
        folder_id: b1g9hv2loamq********
        http:
          backends:
          - name: canary-backend-blue
            backend_weight: "100"
            storage_bucket:
              bucket: <blue_bucket_name>
        created_at: "2021-11-03T10:28:47.680825561Z"
        ```

     1. Set the `canary-backend-green` weight to 0 instead of 100:

        ```bash
        yc alb backend-group update-http-backend \
          --backend-group-name canary-bg-production \
          --name canary-backend-green \
          --weight 0
        ```

        Result:

        ```
        done (1s)
        id: ds7l9puc18c9********
        name: canary-bg-production
        folder_id: b1g9hv2loamq********
        http:
          backends:
          - name: canary-backend-green
            backend_weight: "0"
            storage_bucket:
              bucket: <green_bucket_name>
        created_at: "2021-11-03T10:28:47.680825561Z"
        ```

   - API {#api}

     Use the [BackendGroupService/UpdateBackend](../../application-load-balancer/api-ref/grpc/BackendGroup/updateBackend.md) gRPC API call or the [updateBackend](../../application-load-balancer/api-ref/BackendGroup/updateBackend.md) REST API method.   

   {% endlist %}

1. Open `https://cdn.yandexcloud.example/index.html` in your browser several times. Each time, you should see a page indicating version 1.
1. Same as at steps 1-2, switch all traffic from `cdn-staging.yandexcloud.example` to `canary-backend-green` running version 2 and check the switchover in your browser.
1. Re-enable caching:

   {% list tabs group=instructions %}

   - Management console {#console}

     1. In the [management console]({{ link-console-main }}), select `example-folder`.
     1. From the list of services, select **{{ ui-key.yacloud.iam.folder.dashboard.label_cdn }}**.
     1. Select the created CDN resource (the list of resources will contain its primary domain name: `cdn.yandexcloud.example`).
     1. Navigate to the **{{ ui-key.yacloud.cdn.label_resource-cache }}** tab.
     1. Click **{{ ui-key.yacloud.common.edit }}**.
     1. Enable **{{ ui-key.yacloud.cdn.label_resource-cache-cdn-cache-enabled }}**.
     1. Click **{{ ui-key.yacloud.common.save }}**.

   - API {#api}

     1. Get the ID of the CDN resource you created using the [ResourceService/List](../../cdn/api-ref/grpc/Resource/list.md) gRPC API call or the [list](../../cdn/api-ref/Resource/list.md) REST API method.
     1. Enable caching using the [ResourceService/Update](../../cdn/api-ref/grpc/Resource/update.md) gRPC API call or the [list](../../cdn/api-ref/Resource/update.md) REST API method.

   {% endlist %}

## How to delete the resources you created {#clear-out}

To shut down the infrastructure and stop paying for the resources you created:

1. If you set up CNAME records in {{ dns-name }}, [delete](../../dns/operations/zone-delete.md) the `canary-dns-zone` DNS zone.
1. [Delete](../../cdn/operations/resources/delete-resource.md) the CDN resource with `cdn.yandexcloud.example` as the primary domain name.
1. [Delete](../../application-load-balancer/operations/application-load-balancer-delete.md) the `canary-balancer` L7 load balancer.
1. [Delete](../../storage/operations/objects/delete.md) all objects from the blue and green buckets.
1. [Delete](../../storage/operations/buckets/delete.md) the blue and green buckets.
1. [Delete](../../vpc/operations/subnet-delete.md) `canary-subnet-{{ region-id }}-a`, `canary-subnet-{{ region-id }}-b`, and `canary-subnet-{{ region-id }}-d`.
1. [Delete](../../vpc/operations/network-delete.md) `canary-network`.
