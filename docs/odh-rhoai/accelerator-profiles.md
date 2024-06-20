## Accelerator Profiles

To effectively use [accelerators in OpenShift AI](https://access.redhat.com/documentation/en-us/red_hat_openshift_ai_self-managed/2.9/html/managing_resources/managing-cluster-resources_cluster-mgmt#overview-of-accelerators_cluster-mgmt), data scientists need to create and manage associated accelerator profiles. 

An accelerator profile is a custom resource definition (CRD) that defines the specifications of an accelerator, and it can be managed via the OpenShift AI dashboard under Settings → Accelerator profiles.

When working with GPU nodes in OpenShift AI, it is essential to set proper taints on each node. This prevents workloads from being scheduled on nodes without specific tolerations. Tolerations are then set in the accelerator profiles associated with each type of GPU.

The taints in the GPU Worker Nodes should be set like this:

```yaml
  taints:
    - effect: NoSchedule
      key: nvidia.com/gpu
      value: NVIDIA-A10G-SHARED
```

If the necessary toleration is not present in the Accelerator Profile, workloads will not be scheduled on nodes with the specified taint. For instance, workbenches selecting other GPU types will not be scheduled on nodes tainted with NVIDIA-A10G-SHARED.

For a detailed guide on configuring and managing accelerator profiles in OpenShift AI, refer to our [repository](https://github.com/rh-aiservices-bu/accelerator-profiles-guide/tree/main).