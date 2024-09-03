# KServe Timeout Issues

When deploying large models or when relying on node autoscaling with KServe, KServe may timeout before a model has successfully deployed due to the default progress deadline of 10 minutes set by KNative Serving.

When a pod takes longer than 10 minutes to deploy that leverages KNative Serving, like KServe does, KNative Serving will automatically back the pod deployment off and mark it as failed.  This can happen for a number of reasons including deploying large models that take longer than 10m minutes to pull from S3 or if you are leveraging node autoscaling to reduce the consumption of expensive GPU nodes.

To resolve this issue, KNative supports an annotion that can be added to a KServe `ServingRuntime` that can be updated to set a custom progress-deadline for your application:

```yaml
apiVersion: serving.kserve.io/v1alpha1
kind: ServingRuntime
metadata:
  name: my-serving-runtime
spec:
  annotations:
    serving.knative.dev/progress-deadline: 30m
```

It is important to note that the annotation must be set at `spec.annotations` and not `metadata.annotations`.  By setting it in `spec.annotations` the annotation will be copied to the KNative `Service` object that is created by your KServe `InferenceService`.  The annotation on the `Service` will allow KNative to utilize the manually defined progress-deadline.
