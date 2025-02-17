# KServe Timeout Issues

When deploying large models or when relying on node autoscaling with KServe, KServe may timeout before a model has successfully deployed due to the default progress deadline of 10 minutes set by KNative Serving.

When a pod takes longer than 10 minutes to deploy that leverages KNative Serving, like KServe does, KNative Serving will automatically back the pod deployment off and mark it as failed.  This can happen for a number of reasons including deploying large models that take longer than 10m minutes to pull from S3 or if you are leveraging node autoscaling to reduce the consumption of expensive GPU nodes.

To resolve this issue, KNative supports an annotation that can be added to a KServe `InferenceService` or `ServingRuntime` objects which will set a custom timeout for the KNative service object.  Please note that the annotation is not set on the top level objects `metadata.annotations` sections, but instead an annotation field under `spec` that gets applied to the KNative Service object.

It is generally recommended to set the annotation on the `InferenceService` as that object also contains information about the model you are deploying and is the most likely to impact the deployment time of the model server.

## Inference Service

The following annotation on the `InferenceService` will update the default timeout:

```yaml
apiVersion: serving.kserve.io/v1beta1
kind: InferenceService
metadata:
  name: my-model-server
spec:
  predictor:
    annotations:
      serving.knative.dev/progress-deadline: 30m
```

## Serving Runtime

Alternatively, the following annotation on the `ServingRuntime` will also update the default timeout:

```yaml
apiVersion: serving.kserve.io/v1alpha1
kind: ServingRuntime
metadata:
  name: my-serving-runtime
spec:
  annotations:
    serving.knative.dev/progress-deadline: 30m
```
