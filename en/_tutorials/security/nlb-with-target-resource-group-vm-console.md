# Migrating services from an NLB with an instance group as a target to an L7 ALB using the management console


To migrate a service from a network load balancer to an L7 load balancer:

1. [See the recommendations for service migration](#recommendations).
1. [Complete the prerequisite steps](#before-you-begin).
1. [Create a {{ sws-full-name }} profile](#create-profile-sws).
1. [Create an L7 load balancer](#create-alb). At this step, you will associate the {{ sws-name }} security profile with a virtual host of the L7 load balancer.
1. [Migrate the user load from the network load balancer to the L7 load balancer](#migration-nlb-to-alb).

## Service migration recommendations {#recommendations}

{% include [group-vm-recommendations](../_tutorials_includes/migration-from-nlb-to-alb/group-vm-recommendations.md) %}

## Getting started {#before-you-begin}

1. [Create subnets](../../vpc/operations/subnet-create.md) in three availability zones for the L7 load balancer.

1. Create [security groups](../../application-load-balancer/concepts/application-load-balancer.md#security-groups) that allow the L7 load balancer to receive incoming traffic and send it to the targets and allow the targets to receive incoming traffic from the load balancer.

1. When using HTTPS, [add the TLS certificate of your service](../../certificate-manager/operations/import/cert-create.md#create-certificate) to [{{ certificate-manager-full-name }}](../../certificate-manager/).

1. [Reserve a static public IP address with DDoS protection](../../vpc/operations/get-static-ip.md) at Layer 3 – Layer 4 for the L7 load balancer. See the [service migration recommendations](#recommendations).

## Create a {{ sws-name }} security profile {#create-profile-sws}

[Create](../../smartwebsecurity/operations/profile-create.md) a {{ sws-name }} security profile by selecting **{{ ui-key.yacloud.smart-web-security.title_default-template }}**.

Use these settings when creating the profile:

* In the **{{ ui-key.yacloud.smart-web-security.form.label_default-action }}** field, select `{{ ui-key.yacloud.smart-web-security.form.label_action-allow }}`.
* For the **{{ ui-key.yacloud.smart-web-security.overview.label_smart-protection-rule }}** rule, enable **{{ ui-key.yacloud.smart-web-security.overview.column_dry-run-rule }} (dry run)**.

These settings are limited to logging the info about the traffic without applying any actions to it. This will reduce the risk of disconnecting users due to profile configuration issues. As you move along, you will be able to disable **{{ ui-key.yacloud.smart-web-security.overview.column_dry-run-rule }} (dry run)** and configure deny rules in the security profile.

## Create an L7 load balancer {#create-alb}

1. [Create a target group](../../application-load-balancer/operations/target-group-create.md) for the L7 load balancer. Under **{{ ui-key.yacloud.alb.label_targets }}**, select the VMs in your network load balancer's target group.

1. [Create a group of backends](../../application-load-balancer/operations/backend-group-create.md) with the following parameters:

    1. Select `{{ ui-key.yacloud.alb.label_proto-http }}` as the backend group type.
    1. If your service requires requests to be processed within a single user session by the same backend resource, enable [session affinity](../../application-load-balancer/concepts/backend-group.md#session-affinity) for the backend group.
    1. Under **{{ ui-key.yacloud.alb.label_backends }}**, click **{{ ui-key.yacloud.common.add }}** and set up the backend:

        * **{{ ui-key.yacloud.common.type }}**: `{{ ui-key.yacloud.alb.label_target-group }}`.
        * **{{ ui-key.yacloud.alb.label_target-groups }}**: Target group you created earlier.
        * **{{ ui-key.yacloud.alb.label_port }}**: TCP port of your service the VMs accept incoming traffic on.
        * Under **{{ ui-key.yacloud.alb.label_protocol-settings }}**, specify the settings for connecting the L7 load balancer to the backend. Depending on the protocol type on your backend, select `{{ ui-key.yacloud.alb.label_proto-http-plain }}` or `{{ ui-key.yacloud.alb.label_proto-http-tls }}`.
        * Under **HTTP health check**, configure the health check using [these recommendations](../../application-load-balancer/concepts/best-practices.md).
        * Optionally, configure other settings as per [this guide](../../application-load-balancer/operations/backend-group-create.md).

1. [Create an HTTP router](../../application-load-balancer/operations/http-router-create.md).

    Under **{{ ui-key.yacloud.alb.label_virtual-hosts }}**, click **{{ ui-key.yacloud.alb.button_virtual-host-add }}** and configure the virtual host:

    * **{{ ui-key.yacloud.alb.label_authority }}**: Your service domain name.
    * **{{ ui-key.yacloud.alb.label_security-profile-id }}**: {{ sws-name }} profile you created earlier.

        {% note warning %}

        Linking your security profile to a virtual host of the L7 load balancer is the key step for enabling {{ sws-name }}.

        {% endnote %}

    * Click **{{ ui-key.yacloud.alb.button_add-route }}** and configure the route:

        * **{{ ui-key.yacloud.alb.label_path }}**: `Starts with` `/`.
        * **{{ ui-key.yacloud.alb.label_route-action }}**: `{{ ui-key.yacloud.alb.label_route-action-route }}`.
        * **{{ ui-key.yacloud.alb.label_backend-group }}**: Backend group you created earlier.

1. [Create an L7 load balancer](../../application-load-balancer/operations/application-load-balancer-create.md) by selecting **{{ ui-key.yacloud.alb.label_alb-create-form }}**:

    * Specify the previously created security group.
    * Under **{{ ui-key.yacloud.alb.section_allocation-settings }}**, select the subnets in three availability zones for the load balancer nodes. Enable traffic in these subnets.
    * Under **{{ ui-key.yacloud.alb.section_autoscale-settings }}**, specify the [minimum number of resource units](../../application-load-balancer/concepts/application-load-balancer.md#lcu-scaling-settings) per availability zone based on expected load.

        We recommend selecting the number of resource units based on the load expressed in:

        * Number of requests per second (RPS)
        * Number of concurrent active connections
        * Number of new connections per second
        * Traffic processed per second

    * Under **{{ ui-key.yacloud.alb.label_listeners }}**, click **{{ ui-key.yacloud.alb.button_add-listener }}** and set up the listener:

        * Under **{{ ui-key.yacloud.alb.section_external-address-specs }}**, specify:

            * **{{ ui-key.yacloud.alb.label_port }}**: TCP port of your service the VMs accept incoming traffic on.
            * **{{ ui-key.yacloud.common.type }}**: `{{ ui-key.yacloud.alb.label_address-list }}`. Select from the list a public IP address with DDoS protection at L3-L4. For more information, see the [service migration recommendations](#recommendations).
        * Under **{{ ui-key.yacloud.alb.section_common-address-specs }}**, specify:

            * **{{ ui-key.yacloud.alb.label_listener-type }}**: `{{ ui-key.yacloud.alb.label_listener-type-http }}`.
            * **{{ ui-key.yacloud.alb.label_protocol-type }}**: Depending on your service, select `{{ ui-key.yacloud.alb.label_proto-http-plain }}` or `{{ ui-key.yacloud.alb.label_proto-http-tls }}`.
            * If you select `{{ ui-key.yacloud.alb.label_proto-http-tls }}`, specify the TLS certificate you added to {{ certificate-manager-name }} earlier in the **{{ ui-key.yacloud.alb.label_certificate }}** field.
            * **{{ ui-key.yacloud.alb.label_http-router }}**: HTTP router you created earlier.

1. Wait until the L7 load balancer goes `Active`.

1. Go to the new L7 load balancer and select **{{ ui-key.yacloud.alb.label_healthchecks }}** on the left. Make sure you get `HEALTHY` for all the L7 load balancer's health checks.

1. {% include [test](../_tutorials_includes/migration-from-nlb-to-alb/test.md) %}

## Migrate the user load from the network load balancer to the L7 load balancer {#migration-nlb-to-alb}

{% note warning %}

Backend VMs will be recreated during the migration process.

{% endnote %}

If the network load balancer's listener uses a public IP address without DDoS protection, take note of the current [health check](../../network-load-balancer/concepts/health-check.md) settings for the target group in the network load balancer before proceeding to the next step.

1. Change the integration with the target group for the VM group:

    1. In the [management console]({{ link-console-main }}), select the folder containing your VM group.
    1. Select **{{ ui-key.yacloud.iam.folder.dashboard.label_compute }}**.
    1. In the left-hand panel, select ![image](../../_assets/console-icons/layers-3-diagonal.svg) **{{ ui-key.yacloud.compute.switch_groups }}**.
    1. Select the group to update.
    1. In the top-right corner of the page, click **{{ ui-key.yacloud.common.edit }}**.
    1. Under **{{ ui-key.yacloud.compute.groups.create.section_alb }}**, enable **{{ ui-key.yacloud.compute.groups.create.field_target-group-attached }}**.
    1. Specify the name of the L7 load balancer target group and, optionally, other [target group settings](../../compute/concepts/instance-groups/balancers.md#settings-alb).
    1. Click **{{ ui-key.yacloud.compute.groups.create.button_edit }}**.

    When updating the VM group:

    * VMs in the group are automatically [recreated](../../compute/concepts/instance-groups/deploy/instance.md#ch-gr-affect).
    * Targets are removed from the network load balancer target group, and the user load is distributed among the remaining targets. The service becomes partially unavailable to users through the network load balancer during this period.
    * After all targets are deleted, the target group is deleted. The service becomes unavailable through the network load balancer.

    Proceed to the next step without waiting for the VM group update to complete.

1. In the L7 load balancer backend group, [change the backend](../../application-load-balancer/operations/backend-group-update.md#update-backend) target group. Specify only the target group created in the previous step.

    As you perform the operation specified in the previous step, VMs from the VM group will be automatically added to the L7 load balancer target group.

1. Select one of the options to further migrate the user load from the network load balancer to the L7 load balancer depending on whether the network load balancer listener has a public IP address with or without DDoS protection:

    * [The network load balancer listener uses a public IP address with DDoS protection](#ip-with-ddos-protection). During migration, your service will keep its public IP address.
    * [The network load balancer listener uses a public IP address without DDoS protection](#ip-without-ddos-protection). During migration, your service will get a new public IP address.

### The network load balancer listener uses a public IP address with DDoS protection {#ip-with-ddos-protection}

1. Monitor the [status of targets](../../network-load-balancer/operations/check-resource-health.md) of your network load balancer. Wait until the targets are automatically deleted from the target group.

1. [Delete the listener](../../network-load-balancer/operations/listener-remove.md) in the network load balancer to release the static public IP address.

1. In the L7 load balancer, assign to the listener the public IP address previously used by the network load balancer:

    {% list tabs group=instructions %}

    * CLI {#cli}

        {% include [include](../../_includes/cli-install.md) %}

        {% include [default-catalogue](../../_includes/default-catalogue.md) %}

        To change the public IP address, run this command:

        ```bash
        yc application-load-balancer load-balancer update-listener <load_balancer_name> \
           --listener-name <listener_name> \
           --external-ipv4-endpoint address=<service_public_IP_address>,port=<service_port>
        ```

        Where `address` is the public IP address previously used by the network load balancer.

    {% endlist %}

1. After the IP address changes, your service will again be available through the L7 load balancer. Monitor the L7 load balancer's user load on the [load balancer statistics](../../application-load-balancer/operations/application-load-balancer-get-stats.md) charts.

1. Delete the now free static public IP address you selected when creating the L7 load balancer.

1. Optionally, [delete the network load balancer](../../network-load-balancer/operations/load-balancer-delete.md) after migrating the user load to the L7 load balancer.

### The network load balancer listener uses a public IP address without DDoS protection {#ip-without-ddos-protection}

1. Monitor the [status of targets](../../network-load-balancer/operations/check-resource-health.md) of your network load balancer. Wait until the targets are automatically deleted from the target group.

1. [Create a target group](../../network-load-balancer/operations/target-group-create.md) for the network load balancer. Add the VMs recreated when updating the VM group.

1. In the network load balancer, [connect the target group](../../network-load-balancer/operations/target-group-attach.md) created in the previous step. When connecting the target group, configure the health checks the original target group had.

1. Wait until the VM [health checks](../../network-load-balancer/operations/check-resource-health.md) in the network load balancer target group go `Healthy`. This will make your service once again available through the network load balancer.

1. To migrate the user load from a network load balancer to an L7 load balancer, in the DNS service of your domain's public zone, change the A record value for the service domain name to the public IP address of the L7 load balancer. If the public domain zone was created in [{{ dns-full-name }}](../../dns/), change the record using [this guide](../../dns/operations/resource-record-update.md).

    {% note info %}

    The propagation of DNS record updates depends on the time-to-live (TTL) value and the number of links in the DNS request chain. This process can take a long time.

    {% endnote %}

1. As the DNS record updates propagate, follow the increase of requests to the L7 load balancer on the [load balancer statistics](../../application-load-balancer/operations/application-load-balancer-get-stats.md) charts.

1. Follow the decrease of the network load balancer load using the `processed_bytes` and `processed_packets` [load balancer metrics](../../monitoring/metrics-ref/network-load-balancer-ref.md). You can also [create a dashboard](../../monitoring/operations/dashboard/create.md) to visualize these metrics. The absence of load on the network load balancer for a prolonged period of time indicates that the user load has been transferred to the L7 load balancer.

1. Optionally, [delete the network load balancer](../../network-load-balancer/operations/load-balancer-delete.md) after migrating the user load to the L7 load balancer.
