# Managing RHOAI with GitOps

GitOps is a common way to manage and deploy applications and resouces on Kubernetes clusters.

This page is intended to provide an overview of the different objects involved in manaing both the installation, administration, and usage of OpenShift AI components using GitOps.  This page is by no means intended to be an exhaustive tutorial on each object and all of the features available in them.

When first implimenting features with GitOps it is highly recommended to deploy the resources manually using the Dashboard, then extract the resources created by the Dashboard and duplicate them in your gitops repo.

## Installation

### Operator Installation

The Red Hat OpenShift AI operator is installed and managed by OpenShift's Operator Lifecycle Manager (OLM) and follows common patterns that can be used to install many different operators.

The Red Hat OpenShift AI operator should be installed in the redhat-ods-operator namespace:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  annotations:
    openshift.io/display-name: "Red Hat OpenShift AI"
  name: redhat-ods-operator
```

After creating the namespace, OLM requires you create an Operator Group to help manage any operators installed in that namespace:

```yaml
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: redhat-ods-operator-group
  namespace: redhat-ods-operator
```

Finally, a Subscription can be created to install the operator:

```yaml
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: rhods-operator
  namespace: redhat-ods-operator
spec:
  channel: stable # <1>
  installPlanApproval: Automatic # <2>
  name: rhods-operator
  source: redhat-operators
  sourceNamespace: openshift-marketplace
```

Subscription Options:

1. Operator versions are managed with the `channel` in OLM.  Users are able to select a channel that corresponds to the upgrade lifecycle they wish to follow and OLM will update versions as they are released on that channel.  To learn more about the available channels and the release lifecycle, please refer to the official [lifecycle documentation](https://access.redhat.com/support/policy/updates/rhoai-sm/lifecycle)
2. Platform administrators also have an option to set how upgrades are applied for the operator with the `installPlanApproval`.  If set to `Automatic` RHOAI is automatically updated to the latest version that is available on the selected channel.  If set to `Manual` administrators will be required to approve all upgrades.

### Component Configuration

When the operator is installed it automatically creates a `DSCInitialization` object that sets up several default configurations.  While it is not required, administrators can choose to manage the `DSCinitialization` object via GitOps. 

```yaml
apiVersion: dscinitialization.opendatahub.io/v1
kind: DSCInitialization
metadata:
  name: default-dsci
spec:
  applicationsNamespace: redhat-ods-applications
  serviceMesh:
    auth:
      audiences:
        - 'https://kubernetes.default.svc'
    controlPlane:
      metricsCollection: Istio
      name: data-science-smcp
      namespace: istio-system
    managementState: Managed # <1>
  trustedCABundle:
    customCABundle: ''
    managementState: Managed # <2>
```

DSCInitialization Options:

1. KServe requires a ServiceMesh instance to be installed on the cluster.  By default the Red Hat OpenShift AI operator will attempt to configure an instance if the ServiceMesh operator is installed.  If your cluster already has ServiceMesh configured, you may choose to set `Unmanaged`
2. User can provide customized certification to be used by components across Red Hat OpenShift AI.

After the operator is installed, a DataScienceCluster object will need to be configured with the different components.  Each component has a `managementState` option which can be set to `Managed` or `Removed`.  Admins can choose which components are installed on the cluster.

```yaml
kind: DataScienceCluster
apiVersion: datasciencecluster.opendatahub.io/v1
metadata:
  name: default
spec:
  components:
    codeflare:
      managementState: Managed
    kserve:
      managementState: Managed
      serving:
        ingressGateway:
          certificate:
            type: OpenshiftDefaultIngress
        managementState: Managed
        name: knative-serving
    trustyai:
      managementState: Managed
    ray:
      managementState: Managed
    kueue:
      managementState: Managed
    workbenches:
      managementState: Managed
    dashboard:
      managementState: Managed
    modelmeshserving:
      managementState: Managed
    datasciencepipelines:
      managementState: Managed
    trainingoperator:
      managementState: Removed
    modelregistry:
      managementState: Removed
      registriesNamespace: rhoai-model-registries
```

After the `DataScienceCluster` object is created, the operator will install and configure the different components on the cluster.  Only one `DataScienceCluster` object can be created on a cluster.

## Administration

### Dashboard Configs

The Red Hat OpenShift AI Dashboard has many different configurable options through the UI that can be managed using the `OdhDashboardConfig` config object.  A default `OdhDashboardConfig` is created when the Dashboard component is installed

```yaml
apiVersion: opendatahub.io/v1alpha
kind: OdhDashboardConfig
metadata:
  name: odh-dashboard-config
  namespace: redhat-ods-applications
  labels:
    app.kubernetes.io/part-of: rhods-dashboard
    app.opendatahub.io/rhods-dashboard: 'true'
spec:
  dashboardConfig:
    enablement: true
    disableAcceleratorProfiles: false
    disableBYONImageStream: false
    disableClusterManager: false
    disableConnectionTypes: true
    disableCustomServingRuntimes: false
    disableDistributedWorkloads: false
    disableHome: false
    disableISVBadges: false
    disableInfo: false
    disableKServe: false
    disableKServeAuth: false
    disableKServeMetrics: false
    disableModelMesh: false
    disableModelRegistry: false
    disableModelServing: false
    disableNIMModelServing: true
    disablePerformanceMetrics: false
    disablePipelines: false
    disableProjectSharing: false
    disableProjects: false
    disableStorageClasses: false
    disableSupport: false
    disableTracking: false
    disableTrustyBiasMetrics: false
  groupsConfig:
    adminGroups: rhods-admins # <1>
    allowedGroups: 'system:authenticated' # <2>
  modelServerSizes: # <3>
    - name: Small
      resources:
        limits:
          cpu: '2'
          memory: 8Gi
        requests:
          cpu: '1'
          memory: 4Gi
    - name: Medium
      resources:
        limits:
          cpu: '8'
          memory: 10Gi
        requests:
          cpu: '4'
          memory: 8Gi
    - name: Large
      resources:
        limits:
          cpu: '10'
          memory: 20Gi
        requests:
          cpu: '6'
          memory: 16Gi
  notebookController:
    enabled: true
    notebookNamespace: rhods-notebooks
    pvcSize: 20Gi # <4>
  notebookSizes: # <5>
    - name: Small
      resources:
        limits:
          cpu: '2'
          memory: 8Gi
        requests:
          cpu: '1'
          memory: 8Gi
    - name: Medium
      resources:
        limits:
          cpu: '6'
          memory: 24Gi
        requests:
          cpu: '3'
          memory: 24Gi
    - name: Large
      resources:
        limits:
          cpu: '14'
          memory: 56Gi
        requests:
          cpu: '7'
          memory: 56Gi
    - name: X Large
      resources:
        limits:
          cpu: '30'
          memory: 120Gi
        requests:
          cpu: '15'
          memory: 120Gi
  templateDisablement: []
  templateOrder:
    - caikit-tgis-runtime
    - kserve-ovms
    - ovms
    - tgis-grpc-runtime
    - vllm-runtime
```

OdhDashboardConfig Options:

1. The Dashboard creates a group called `rhods-admins` by default which users can be added to be granted admin privileges through the Dashboard.  Additionally, any user with the cluster-admin role are admins in the Dashboard by default.  If you wish to change the group which is used to manage admin access, this option can be updated.  It is important to note that this field only impacts a users ability to modify settings in the Dashboard, and will have no impact to a users ability to modify configurations through the Kubernetes objects such as this OdhDashboardConfig object.
2. By default any user that has access to the OpenShift cluster where Red Hat OpenShift AI is installed will have the ability to access the Dashboard.  If you wish to restrict who has access to the Dashboard this option can be updated to another group.  Like the admin group option, this option only impacts the users ability to access the Dashboard and does not restrict their ability to interact directly with the Kubernetes objects used to deploy AI resources.
3. When a user creates a new Model Server through the Dashboard they are presented with an option to choose a server size which will impact the resources available to the pod created for the Model Server.  Administrators have the ability to configure the default options that are available to their users.
4. When creating a new Workbench, users are asked to create storage for their Workbench.  The storage will default to the value set here and users will have the option to choose a different amount of storage if their use case requires more or less storage.  Admins can choose another default storage size that is presented to users by configuring this option.
5. Like the Model Server size, users are presented with a drop down menu of options to select what size of Workbench they wish to create.  Admins have the ability to customize the size options that are presented to users.

### Idle Notebook Culling

Admins have the ability to enable Idle Notebook Culling which will automatically stop any Notebooks/Workbenches that users haven't interacted with in a period of time by creating the following ConfigMap:

```yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: notebook-controller-culler-config
  namespace: redhat-ods-applications
  labels:
    opendatahub.io/dashboard: 'true'
data:
  CULL_IDLE_TIME: '240' # <1>
  ENABLE_CULLING: 'true'
  IDLENESS_CHECK_PERIOD: '1'
```

Idle Notebook Culling Options:

1. The `CULL_IDLE_TIME` looks for metrics from Jupyter to understand when the last time a user interacted with the Workbench and will shut the pod down if it has passed the time set here.  The time is the number of minutes so 240 minutes or 4 hours.

### Accelerator Profiles

Accelerator Profiles allow admins to configure different types of GPU options that they can present to end users and automatically configure a toleration on Workbenches or Model Servers when they are selected.  Admins can configure an Accelerator Profile with the `AcceleratorProfile` object:

```yaml
apiVersion: dashboard.opendatahub.io/v1
kind: AcceleratorProfile
metadata:
  name: nvidia-gpu
  namespace: redhat-ods-applications
spec:
  displayName: nvidia-gpu
  enabled: true
  identifier: nvidia.com/gpu
  tolerations:
    - effect: NoSchedule
      key: nvidia-gpu-only
      operator: Exists
      value: ''
```

### Notebook Images

Red Hat OpenShift AI ships with several out of the box Notebook/Workbench Images but admins can create additional custom images that users can use to launch new Workbench instances.  A Notebook Image is managed with an OpenShift ImageStream object with some required labels:

```yaml
kind: ImageStream
apiVersion: image.openshift.io/v1
metadata:
  annotations:
    opendatahub.io/notebook-image-desc: A custom Jupyter Notebook built for my organization # <1>
    opendatahub.io/notebook-image-name: My Custom Notebook # <2>
  name: my-custom-notebook
  namespace: redhat-ods-applications
  labels: # <3>
    app.kubernetes.io/created-by: byon
    opendatahub.io/dashboard: 'true'
    opendatahub.io/notebook-image: 'true'
spec:
  lookupPolicy:
    local: true
  tags:
    - name: '1.0' # <4>
      annotations:
        opendatahub.io/notebook-python-dependencies: '[{"name":"PyTorch","version":"2.2"}]' # <5>
        opendatahub.io/notebook-software: '[{"name":"Python","version":"v3.11"}]' # <6>
        opendatahub.io/workbench-image-recommended: 'true' # <7>
      from:
        kind: DockerImage
        name: 'quay.io/my-org/my-notebook:latest' # <8>
      importPolicy:
        importMode: Legacy
      referencePolicy:
        type: Source
```

Notebook Image Options:

1. A description for the purpose of the notebook image
2. The name that will be displayed to end users in the drop down menu when creating a Workbench
3. The notebook image requires several labels for them to appear in the Dashboard, including the `app.kubernetes.io/created-by: byon` label.  While traditionally this label is utilized to trace where an object originated from, this label is required for the notebooks to be made available to end users. Ensure `disableBYONImageStream` is set to false with this label
4. Multiple image versions can be configured as part of the same Notebook and users have the ability to select which version of the image they wish to use.  This is helpful if you release updated versions of the Notebook image and you wish to avoid breaking end user environments with package changes and allow them to upgrade as they wish.
5. When selecting a Notebook image users will be presented with some information about the notebook based on the information presented in this annotation.  `opendatahub.io/notebook-python-dependencies` is most commonly used to present information about versions from the most important Python packages that are pre-installed in the Image.
6. Like the python dependencies annotation, the `opendatahub.io/notebook-software` annotation is used to present the end user with information about what software is installed in the Image.  Most commonly this field is used to present information such as the Python version, Jupyter versions, or CUDA versions.
7. When multiple tags are created on the ImageStream, the `opendatahub.io/workbench-image-recommended` is used to control what version of the image is presented by default to end users.  Only one tag should be set to `true` at any give time.
8. Notebook images are generally recommended to be stored in an Image Registry outside of the cluster and referenced in the ImageStream.

While it is possible to build a Notebook Image on an OpenShift cluster and publish it directly to an ImageStream using a BuildConfig or a Tekton Pipeline, it can be challenging to get that image to be seen by the Red Hat OpenShift AI Dashboard.  The Dashboard is only looks at images listed in the `spec.tags` section and images pushed directly to the internal image registry are recorded in the `status.tags`.  As a work around, it is possible to "link" a tag pushed directly to the internal image registry to a tag that is visible by the Dashboard:

```yaml
kind: ImageStream
apiVersion: image.openshift.io/v1
metadata:
  annotations:
    opendatahub.io/notebook-image-desc: A custom Jupyter Notebook built for my organization
    opendatahub.io/notebook-image-name: My Custom Notebook
  name: my-custom-notebook
  namespace: redhat-ods-applications
  labels:
    app.kubernetes.io/created-by: byon
    opendatahub.io/dashboard: 'true'
    opendatahub.io/notebook-image: 'true'
spec:
  lookupPolicy:
    local: false
  tags:
    - name: '1.0'
      annotations:
        opendatahub.io/notebook-python-dependencies: '[{"name":"PyTorch","version":"2.2"}]'
        opendatahub.io/notebook-software: '[{"name":"Python","version":"v3.11"}]'
        opendatahub.io/workbench-image-recommended: 'true'
      from:
        kind: DockerImage
        name: 'image-registry.openshift-image-registry.svc:5000/redhat-ods-applications/my-custom-workbench:latest'
      importPolicy:
        importMode: Legacy
      referencePolicy:
        type: Source
status:
  dockerImageRepository: 'image-registry.openshift-image-registry.svc:5000/redhat-ods-applications/dsp-example'
  tags:
    - tag: latest
```

### Serving Runtime Templates

Red Hat OpenShift AI ships with several out of the box Serving Runtime Templates such as OpenVino and vLLM, but admins have the ability to configure additional templates that allow users to deploy additional ServingRuntimes.  A Serving Runtime template is an OpenShift Template object that wraps around a ServingRuntime object:

```yaml
kind: Template
apiVersion: template.openshift.io/v1
metadata:
  name: trition-serving-runtime
  namespace: redhat-ods-applications
  labels:
    opendatahub.io/dashboard: 'true'
  annotations:
    opendatahub.io/apiProtocol: REST
    opendatahub.io/modelServingSupport: '["multi"]'
objects:
  - apiVersion: serving.kserve.io/v1alpha1
    kind: ServingRuntime
    metadata:
      name: triton-23.05
      labels:
        name: triton-23.05
      annotations:
        maxLoadingConcurrency: '2'
        openshift.io/display-name: Triton runtime 23.05
    spec:
      supportedModelFormats:
        - name: keras
          version: '2'
          autoSelect: true
        - name: onnx
          version: '1'
          autoSelect: true
        - name: pytorch
          version: '1'
          autoSelect: true
        - name: tensorflow
          version: '1'
          autoSelect: true
        - name: tensorflow
          version: '2'
          autoSelect: true
        - name: tensorrt
          version: '7'
          autoSelect: true
        - name: sklearn
          version: "1"
          autoSelect: true         
      protocolVersions:
        - grpc-v2
      multiModel: true
      grpcEndpoint: 'port:8085'
      grpcDataEndpoint: 'port:8001'
      volumes:
        - name: shm
          emptyDir:
            medium: Memory
            sizeLimit: 2Gi
      containers:
        - name: triton
          image: 'nvcr.io/nvidia/tritonserver:23.05-py3'
          command:
            - /bin/sh
          args:
            - '-c'
            - 'mkdir -p /models/_triton_models; chmod 777 /models/_triton_models; exec tritonserver "--model-repository=/models/_triton_models" "--model-control-mode=explicit" "--strict-model-config=false" "--strict-readiness=false" "--allow-http=true" "--allow-sagemaker=false" '
          volumeMounts:
            - name: shm
              mountPath: /dev/shm
          resources:
            requests:
              cpu: 500m
              memory: 1Gi
            limits:
              cpu: '5'
              memory: 1Gi
          livenessProbe:
            exec:
              command:
                - curl
                - '--fail'
                - '--silent'
                - '--show-error'
                - '--max-time'
                - '9'
                - 'http://localhost:8000/v2/health/live'
            initialDelaySeconds: 5
            periodSeconds: 30
            timeoutSeconds: 10
      builtInAdapter:
        serverType: triton
        runtimeManagementPort: 8001
        memBufferBytes: 134217728
        modelLoadingTimeoutMillis: 90000
```

## End User Resources

### Data Science Projects

Data Science Projects are simply a normal OpenShift Project with an extra label to distinguish them from normal OpenShift projects by the Red Hat OpenShift AI Dashboard.  Like OpenShift Projects it is recommended to create a `namespace` object and allow OpenShift to create the corresponding `project` object:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: my-data-science-project
  labels:
    opendatahub.io/dashboard: "true"
```

Additionally, when a project going to be utilized by ModelMesh for Multi-model serving, there is an additional ModelMesh label that should be applied to the namespace:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: my-multi-model-serving-project
  labels:
    opendatahub.io/dashboard: "true"
    modelmesh-enabled: "true"
```

### Workbenches

Workbench objects are managed using the Notebook custom resource.  The Notebook object contains a fairly complex configuration, with many items that will be autogenerated, and required annotations to display correctly in the Dashboard.  The Notebook object essentially acts as a wrapper around a normal pod definition and you will find many similarities to managing a pod with options such as the image, pvcs, secrets, etc.  

It is highly recommended to thoroughly test any Notebook configurations configured with GitOps.

```yaml
apiVersion: kubeflow.org/v1
kind: Notebook
metadata:
  annotations:
    notebooks.opendatahub.io/inject-oauth: 'true' # <1>
    opendatahub.io/image-display-name: Minimal Python
    notebooks.opendatahub.io/oauth-logout-url: 'https://rhods-dashboard-redhat-ods-applications.apps.my-cluster.com/projects/my-data-science-project?notebookLogout=my-workbench'
    opendatahub.io/accelerator-name: ''
    openshift.io/description: ''
    openshift.io/display-name: my-workbench
    notebooks.opendatahub.io/last-image-selection: 's2i-minimal-notebook:2024.1'
    notebooks.kubeflow.org/last_activity_check_timestamp: '2024-07-30T20:43:25Z'
    notebooks.opendatahub.io/last-size-selection: Small
    opendatahub.io/username: 'kube:admin'
    notebooks.kubeflow.org/last-activity: '2024-07-30T20:27:25Z'
  name: my-workbench
  namespace: my-data-science-project
spec:
  template:
    spec:
      affinity: {}
      containers:
        - resources: # <2>
            limits:
              cpu: '2'
              memory: 8Gi
            requests:
              cpu: '1'
              memory: 8Gi
          readinessProbe:
            failureThreshold: 3
            httpGet:
              path: /notebook/my-data-science-project/my-workbench/api
              port: notebook-port
              scheme: HTTP
            initialDelaySeconds: 10
            periodSeconds: 5
            successThreshold: 1
            timeoutSeconds: 1
          name: my-workbench
          livenessProbe:
            failureThreshold: 3
            httpGet:
              path: /notebook/my-data-science-project/my-workbench/api
              port: notebook-port
              scheme: HTTP
            initialDelaySeconds: 10
            periodSeconds: 5
            successThreshold: 1
            timeoutSeconds: 1
          env:
            - name: NOTEBOOK_ARGS
              value: |-
                --ServerApp.port=8888
                --ServerApp.token=''
                --ServerApp.password=''
                --ServerApp.base_url=/notebook/my-data-science-project/my-workbench
                --ServerApp.quit_button=False
                --ServerApp.tornado_settings={"user":"kube-3aadmin","hub_host":"https://rhods-dashboard-redhat-ods-applications.apps.my-cluster.com", "hub_prefix":"/projects/my-data-science-project"}
            - name: JUPYTER_IMAGE
              value: 'image-registry.openshift-image-registry.svc:5000/redhat-ods-applications/s2i-minimal-notebook:2024.1'
            - name: PIP_CERT
              value: /etc/pki/tls/custom-certs/ca-bundle.crt
            - name: REQUESTS_CA_BUNDLE
              value: /etc/pki/tls/custom-certs/ca-bundle.crt
            - name: SSL_CERT_FILE
              value: /etc/pki/tls/custom-certs/ca-bundle.crt
            - name: PIPELINES_SSL_SA_CERTS
              value: /etc/pki/tls/custom-certs/ca-bundle.crt
            - name: GIT_SSL_CAINFO
              value: /etc/pki/tls/custom-certs/ca-bundle.crt
          ports:
            - containerPort: 8888
              name: notebook-port
              protocol: TCP
          imagePullPolicy: Always
          volumeMounts:
            - mountPath: /opt/app-root/src
              name: my-workbench
            - mountPath: /dev/shm
              name: shm
            - mountPath: /etc/pki/tls/custom-certs/ca-bundle.crt
              name: trusted-ca
              readOnly: true
              subPath: ca-bundle.crt
          image: 'image-registry.openshift-image-registry.svc:5000/redhat-ods-applications/s2i-minimal-notebook:2024.1' # <3>
          workingDir: /opt/app-root/src
        - resources: # <4>
            limits:
              cpu: 100m
              memory: 64Mi
            requests:
              cpu: 100m
              memory: 64Mi
          readinessProbe:
            failureThreshold: 3
            httpGet:
              path: /oauth/healthz
              port: oauth-proxy
              scheme: HTTPS
            initialDelaySeconds: 5
            periodSeconds: 5
            successThreshold: 1
            timeoutSeconds: 1
          name: oauth-proxy
          livenessProbe:
            failureThreshold: 3
            httpGet:
              path: /oauth/healthz
              port: oauth-proxy
              scheme: HTTPS
            initialDelaySeconds: 30
            periodSeconds: 5
            successThreshold: 1
            timeoutSeconds: 1
          env:
            - name: NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
          ports:
            - containerPort: 8443
              name: oauth-proxy
              protocol: TCP
          imagePullPolicy: Always
          volumeMounts:
            - mountPath: /etc/oauth/config
              name: oauth-config
            - mountPath: /etc/tls/private
              name: tls-certificates
          image: 'registry.redhat.io/openshift4/ose-oauth-proxy@sha256:4bef31eb993feb6f1096b51b4876c65a6fb1f4401fee97fa4f4542b6b7c9bc46'
          args:
            - '--provider=openshift'
            - '--https-address=:8443'
            - '--http-address='
            - '--openshift-service-account=my-workbench'
            - '--cookie-secret-file=/etc/oauth/config/cookie_secret'
            - '--cookie-expire=24h0m0s'
            - '--tls-cert=/etc/tls/private/tls.crt'
            - '--tls-key=/etc/tls/private/tls.key'
            - '--upstream=http://localhost:8888'
            - '--upstream-ca=/var/run/secrets/kubernetes.io/serviceaccount/ca.crt'
            - '--email-domain=*'
            - '--skip-provider-button'
            - '--openshift-sar={"verb":"get","resource":"notebooks","resourceAPIGroup":"kubeflow.org","resourceName":"my-workbench","namespace":"$(NAMESPACE)"}'
            - '--logout-url=https://rhods-dashboard-redhat-ods-applications.apps.my-cluster.com/projects/my-data-science-project?notebookLogout=my-workbench'
      enableServiceLinks: false
      serviceAccountName: my-workbench
      volumes:
        - name: my-workbench
          persistentVolumeClaim:
            claimName: my-workbench
        - emptyDir:
            medium: Memory
          name: shm
        - configMap:
            items:
              - key: ca-bundle.crt
                path: ca-bundle.crt
            name: workbench-trusted-ca-bundle
            optional: true
          name: trusted-ca
        - name: oauth-config
          secret:
            defaultMode: 420
            secretName: my-workbench-oauth-config
        - name: tls-certificates
          secret:
            defaultMode: 420
            secretName: my-workbench-tls
```

1. The Notebook object contains several different annotations that are used by OpenShift AI, but the `inject-oauth` annotation is one of the most important.  There are several oauth based configurations that in the Notebook that will be automatically generated by this annotation, allowing you to exclude a large amount of notebook configuration from what is contained in your GitOps repo.
2. While selecting the resource size through the Dashboard you have more limited options for what sizes you can select, you can choose any size you wish for your notebook through the YAML.  By selecting a non-standard size the Dashboard may report an "unknown" size however.
3. Just like the resources size, you can choose any number of images for the Notebook, including ones that are not available in the Dashboard.  By selecting a non-standard notebook image the Dashboard may report issues however.
4. The oauth-proxy container is one such item that can be removed from the gitops based configuration when utilizing the `inject-oauth` annotation.  Instead of including this section and some other oauth related configurations, you can simply rely on the annotation, and allow the Notebook controller to manage this portion of the object for you.  This will help to prevent problems when upgrading RHOAI.

Users have the ability to start and stop the Workbench to help conserve resources on the cluster.  To stop a Notebook, the following annotation should be applied to the Notebook object:

```yaml
metadata:
  annotations:
    kubeflow-resource-stopped: '2024-07-30T20:52:37Z'
```

Generally, you do not want to include this annotation in your GitOps configuration, as it will enforce the Notebook to be shutdown, not allowing users to start their Notebooks.  The value of the annotation doesn't matter, but by default the Dashboard will apply a timestamp with the time the Notebook was shut down.

### Data Science Connections

A Data Science Connection is a normal Kubernetes Secret object with several annotations that follow a specific format for the data.

```yaml
kind: Secret
apiVersion: v1
type: Opaque
metadata:
  name: aws-connection-my-dataconnection # <1>
  labels:
    opendatahub.io/dashboard: 'true' # <2>
    opendatahub.io/managed: 'true'
  annotations:
    opendatahub.io/connection-type: s3 # <3>
    openshift.io/display-name: my-dataconnection # <4>
data: # <5>
  AWS_ACCESS_KEY_ID: dGVzdA==
  AWS_DEFAULT_REGION: 'dGVzdA=='
  AWS_S3_BUCKET: 'dGVzdA=='
  AWS_S3_ENDPOINT: dGVzdA==
  AWS_SECRET_ACCESS_KEY: dGVzdA==
```

1. When creating a data connection through the Dashboard, the name is automatically generated as `aws-connection-<your-entered-name>`.  When generating the data connection from outside of the Dashboard, you do not need to follow this naming convention.
2. The `opendatahub.io/dashboard: 'true'` label is used to help determine what secrets to display in the Dashboard.  This option must be set to true if you wish for it to be available in the UI.
3. At this point in time, the Dashboard only supports the S3 as a connection-type, but other types may be supported in the future.
4. The name of the data connection as it will appear in the Dashboard UI
5. Like all secrets, data connections data is stored in a base64 encoding.  This data is not secure to be stored in this format and users should instead look into tools such as SealedSecrets or ExternalSecrets to manage secret data in a gitops workflow.

### Data Science Pipelines

When setting up a new project, a Data Science Pipeline instance needs to be created using the DataSciencePipelineApplication object.  The DSPA will create the pipeline servers for the project and allow users to begin interacting with Data Science Pipelines.

```yaml
apiVersion: datasciencepipelinesapplications.opendatahub.io/v1alpha1
kind: DataSciencePipelinesApplication
metadata:
  name: dspa # <1>
  namespace: my-data-science-project
spec:
  apiServer:
    caBundleFileMountPath: ''
    stripEOF: true
    dbConfigConMaxLifetimeSec: 120
    applyTektonCustomResource: true
    caBundleFileName: ''
    deploy: true
    enableSamplePipeline: false
    autoUpdatePipelineDefaultVersion: true
    archiveLogs: false
    terminateStatus: Cancelled
    enableOauth: true
    trackArtifacts: true
    collectMetrics: true
    injectDefaultScript: true
  database:
    disableHealthCheck: false
    mariaDB:
      deploy: true
      pipelineDBName: mlpipeline
      pvcSize: 10Gi
      username: mlpipeline
  dspVersion: v2
  objectStorage:
    disableHealthCheck: false
    enableExternalRoute: false
    externalStorage: # <2>
      basePath: ''
      bucket: pipelines
      host: 'minio.ai-example-training.svc.cluster.local:9000'
      port: ''
      region: us-east-1
      s3CredentialsSecret:
        accessKey: AWS_SECRET_ACCESS_KEY
        secretKey: AWS_ACCESS_KEY_ID
        secretName: aws-connection-my-dataconnection
      scheme: http
  persistenceAgent:
    deploy: true
    numWorkers: 2
  scheduledWorkflow:
    cronScheduleTimezone: UTC
    deploy: true
```

1. The Dashboard expects to look for an object called `dspa` and it is not recommended to deploy more than a single DataSciencePipelineApplication object in a single namespace.
2. The externalStorage is a critical configuration for setting up S3 backend storage for Data Science Pipelines.  While using the dashboard you are required to configure the connection details.  While you can import these details from a data connection, it will create a separate secret containing the s3 secrets instead of reusing the existing data connection secret.

Once a Data Science Pipeline instance has been created, users may wish to configure and manage their pipelines via GitOps.  It is important to note that Data Science Pipelines is not "gitops friendly".  While working with Elyra or a kfp pipeline, users are required to manually upload a pipeline file to the Dashboard which does not generate a corresponding Kubernetes object.  Additionally, when executing a pipeline run, uses may find a ArgoWorkflow object that is generated for the run, however this object can not be re-used in a gitops application to create a new pipeline run in Data Science Pipelines.

As a work around, one common pattern to "gitops-ify" a Data Science Pipeline while using kfp is to instead create a Tekton pipeline that either compiles the pipeline, and uses the kfp skd to upload the pipeline to Data Science Pipelines, or the kfp sdk can automatically trigger a new pipeline run directly from your pipeline code.

### Model Serving

Model Serving in RHOAI has two different flavors, Single Model Serving (KServe) and Multi-Model Serving (ModelMesh).  Both model server options utilize the same Kubernetes objects (ServingRuntime and InferenceService), but have different controllers managing them.  

As mentioned in the Data Science Project section, in order to utilize ModelMesh, a `modelmesh-enabled` label must be applied to the namespace:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: my-multi-model-serving-project
  labels:
    opendatahub.io/dashboard: "true"
    modelmesh-enabled: "true"
```

When creating a model server through the Dashboard, users can select a "Serving Runtime Template" which will create a ServingRuntime instance in their namespace which can be managed via GitOps.  The ServingRuntime helps to define different things such as the container definition, the supported model types, and available ports.

```yaml
apiVersion: serving.kserve.io/v1alpha1
kind: ServingRuntime
metadata:
  annotations: # <1>
    enable-route: 'true'
    opendatahub.io/accelerator-name: ''
    opendatahub.io/apiProtocol: REST
    opendatahub.io/recommended-accelerators: '["nvidia.com/gpu"]'
    opendatahub.io/template-display-name: OpenVINO Model Server
    opendatahub.io/template-name: ovms
    openshift.io/display-name: multi-model-server
  name: multi-model-server
  labels:
    opendatahub.io/dashboard: 'true'
spec:
  supportedModelFormats:
    - autoSelect: true
      name: openvino_ir
      version: opset1
    - autoSelect: true
      name: onnx
      version: '1'
    - autoSelect: true
      name: tensorflow
      version: '2'
  builtInAdapter:
    env:
      - name: OVMS_FORCE_TARGET_DEVICE
        value: AUTO
    memBufferBytes: 134217728
    modelLoadingTimeoutMillis: 90000
    runtimeManagementPort: 8888
    serverType: ovms
  multiModel: true
  containers:
    - args:
        - '--port=8001'
        - '--rest_port=8888'
        - '--config_path=/models/model_config_list.json'
        - '--file_system_poll_wait_seconds=0'
        - '--grpc_bind_address=127.0.0.1'
        - '--rest_bind_address=127.0.0.1'
      image: 'quay.io/modh/openvino_model_server@sha256:5d04d405526ea4ce5b807d0cd199ccf7f71bab1228907c091e975efa770a4908'
      name: ovms
      resources:
        limits:
          cpu: '2'
          memory: 8Gi
        requests:
          cpu: '1'
          memory: 4Gi
      volumeMounts:
        - mountPath: /dev/shm
          name: shm
  protocolVersions:
    - grpc-v1
  grpcEndpoint: 'port:8085'
  volumes:
    - emptyDir:
        medium: Memory
        sizeLimit: 2Gi
      name: shm
  replicas: 1
  tolerations: []
  grpcDataEndpoint: 'port:8001'
```

While KServe and ModelMesh share the same object definition, they have some subtle differences, in particular the annotations that are available on them.  `enable-route` is one annotation that is available on a ModelMesh ServingRuntime that is not available on a KServe based Model Server.

The InferenceService is responsible for a definition of the model that will be deployed as well as which ServingRuntime it should use to deploy it.

```yaml
apiVersion: serving.kserve.io/v1beta1
kind: InferenceService
metadata:
  annotations:
    openshift.io/display-name: fraud-detection-model
    serving.kserve.io/deploymentMode: ModelMesh
  name: fraud-detection-model
  labels:
    opendatahub.io/dashboard: 'true'
spec:
  predictor:
    model:
      modelFormat:
        name: onnx
        version: '1'
      name: ''
      resources: {}
      runtime: multi-model-server  # <1>
      storage:
        key: aws-connection-multi-model
        path: models/fraud-detection-model/frauddetectionmodel.onnx
```

1. The runtime must match the name of the ServingRuntime object that you wish to utilize to deploy the model.

One major difference between ModelMesh and KServe is which object is responsible for creating and managing the pod where the model is deployed.  

With KServe, the ServingRuntime acts as a "pod template" and each InferenceService creates it's own pod to deploy a model.  A ServingRuntime can be used by multiple InferenceServices and each InferenceService will create a separate pod to deploy a model.

By contrast, a ServingRuntime creates a pod with ModelMesh, and the InferenceService simply tells the model server pod what models to load and from where.  With ModelMesh a single ServingRuntime with multiple InferenceServices will create a single pod to load all of the models.

For auth token to work, several parts need to be adapted
1. two extra annotations need to be added on InferenceService object `fraud-detection-mode`

```yaml
apiVersion: serving.kserve.io/v1beta1
kind: InferenceService
metadata:
  annotations:
    security.opendatahub.io/enable-auth: 'true'
    serving.knative.openshift.io/enablePassthrough: 'true'
    ...
  name: fraud-detection-model
  ...
```

2. your service account `fraud-detection-s` created for inference requires proper permission to get InferenceService object `fraud-detection-mode`

```yaml
---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: fraud-detection-get-role
  namespace: my-multi-model-serving-project
  labels:
    opendatahub.io/dashboard: 'true'
rules:
  - verbs:
      - get
    apiGroups:
      - serving.kserve.io
    resources:
      - inferenceservices
    resourceNames:
      - fraud-detection-model
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: fraud-detection-get-rolebinding
  namespace: my-multi-model-serving-project
  labels:
    opendatahub.io/dashboard: 'true'
subjects:
  - kind: ServiceAccount
    name: fraud-detection-sa
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: fraud-detection-get-role
```

3. Lastly, create secret attaching to your service account `fraud-detection-sa`

```yaml
kind: Secret
apiVersion: v1
  annotations:
    kubernetes.io/service-account.name: fraud-detection-sa
  name: auth-token-secret
type: kubernetes.io/service-account-token
```

## ArgoCD Health Checks

Out of the box, ArgoCD and OpenShift GitOps ship with a health check for a KServe InferenceService which is not compatible with a ModelMesh InferenceService.  When attempting to deploy a ModelMesh based InferenceService, ArgoCD will report the object as degraded.

Custom health checks can be added to your ArgoCD instance that are compatible with both KServe and ModelMesh as well as other RHOAI objects to resolve this issue.  The Red Hat AI Services Practice maintains several custom health checks that you can utilize in your own ArgoCD instance [here](https://github.com/redhat-ai-services/ai-accelerator/tree/main/components/operators/openshift-gitops/instance/components/health-check-openshift-ai).
