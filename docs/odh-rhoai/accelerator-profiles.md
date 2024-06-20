## Accelerator Profiles

To effectively use [accelerators in OpenShift AI](https://access.redhat.com/documentation/en-us/red_hat_openshift_ai_self-managed/2-latest/html/managing_resources/managing-cluster-resources_cluster-mgmt#overview-of-accelerators_cluster-mgmt), OpenShift AI Administrators need to create and manage associated accelerator profiles. 

An accelerator profile is a custom resource definition (CRD) that defines specifications for this accelerator. It can be direclty managed via the OpenShift AI Dashboard under Settings → Accelerator profiles.

When working with GPU nodes in OpenShift AI, it is essential to set proper taints on those nodes. This prevents unwanted workloads from being scheduled on them when they don't have specific tolerations set. Those tolerations are configured in the accelerator profiles associated with each type of GPU, then applied to the workloads (Workbenches, Model servers,...) for which you have selected an accelerator profile.

The taints in the GPU Worker Nodes should be set like this:

```yaml
  taints:
    - effect: NoSchedule
      key: nvidia.com/gpu
      value: NVIDIA-A10G-SHARED
```

A corresponding Accelerator profile can then be created to allow workloads to run on this type of node (in this example, nodes having an A10G GPU). Workloads that use another accelerator profile (for another type of GPU for example) or that don't have any Accelerator profile set will not be scheduled on nodes tainted with NVIDIA-A10G-SHARED.

For a detailed guide on configuring and managing accelerator profiles in OpenShift AI, refer to our [repository](https://github.com/rh-aiservices-bu/accelerator-profiles-guide/tree/main).
