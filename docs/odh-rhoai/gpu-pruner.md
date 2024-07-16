# GPU Pruner

In this [repo](https://github.com/wseaton/gpu-pruner), you will find the source code and usage instructions for the **GPU Pruner**.

In certain environments it is very easy for cluster users to **request GPUs** and then (either accidentally or not accidentally) **not consume GPU resources**. We needed a method to proactively identify this type of use, and scale down workloads that are **idle from the GPU hardware perspective**, compared to the default for Notebook resources which is web activity. It is totally possible for a user to consume a GPU from a pod PoV but never actually run a workload on it!

The `gpu-pruner` is a non-destructive idle culler that works with Red Hat OpenShift AI/Kubeflow provided APIs (`InferenceService` and `Notebook`), as well as generic `Deployment`, `ReplicaSet` and `StatefulSet`.

The culler politely pauses workloads that appear idle by scaling them down to 0 replicas. Features may be added in the future for better notifications, but the idea is that a user can simply re-enable the workload when they are ready to test/demo again.

It works is by querying cluster NVIDIA DCGM metrics and looking at a window of GPU utilization per pod. A scaling decision is made by looking up the pods metadata, and using owner-references to figure out the owning resource.
