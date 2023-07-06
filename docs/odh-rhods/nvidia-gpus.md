# Working with NVIDIA GPUs

## Using NVIDIA GPUs on OpenShift

### How does this work?

NVIDIA GPUs can be easily installed on OpenShift. Basically it involves installing two different operators.

The Node Feature Discovery operator will "discover" your cards from a hardware perspective and appropriately label the relevant nodes with this information.

Then the NVIDIA GPU operator will install the necessary drivers and tooling to those nodes. It will also integrate into Kubernetes so that when a Pod requires GPU resources it will be scheduled on the right node, and make sure that the containers are "injected" with the right drivers,  configurations and tools to properly use the GPU.

So from a user perspective, the only thing you have to worry about is asking for GPU resources when defining your pods, with something like:

```yaml
spec:
  containers:
  - name: app
    image: ...
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
        nvidia.com/gpu: 2
      limits:
        memory: "128Mi"
        cpu: "500m"
```

But don't worry, OpenShift Data Science and Open Data Hub take care of this part for you when you launch notebooks, workbenches, model servers, or pipeline runtimes!

### Installation

Here is the documentation you can follow:

- [OpenShift Data Science documentation](https://access.redhat.com/documentation/en-us/red_hat_openshift_data_science_self-managed/1-latest/html/installing_openshift_data_science_self-managed/enabling-gpu-support-in-openshift-data-science_install)
- [NVIDIA documentation (more detailed)](https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/openshift/contents.html)

## Advanced configuration

### Working with taints

In many cases, you will want to restrict access to GPUs, or be able to provide choice between different types of GPUs: simply stating "I want a GPU" is not enough. Also, if you want to make sure that **only the Pods requiring GPUs** end up on GPU-enabled nodes (and not other Pods that just end up being there at random because that's how Kubernetes works...), you're at the right place!

The only supported method at the moment to achieve this is to taint nodes, then apply tolerations on the Pods depending on where you want them scheduled. If you don't pay close attention though when applying taints on Nodes, you may end up with the NVIDIA drivers not installed on those nodes...

In this case you must:

- Apply the taints you need to your Nodes or MachineSets, for example:

```yaml
apiVersion: machine.openshift.io/v1beta1
kind: MachineSet
metadata:
  ...
spec:
  replicas: 1
  selector:
    ...
  template:
    ...
    spec:
      ...
      taints:
        - key: nvidia.com/gpu
          value: "yes"
          effect: NoSchedule
```

- Apply the relevant toleration to the NVIDIA Operator.
    - In the `nvidia-gpu-operator` namespace, get to the Installed Operator menu, open the NVIDIA GPU Operator settings, get to the ClusterPolicy tab, and edit the ClusterPolicy.
![Cluster Policy](img/cluster-policy.png)
    - Edit the YAML, and add the toleration in the daemonset section:

```yaml
apiVersion: nvidia.com/v1
kind: ClusterPolicy
metadata:
  ...
  name: gpu-cluster-policy
spec:
  vgpuDeviceManager: ...
  migManager: ...
  operator: ...
  dcgm: ...
  gfd: ...
  dcgmExporter: ...
  cdi: ...
  driver: ...
  devicePlugin: ...
  mig: ...
  sandboxDevicePlugin: ...
  validator: ...
  nodeStatusExporter: ...
  daemonsets:
    ...
    tolerations:
      - effect: NoSchedule
        key: nvidia.com/gpu
        operator: Exists
  sandboxWorkloads: ...
  gds: ...
  vgpuManager: ...
  vfioManager: ...
  toolkit: ...
...
```

That's it, the operator is now able to deploy all the NVIDIA tooling on the nodes, regardless of their taints. Repeat the procedure for any taint you want to apply to your nodes.

!!! note
    You can apply many different taints at the same time. In this example we only made sure that Pods really needing GPUs are scheduled on a GPU Node. Notebooks, Workbenches or other components from ODH/RHODS that request GPUs will already have this toleration in place. For other Pods you schedule yourself, or using Pipelines, you should make sure the toleration is also applied.

    But you could also add another taint to restrict access to certain type of GPUs. Of course, you would have to apply the matching toleration on the NVIDIA GPU Operator, as well as on the Pods that need to run there.

### Time Slicing (GPU sharing)

Do you want to **share GPUs** between different Pods? Time Slicing is one of the solutions you can use!

The NVIDIA GPU Operator enables oversubscription of GPUs through a set of extended options for the NVIDIA Kubernetes Device Plugin. GPU time-slicing enables workloads that are scheduled on oversubscribed GPUs to interleave with one another.

This mechanism for enabling time-slicing of GPUs in Kubernetes enables a system administrator to define a set of replicas for a GPU, each of which can be handed out independently to a pod to run workloads on. Unlike Multi-Instance GPU (MIG), there is no memory or fault-isolation between replicas, but for some workloads this is better than not being able to share at all. Internally, GPU time-slicing is used to multiplex workloads from replicas of the same underlying GPU.

*[Full reference](https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/gpu-sharing.html)*

#### Configuration

This is a simple example on how to quickly setup Time Slicing on your OpenShift cluster. In this example, we have a MachineSet that can provide nodes with one T4 card each that we want to make "seen" as 4 different cards so that multiple Pods requiring GPUs can be launched, even if we only have one node of this type.

- Create the ConfigMap that will define how we want to slice our GPU:

```yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: time-slicing-config
  namespace: nvidia-gpu-operator
data:
  tesla-t4: |-
    version: v1
    sharing:
      timeSlicing:
        resources:
        - name: nvidia.com/gpu
          replicas: 4
```

!!! note
    - The ConfigMap has to be called `time-slicing-config` and must be created in the `nvidia-gpu-operator` namespace.
    - You can add many different resources with different configurations. You simply have to provide the corresponding Node label that has been applied by the operator, for example `name: nvidia.com/mig-1g.5gb / replicas: 2` if you have a MIG configuration applied to a Node with a A100.
    - You can modify the value of `replicas` to present less/more GPUs. Be warned though: all the Pods on this node will share the GPU memory, with no reservation. The more slices you create, the more risks of OOM errors (out of memory) you get if your Pods are hungry (or even only one!).

- Modify the `gpu-cluster-policy` (accessible from the NVIDIA Operator view in the `nvidia-gpu-operator` namespace) to point to this configuration, and eventually add the default configuration (in case you nodes are not labelled correctly, see below)
  
```yaml
apiVersion: nvidia.com/v1
kind: ClusterPolicy
metadata:
  ...
  name: gpu-cluster-policy
spec:
  ...
  devicePlugin:
    config:
      default: tesla-t4
      name: time-slicing-config
  ...
```

- Apply label to your MachineSet for the specific slicing configuration you want to use on it:

```yaml
apiVersion: machine.openshift.io/v1beta1
kind: MachineSet
metadata:
spec:
  template:
    spec:
      metadata:
        labels:
          nvidia.com/device-plugin.config: tesla-t4
```

### Autoscaler and GPUs

As they are expensive, GPUs are good candidates to put behind an Autoscaler. But due [to this](https://access.redhat.com/solutions/6055181){:target="_blank"} there are some subtleties if you want everything to go smoothly.

#### Configuration

!!! warning
    For the autoscaler to work properly with GPUs, you have to set a specific label to the MachineSet. It will help to Autoscaler figure out (in fact simulate) what it is allowed to do. This is especially true if you have different MachineSets that feature different types of GPUs.

    As per the referenced article above, the `type` for gpus you set through the label cannot be `nvidia.com/gpu` (as you will sometimes find in the standard documentation), because it's not a valid label. Therefore, only for the autoscaling purpose, you should give the `type` a specific name with letters, numbers and dashes only, like `Tesla-T4-SHARED` in this example.

- Edit the MachineSet configuration to add the label that the Autoscaler will expect:

```yaml
apiVersion: machine.openshift.io/v1beta1
kind: MachineSet
...
spec:
  ...
  template:
    ...
    spec:
      metadata:
        labels:
          cluster-api/accelerator: Tesla-T4-SHARED
```

- Create your ClusterAutoscaler configuration (example):

```yaml
apiVersion: autoscaling.openshift.io/v1
kind: ClusterAutoscaler
metadata:
  name: "default"
spec:
  logVerbosity: 4
  maxNodeProvisionTime: 15m
  podPriorityThreshold: -10
  resourceLimits:
    gpus:
      - type: Tesla-T4-SHARED
        min: 0 
        max: 8
  scaleDown: 
    enabled: true 
    delayAfterAdd: 20m 
    delayAfterDelete: 5m 
    delayAfterFailure: 30s 
    unneededTime: 5m
```

!!! note
    The `delayAfterAdd` parameter has to be set higher than standard value as NVIDIA tooling can take a lot of time to deploy, 10-15mn.

- Create the MachineSet Autoscaler:

```yaml
apiVersion: autoscaling.openshift.io/v1beta1
kind: MachineAutoscaler
metadata:
  name: machineset-name
  namespace: "openshift-machine-api"
spec:
  minReplicas: 1 
  maxReplicas: 2
  scaleTargetRef: 
    apiVersion: machine.openshift.io/v1beta1
    kind: MachineSet 
    name: machineset-name 
```

#### Scaling to zero

As GPUs are expensive resources, you may want to **scale down your MachineSet to zero to save on resources**. This will however require some more configuration than just setting the minimum size to zero...

First, some background to help you understand and enable you to solve issues that may arise. You can skip the whole explanation, but it's worth it, so please bear with me.

When you request resources that aren't available, the Autoscaler looks at all the MachineAutoscalers that are available, with their corresponding MachineSets. But how to know which one to use? Well, it will first simulate the provisioning of a Node from each MachineSet, and see if it would fit the request. Of course, if there is already at least one Node available from a given MachineSet, the simulation would be bypassed as the Autoscaler already knows what it will get. If there are different MachineSets that fit and to choose from, the default and only "Expander" available for now in OpenShift to make its decision is `random`. So it will simply picks one totally randomly.

That's all perfect and everything, but for GPUs, if you don't start the Node for real, we don't know what's in it! So that's where we have to help the Autoscaler with a small hint.

- Set this annotation manually if it's not there. It will stick after the first scale up though, along with some other annotations the Autoscaler will add, thanks for its newly discovered knowledge.

```yaml
apiVersion: machine.openshift.io/v1beta1
kind: MachineSet
metadata:
  annotations:
    machine.openshift.io/GPU: "1"
```

Now to the other issue that may happen if you are in an environment with multiple availability zones (AZ)...

Although when you define a MachineSet you can set the AZ and have all the Nodes spawned properly in it, the Autoscaler simulator is not that clever. So it will simply pick a Zone at random. If this is not the one where you want/need your Pod to run, this will be a problem...

For example, you may already have a Persistent Volume attached to you Notebook. If your storage does now support AZ-spanning (like AWS EBS volumes), your PV is bound to a specific AZ. If the Simulator creates a virtual Node in a different AZ, there will be a mismatch, your Pod would not be schedulable on this Node, and the Autoscaler will (wrongly) conclude that it cannot use this MachineSet for a scale up!

Here again, we have to give a hint to the Autoscaler to what the Node will look like in the end.

- In you MachineSet, in the labels that will be added to the node, add information regarding the topology of the Node, as well as for the volumes that may be attached to it. For example:

```yaml
apiVersion: machine.openshift.io/v1beta1
kind: MachineSet
metadata:
spec:
  template:
    spec:
      metadata:
        labels:
          ...
          topology.kubernetes.io/zone: us-east-2a
          topology.ebs.csi.aws.com/zone: us-east-2a
```

With this, the simulated Node will be at the right place, and the Autoscaler will consider the MachineSet valid for scale up!

Reference material:

- [https://cloud.redhat.com/blog/autoscaling-nvidia-gpus-on-red-hat-openshift](https://cloud.redhat.com/blog/autoscaling-nvidia-gpus-on-red-hat-openshift)
- [https://access.redhat.com/solutions/6055181](https://access.redhat.com/solutions/6055181)
- [https://bugzilla.redhat.com/show_bug.cgi?id=1943194](https://bugzilla.redhat.com/show_bug.cgi?id=1943194)
