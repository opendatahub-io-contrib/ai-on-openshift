# Data Science Pipeline

## What is it?

OpenShift AI allows building of machine line workflows with a data science pipeline. From OpenShift AI version 2.9, data science pipelines are based on KubeFlow Pipelines (KFP) version 2.0.

## What is Kubeflow Pipelines?
Kubeflow Pipelines (KFP) is a platform for building and deploying portable and scalable machine learning (ML) workflows using Docker containers.

With KFP you can author components and pipelines using the KFP Python SDK, compile pipelines to an intermediate representation YAML, and submit the pipeline to run on a KFP-conformant backend.

The open source KFP backend is available as a core component of Kubeflow or as a standalone installation. Follow the installation instructions and Hello World Pipeline example to quickly get started with KFP.

## Example 

## Architectural Diagram

![dsp-arch](img/rhoai-dsp.jpg)

The demo uses the following components:

| Component | Descrioption|
|---|---|
| Gitea | To store pipeline source code
| Model Registry | To store model metadata
| OpenShift Pipelines | Using Tekton to build the pipeline
| Data Science Pipeline | To run the pipeline using KFP
| Minio | S3 bucket to store the model
| KServe | To serve the model

## Prerequisite

You will need OpenShift 2.15 installed with ModelRegistry set to `Managed`. In 2.15, the model registry feature is currently in Tech Preview.

### Running the Example

The sample code is available [here](https://github.com/tsailiming/openshift-ai-dsp).




