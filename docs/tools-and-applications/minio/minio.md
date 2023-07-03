# Minio

## What is it?

[Minio](https://min.io/) is a high-performance, S3 compatible **object store**. It can be deployed on a wide variety of platforms, and it comes in multiple [flavors](https://min.io/pricing).

## Why this guide?

This guide is a very quick way of deploying the community version of Minio in order to quickly setup a fully standalone Object Store, in an OpenShift Cluster. This can then be used for various prototyping tasks that require Object Storage.

Note that nothing in this guide should be used in production-grade environments. Also, Minio is not included in RHODS, and Red Hat does not provide support for Minio.

## Pre-requisites

* Access to an OpenShift cluster
* Namespace-level admin permissions, or permission to create your own project

## Deploying Minio on OpenShift

### Create a Data Science Project (Optional)

If you already have your own Data Science Project, or OpenShift project, you can skip this step.

1. If your cluster already has [Red Hat OpenShift Data Science](https://developers.redhat.com/products/red-hat-openshift-data-science/overview) installed, you can use the Dashboard Web Interface to create a Data Science project.
1. Simply navigate to **Data Science Projects**
1. And click **Create Project**
1. Choose a name for your project (here, **Showcase**) and click **Create**:

    ![alt_text](img/create.project.png)

1. Make sure to make a note of the Resource name, in case it's different from the name.

### Log on to your project in OpenShift Console

1. Go to your cluster's OpenShift Console:

    ![alt_text](img/openshift.console.png)

1. Make sure you use the **Administrator** view, not the developer view.
1. Go to **Workloads** then **Pods**, and confirm the selected **project** is the right one

    ![alt_text](img/workloads.pods.png)

1. You now have a project in which to deploy Minio

### Deploy Minio in your project

1. Click on the **+** ("Import YAML") button:

    ![alt_text](img/import.yaml.png)

1. Paste the following YAML in the box, but **don't press ok yet!**:
  ```yaml
  ---
  kind: PersistentVolumeClaim
  apiVersion: v1
  metadata:
    name: minio-pvc
  spec:
    accessModes:
      - ReadWriteOnce
    resources:
      requests:
        storage: 20Gi
    volumeMode: Filesystem
  ---
  kind: Secret
  apiVersion: v1
  metadata:
    name: minio-secret
  stringData:
    # change the username and password to your own values.
    # ensure that the user is at least 3 characters long and the password at least 8
    minio_root_user: minio
    minio_root_password: minio123
  ---
  kind: Deployment
  apiVersion: apps/v1
  metadata:
    name: minio
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: minio
    template:
      metadata:
        creationTimestamp: null
        labels:
          app: minio
      spec:
        volumes:
          - name: data
            persistentVolumeClaim:
              claimName: minio-pvc
        containers:
          - resources:
              limits:
                cpu: 250m
                memory: 1Gi
              requests:
                cpu: 20m
                memory: 100Mi
            readinessProbe:
              tcpSocket:
                port: 9000
              initialDelaySeconds: 5
              timeoutSeconds: 1
              periodSeconds: 5
              successThreshold: 1
              failureThreshold: 3
            terminationMessagePath: /dev/termination-log
            name: minio
            livenessProbe:
              tcpSocket:
                port: 9000
              initialDelaySeconds: 30
              timeoutSeconds: 1
              periodSeconds: 5
              successThreshold: 1
              failureThreshold: 3
            env:
              - name: MINIO_ROOT_USER
                valueFrom:
                  secretKeyRef:
                    name: minio-secret
                    key: minio_root_user
              - name: MINIO_ROOT_PASSWORD
                valueFrom:
                  secretKeyRef:
                    name: minio-secret
                    key: minio_root_password
            ports:
              - containerPort: 9000
                protocol: TCP
              - containerPort: 9090
                protocol: TCP
            imagePullPolicy: IfNotPresent
            volumeMounts:
              - name: data
                mountPath: /data
                subPath: minio
            terminationMessagePolicy: File
            image: >-
              quay.io/minio/minio:RELEASE.2023-06-19T19-52-50Z
            args:
              - server
              - /data
              - --console-address
              - :9090
        restartPolicy: Always
        terminationGracePeriodSeconds: 30
        dnsPolicy: ClusterFirst
        securityContext: {}
        schedulerName: default-scheduler
    strategy:
      type: Recreate
    revisionHistoryLimit: 10
    progressDeadlineSeconds: 600
  ---
  kind: Service
  apiVersion: v1
  metadata:
    name: minio-service
  spec:
    ipFamilies:
      - IPv4
    ports:
      - name: api
        protocol: TCP
        port: 9000
        targetPort: 9000
      - name: ui
        protocol: TCP
        port: 9090
        targetPort: 9090
    internalTrafficPolicy: Cluster
    type: ClusterIP
    ipFamilyPolicy: SingleStack
    sessionAffinity: None
    selector:
      app: minio
  ---
  kind: Route
  apiVersion: route.openshift.io/v1
  metadata:
    name: minio-api
  spec:
    to:
      kind: Service
      name: minio-service
      weight: 100
    port:
      targetPort: api
    wildcardPolicy: None
    tls:
      termination: edge
      insecureEdgeTerminationPolicy: Redirect
  ---
  kind: Route
  apiVersion: route.openshift.io/v1
  metadata:
    name: minio-ui
  spec:
    to:
      kind: Service
      name: minio-service
      weight: 100
    port:
      targetPort: ui
    wildcardPolicy: None
    tls:
      termination: edge
      insecureEdgeTerminationPolicy: Redirect
  ```

1. By default, the size of the storage is 20 GB. (see line 11). Change it if you need to.
1. If you want to, edit lines 21-22 to change the default user/password.
1. Press Create.
1. You should see:

    ![alt_text](img/resources.created.png)

1. And there should now be a running minio pod:

    ![alt_text](img/running.pod.png)

1. As well as  **two** minio routes:

    ![alt_text](img/routes.png)

1. The `-api` route is for programmatic access to Minio
1. The `-ui` route is for browser-based access to Minio
1. Your Minio Object Store is now deployed, but we still need to create at least one bucket in it, to make it useful.

## Creating a bucket in Minio

### Log in to Minio

1. Locate the **minio-ui** Route, and open its location URL in a web browser:
1. When prompted, log in
    * if you kept the default values, then:
    * user: `minio`
    * pass: `minio123`

    ![alt_text](img/minio.login.png){style="width:400px"}

1. You should now be logged into your Minio instance.

### Create a bucket

1. Click on **Create a Bucket**

    ![alt_text](img/create.bucket.01.png)

1. Choose a name for your bucket (for example `mybucket`) and click **Create Bucket**:

    ![alt_text](img/create.bucket.02.png)

1. Repeat those steps to create as many buckets as you will need.

## Create a matching Data Connection for Minio

1. Back in RHODS, inside of your Data Science Project, Click on **Add data connection**:

    ![alt_text](img/add.connection.png)

1. Then, fill out the required field to match with your newly-deployed Minio Object Storage

    ![alt_text](img/connection.details.png)

1. You now have a Data Connection that maps to your **mybucket** bucket in your Minio Instance.
1. This data connection can be used, among other things
    * In your Workbenches
    * For your Model Serving
    * For your Pipeline Server Configuration

## Notes and FAQ

* As long as you are using the Route URLs, a Minio running in one namespace can be used by any other application, even running in another namespace, or even in another cluster altogether.

## Uninstall instructions:

This will completely remove Minio and all its content. Make sure you have a backup of the things your need before doing so!

1. Track down those objects created earlier:

    ![alt_text](img/resources.created.png)

1. Delete them all.
