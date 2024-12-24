# Using Kubernetes Image Puller Operator to Speed Up Start-Up Times

OpenShift AI provides a powerful suite of tools for building, training, and deploying AI/ML models. As with any powerful platform, optimizing start-up times for workbenches and model deployments can further enhance the overall user experience.

By using Kubernetes Image Puller Operator, we can pre-download necessary images and speed up the launch of workbenches and model deployments. In this blog post, we'll explore how Kubernetes Image Puller works and how it can be used to optimize start-up times in OpenShift AI environments.

## What is the Kubernetes Image Puller?

Kubernetes Image Puller is a Kubernetes-native solution designed to automatically pre-pull container images to reduce start-up times for containerized workloads. By ensuring that the required container images are pre-fetched and stored locally, the Kubernetes Image Puller minimizes the need for pulling images from image registries during pod startup, thereby improving performance and reducing latency. 

## Why Use Kubernetes Image Puller for OpenShift AI?

Using Kubernetes Image Puller for OpenShift AI can have several benefits:

- **Faster Start-Up Times:** Pre-downloading workbench, runtime images, or ModelCars eliminates delays associated with image pulls during start-up. For example, workbench images often include a variety of tooling and dependencies, making them significantly larger than traditional microservice images.
- **Reduced Latency:** By minimizing the time needed to pull images from registries, you can ensure that your workbenches and model deployments are available faster.
- **Improved Efficiency:** By keeping a local cache of images, the operator can reduce load on image registries and improve network efficiency.

While the Kubernetes Image Puller is highly effective in optimizing start-up times, it's important to ensure that your nodes have enough disk space to accommodate the pre-pulled images. Large images such as workbenches, serving runtimes with CUDA support, or model images in ModelCars can consume significant storage on each node. Insufficient storage may lead to pod evictions due to storage pressure.

## How to Set Up Kubernetes Image Puller?

1) **Install Kubernetes Image Puller Operator:** Begin by installing the Kubernetes Image Puller Operator, a community operator designed to install, configure, and manage Kubernetes Image Puller deployments on your OpenShift cluster. You can typically do this using the OpenShift web console Administrator view > Operators > Operator Hub and searching for or by applying below Subscription on commandline:
   
```yaml
---
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: kubernetes-imagepuller-operator
  namespace: openshift-operators
spec:
  channel: stable
  installPlanApproval: Automatic
  name: kubernetes-imagepuller-operator
  source: community-operators
  sourceNamespace: openshift-marketplace
```

2) **Specify the Images to Pre-download:** List the images you wish to pre-download. The images should be specified in the `name=image-address` format. Then, add them as a semicolon-separated list in the images field of the `KubernetesImagePuller` Custom Resource.

Here's an example configuration:

```yaml
kind: KubernetesImagePuller
apiVersion: che.eclipse.org/v1alpha1
metadata:
  name: image-puller
  namespace: openshift-operators
spec:
  daemonsetName: k8s-image-puller
  images: |
    jupyter-notebook=quay.io/modh/odh-generic-data-science-notebook@sha256:36454fcf796ea284b1e551469fa1705d64c4b2698a8bf9df82a84077989faf5e;
    code-server=image-registry.openshift-image-registry.svc:5000/redhat-ods-applications/code-server-notebook:2024.2;
    openvino-model-server=quay.io/modh/openvino_model_server@sha256:c2d063dc4085455aae87f0d94e63cb7d88ba772662e888cb28f46226a8ac4542
```


- `images` field: Lists the images that will be pre-pulled. Each entry is specified as `name=image-address`. Ensure that each image is identified by its full address, including the registry and tag, to ensure proper resolution during the pull process.

In this example, three images are pre-defined:

- `odh-generic-data-science`: A Jupyter notebook image.
- `code-server`: A code server image .
- `openvino-model-server`: OpenVINO model server image.
- 

3) After applying your configuration, a DaemonSet definition will be created, and a pod will be launched on each node. Each pod will contain the images you specified as containers, and it will begin the image pulling process.If you would like to learn more check [Kubernetes Image Puller GitHub page](https://github.com/che-incubator/kubernetes-image-puller).