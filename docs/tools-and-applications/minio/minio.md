# Minio

## What is it?

[Minio](https://min.io/) is a high-performance, S3 compatible object store. It can be deployed on a wide variety of platforms, and it comes in multiple [flavors](https://min.io/pricing).

## Why this guide?

This guide is a very quick way of deploying the community version of Minio in order to quickly setup a fully standalone Object Store, in an OpenShift Cluster, which can then be used for various prototyping tasks.

Note that nothing in this guide should be used in production-grade environments.

## Pre-requisites

* Access to an OpenShift cluster
* Namespace-level admin permissions, or permission to create your own project
* 5 minutes

## Deploying Minio on OpenShift

### Create a Data Science Project (Optional)

If you already have your own Data Science Project, or OpenShift project, you can skip this step.

* If your cluster already has [Red Hat OpenShift Data Science](https://developers.redhat.com/products/red-hat-openshift-data-science/overview) installed, you can use the Dashboard Web Interface to create a Data Science project.
* Simply navigate to **Data Science Projects**
* And click **Create Project**

### Deploy Minio in an OpenShift project

### Log in to Minio and create a Bucket



