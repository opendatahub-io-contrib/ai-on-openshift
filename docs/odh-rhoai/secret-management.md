
# Managing Secrets in an AI Platform

## Introduction

At Red Hat, we provide an internal OpenShift AI cluster that serves as a unified platform for experimentation, prototyping, and scalable deployment of AI solutions. Designed to support a potential user base of over 19,000 associates, the platform offers a range of capabilities—including granular role-based access control (RBAC), GPU auto-scaling for efficient resource management, hosting models from the Granite, Llama, Mistral, and DeepSeek families, as well as specialized models for vision, embeddings, and safety filtering. It also supports demo products like [Models as a Service](https://ai-on-openshift.io/generative-ai/ai-for-everyone/) and the [Chat with Your Documentation](https://ai-on-openshift.io/demos/llm-chat-doc/llm-chat-doc/) RAG implementation. This variety enables teams across Red Hat to explore diverse AI workloads and build solutions tailored to their specific use cases by using OpenShift AI.

Like any production-grade platform, supporting these capabilities at scale requires more than just compute resources. It also requires solid engineering practices, including the secure management of sensitive configuration data—such as cloud credentials and authorization tokens used in platform setup.

Since day one, we’ve managed the cluster lifecycle using [GitOps](https://ai-on-openshift.io/odh-rhoai/gitops/). However, the presence of these sensitive values meant that we initially had to keep our configuration repository private. While this worked operationally, it limited our ability to share the implementation more broadly with others.

To address this, we adopted a secret management solution. This enabled us to decouple secrets from the GitOps-managed resources and paved the way for securely opening up parts of the platform’s configuration. You can now explore the repository [here](https://github.com/rh-aiservices-bu/rhoaibu-cluster) to see how we run this AI platform in a secure and scalable way.

In this post, we’ll walk through the high level structure of the repository we’ve opened up, what you can find inside, and how it reflects the way we run and scale OpenShift AI internally. We’ll also take a closer look at the secret management approach we adopted; why we chose External Secrets Operator (ESO), how it fits into our GitOps workflow, and the lessons we learned along the way.

## What's In the Box?

The [GitHub repository](https://github.com/rh-aiservices-bu/rhoaibu-cluster) captures how we run production-grade AI infrastructure at scale—offering reusable patterns, modular design, and automation strategies. Here's a quick overview of what you’ll find inside:

- Declarative GitOps configurations for managing AI/ML infrastructure

- Customizations for OpenShift AI, including workbenches and model serving

- GPU sharing, time-slicing, and autoscaling across different types of GPUs

- Model deployments that power Model-as-a-Service platform
  
- Environment overlays and promotion workflows for dev/prod separation

- Security, observability, and cost-optimization practices built in

Sharing this repo publicly meant revisiting how we manage sensitive values. Because while transparency is great, credentials shouldn’t live in version control as plain text. They should be managed as code, but in a secure and auditable way. To achieve this, we adopted a secret management tool that integrates seamlessly with our GitOps flow.

## What is External Secrets Operator?

[External Secrets Operator](https://external-secrets.io/latest/) is a Kubernetes operator that integrates external secret management systems like AWS Secrets Manager, HashiCorp Vault, GCP Secret Manager, and Azure Key Vault with Kubernetes. It allows you to securely inject secrets from these external systems into your Kubernetes clusters. We use it to pull secrets from AWS Secrets Manager into our OpenShift AI clusters securely and automatically.


## Why We Chose External Secrets Operator?

Several factors influenced our decision to use External Secrets Operator:

1. **Native Kubernetes Integration**: ESO follows the Kubernetes operator pattern, making it easy to integrate with our OpenShift clusters.

2. **AWS Integration**: With our infrastructure running on AWS, ESO's native support for AWS Secrets Manager was a good fit.

3. **GitOps Friendly**: ESO works well with our GitOps workflow using Argo CD, allowing us to manage secret references declaratively in Git without exposing sensitive data.

## Our Implementation

First, let's look at a concrete example of how we use External Secrets in our cluster:

**Database Credentials for Model Registry**
[This example](https://github.com/rh-aiservices-bu/rhoaibu-cluster/blob/dev/components/instances/rhoai-instance/components/model-registry/mysql-secret.yaml) shows how we fetch the database credentials for our Model Registry that is stored in AWS Secrets Manager, demonstrating how one ExternalSecret can map multiple properties from a secret store.

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
 name: registry-db-secrets
 namespace: rhoai-model-registries
spec:
 refreshInterval: 1h
 secretStoreRef:
   name: rhoaibu-external-store
   kind: ClusterSecretStore
 target:
   name: registry-db-secrets
   creationPolicy: Owner
 data:
   - secretKey: MYSQL_ROOT_PASSWORD
     remoteRef:
       key: model-registries-db-credentials
       property: database-password
   - secretKey: MYSQL_USER_NAME
     remoteRef:
       key: model-registries-db-credentials
       property: database-user
```

### Architecture

We implemented External Secrets Operator using a GitOps approach with the following components:


1. **Operator Installation**: We use the Red Hat Community of Practice (CoP)'s GitOps Catalog to deploy the operator:

    ```yaml
    apiVersion: kustomize.config.k8s.io/v1beta1
    kind: Kustomization
    resources:
    - https://github.com/redhat-cop/gitops-catalog/external-secrets-operator/operator/overlays/stable
    ```

2. **AWS Secrets Manager Integration**: We configured a `ClusterSecretStore` that connects to AWS Secrets Manager:
   
    ```yaml
    apiVersion: external-secrets.io/v1beta1
    kind: ClusterSecretStore
    metadata:
      name: rhoaibu-external-store
    spec:
      provider:
        aws:
          service: SecretsManager
          region: us-west-2
    ```


3. **Namespace Scoping**: For security, we explicitly define which namespaces can access the external secrets:

    ```yaml
    conditions:
    - namespaces:
        - "cert-manager"
        - "openshift-config"
        - "rhoai-model-registries"
    ```


## Benefits We've Seen

1. **Improved Security**: Secrets are stored securely in AWS Secrets Manager, separate from our application code.
2. **Simplified Management**: One central place (AWS Secrets Manager) to manage all our secrets.
3. **GitOps Compatible**: WWe can declaratively manage secret references in Git while keeping actual secrets securely stored in AWS.
4. **Automated Syncing**: Secrets are automatically synchronized between AWS and our clusters.


## Alternatives We Considered

1. **Sealed Secrets**: While powerful, it didn't offer the same level of integration with external secret managers.
2. **Vault Operator**: HashiCorp Vault was more complex to set up and maintain compared to using AWS Secrets Manager.
3. **Native Kubernetes Secrets**: Since they are only base64 encoding, they lack the security and management features we needed.


## Challenges and Solutions

1. **Initial Setup**: Required detailed IAM role configurations to ensure secure access to secrets.
2. **Multi-Region Support**: Solved by using environment-specific patches for different AWS regions.

## Good Practices We Follow

1. **Namespace Isolation**: Strictly control which namespaces can access secrets
2. **Minimal Access**: Use specific IAM roles with least privilege
3. **Version Control**: Maintain secret configurations in Git while keeping sensitive data in AWS
4. **Environment Separation**: Different configurations for dev and prod environments

## Conclusion

External Secrets Operator has proven to be a robust solution for our secret management needs. It provides the right balance of security, ease of use, and integration capabilities. Most importantly, it allowed us to open source our entire cluster setup, from installation to Day 2 operations, while keeping sensitive data secure in AWS Secrets Manager. This separation of configuration enables us to share our implementation publicly, allowing others to learn from and build upon our work while maintaining the security of our credentials and sensitive data.
