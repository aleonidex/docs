# {{ ds-inf }}

{{ ml-platform-name }} provides tools for releasing services available to third-party resources. You can deploy a model trained in {{ ml-platform-name }} and use the same tools to develop a fully featured service based on a Docker image.

To publish a service, {{ ml-platform-name }} provides special resources: [nodes](#node) and [aliases](#statuses).

## Nodes {#node}

A _node_ is an isolated group of specially created VMs (_instances_) the computing load is distributed across. Node instances are created with a preset environment and fixed interpreter state. Depending on your needs, you can select different instance [configurations](../../concepts/configurations.md).

{% note warning %}

{% include [pricing nodes](../../../_includes/datasphere/nodes-pricing-warn.md) %}

{% endnote %}

You can access the nodes via the API. API requests can change the state of the node interpreter. To return to the initial state, you will have to recreate the entire node.

{% note info %}

The maximum size of a request to and a response from the node API is 16 MB.

{% endnote %}

To create a node, specify your organization's cloud [folder](../../../resource-manager/concepts/resources-hierarchy.md#folder) where the node will deploy its instances and store its logs. In the folder, [set up a subnet](../../../vpc/operations/subnet-create.md) with internet access via a [NAT gateway](../../../vpc/operations/create-nat-gateway.md) and create a [service account](../../../iam/operations/sa/create.md) with the `vpc.user` [role](../../../vpc/security/index.md#vpc-user). Specify this service account in the [{{ ml-platform-name }} project settings](../../operations/projects/update.md).

### Node from a model {#models-node}

With nodes from [models](../models/index.md), you can [deploy](../../operations/deploy/node-create.md#from-model) your models saved in {{ ml-platform-name }} as a service accessible via the API. You can create a node from the following models:

* [LightGBM](https://lightgbm.readthedocs.io)
* [ONNX](https://onnx.ai/)
* [Tensorflow.Keras](https://keras.io)
* [Tensorflow](https://www.tensorflow.org)
* [PyTorch](https://pytorch.org)
* [XGBoost](https://lightgbm.readthedocs.io)

If you want to deploy a model that is not supported by the Triton server, you can [convert](https://onnx.ai/sklearn-onnx/auto_tutorial/plot_gexternal_catboost.html) it to ONNX format or deploy a [model based on a Docker image](#docker-node).

To [create a node](../../operations/deploy/node-create.md#from-model) from a model trained outside of {{ ml-platform-name }}, load the model from the file to a variable in the notebook and then [create](../../operations/data/models.md#create) a {{ ml-platform-name }} model from this variable.

{{ ml-platform-name }} uses [Triton Inference Server](https://developer.nvidia.com/triton-inference-server) to deploy nodes from models. You can deploy multiple models saved to {{ ml-platform-name }} on a single node. {{ ml-platform-name }} itself will define the input and output parameters, ensure the model API operation, and then provide monitoring for both the node and the Triton server.

{% note info %}

When deploying PyTorch models, {{ ml-platform-name }} cannot automatically figure out the input and output parameters.

{% endnote %}

### Node from a Docker image {#docker-node}

Nodes deployed from a Docker image hosted in a container registry will run as a fully featured service. The Docker image does not have to contain a model trained in {{ ml-platform-name }}. You can create any image and place it in any registry you find appropriate. To learn how to push a Docker image to a {{ container-registry-full-name }} registry, see [{#T}](../../../container-registry/operations/docker-image/docker-image-push.md).

{% note info %}

To use {{ container-registry-full-name }}, the project service account needs the `container-registry.images.puller` [role](../../../container-registry/security/index.md#container-registry-images-puller).

{% endnote %}

When [creating a node from a Docker image](../../operations/deploy/node-create.md#from-docker), you set the node's API, port you want your service to use, connection time, format of metrics you will collect, and other parameters. Once the node is created, {{ ml-platform-name }} will monitor its state, maintain the operation of the instances, and scale the node within the specified instance range as needed. For instances, you can use the `{{ region-id }}-a` and `{{ region-id }}-b` [availability zones](../../../overview/concepts/geo-scope.md).

To work with a node based on a large model or Docker image, you can connect an additional disk from 10 to 4,096 GB. If a node has multiple instances, a disk will be created for each one.

### Node statuses {#statuses}

A {{ ml-platform-name }} node can have one of the following statuses:

* `Healthy`: Number of instances with the `Healthy` status in the node is equal to the minimum number of required instances.
* `Unhealthy`: Number of instances with the `Healthy` status in the node is below the allowed minimum.
* `Created`: Node has just been created.
* `Suspended`: Node is paused.
* `Deleting`: Node is being deleted.

### Instance statuses {#instance-statuses}

Node instances can have one of the following statuses:

* `Healthy`: Instance is healthy and available for balancing.
* `Unhealthy`: There are issues with the instance and it has been excluded from balancing.
* `Created`: VM has been created for the instance.
* `Started`: Connection has been established with the instance's VM.
* `Preparing`: Instance is getting ready to process requests.
* `Deleting`: Instance is being deleted.
* `Undefined`: Initial state of the instance; VM is not created yet.

## Alias {#alias}

An _alias_ is a special resource used for publishing and updating a service. It allows you to replace nodes and update the running service without affecting the user experience.

[Create an alias](../../../datasphere/operations/deploy/alias-create.md) and use it as your service endpoint. You can update related nodes, balance the load across them, and remove deprecated Docker image versions without affecting the user experience.

## Use cases {#examples}

* [{#T}](../../operations/deploy/node-create.md)
* [{#T}](../../operations/deploy/alias-create.md)
* [{#T}](../../tutorials/node-from-model.md)
* [{#T}](../../tutorials/node-from-docker.md)
* [{#T}](../../tutorials/node-from-docker-fast-api.md)
