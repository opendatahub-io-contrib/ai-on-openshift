# Fully GitOpsified implementation of an RHOAI platform

## What is the RHOAI BU Cluster Repository?

The **[RHOAI BU Cluster GitOps Repository](https://github.com/rh-aiservices-bu/rhoaibu-cluster)** manages the **OpenShift AI BU cluster** and is a comprehensive GitOps implementation for managing Red Hat OpenShift AI infrastructure using declarative configuration management.

**What it manages:**

- Two OpenShift clusters: development (`rhoaibu-cluster-dev`) and production (`rhoaibu-cluster-prod`)
- Complete AI/ML platform infrastructure using GitOps practices  
- Models as a Service (MaaS) platform with 15+ AI models

**Purpose:**

- Working example of GitOps for AI infrastructure
- Reference architecture for organizations implementing AI/ML platforms

!!! warning "Internal Service Notice"
    This is an internal development and testing service. No Service Level Agreements (SLAs) or Service Level Objectives (SLOs) are provided or guaranteed. This service is not intended for production use cases or mission-critical applications.

Unlike basic GitOps examples, this repository manages:

- **Complete cluster lifecycle** from development to production workloads
- **Models as a Service (MaaS)** platform with 15+ AI models (Granite, Llama, Mistral, Phi-4), 3Scale API gateway, Red Hat SSO authentication, self-service portal, and usage analytics for internal development and testing
- **Multi-environment support** with dev and prod cluster configurations  
- **Advanced AI-specific components** including GPU autoscaling, declarative model serving, and custom workbenches
- **Advanced features** like OAuth integration, RBAC, and certificate management

!!! info
    This guide provides an overview of the **[RHOAI BU Cluster GitOps Repository](https://github.com/rh-aiservices-bu/rhoaibu-cluster)**, an implementation of GitOps for managing the RHOAIBU Cluster infrastructure at scale including OpenShift AI and MaaS.
    
    For foundational OpenShift AI & GitOps concepts and object definitions, please refer to our **[Managing RHOAI with GitOps](gitops.md)** guide first.

## Why GitOps for AI Infrastructure?

GitOps provides unique advantages for AI/ML workloads that traditional infrastructure management approaches struggle to deliver. Rather than reinventing the wheel, this implementation builds upon the proven **[GitOps Catalog](https://github.com/redhat-cop/gitops-catalog)** from Red Hat Community of Practice, providing battle-tested GitOps patterns and components as the foundation for our AI-specific infrastructure.

**🔄 Infrastructure Reproducibility**: AI experiments require consistent environments. GitOps ensures your development, and production clusters are identical, eliminating "works on my machine" issues.

**📊 GPU Resource Management**: Automated scaling and configuration of expensive GPU resources based on declarative policies, reducing costs while ensuring availability.

**🚀 Faster Iteration**: Version-controlled infrastructure changes enable rapid experimentation with different configurations, operators, and serving runtimes.

**🛡️ Compliance & Auditing**: Complete audit trail of all infrastructure changes, critical for regulated industries deploying AI models.

## Repository Architecture & Hierarchy

The repository follows a layered architecture that separates concerns while maintaining flexibility:

```
rhoaibu-cluster/
├── bootstrap/          # Initial cluster setup and GitOps installation
├── clusters/           # Environment-specific configurations
│   ├── base/          # Shared cluster resources
│   └── overlays/      # Dev/prod environment customizations
├── components/        # Modular GitOps components
│   ├── argocd/       # ArgoCD projects and applications
│   ├── configs/      # Cluster-wide configurations
│   ├── instances/    # Operator instance configurations
│   ├── operators/    # Core Red Hat operators
│   └── operators-extra/ # Community and third-party operators
├── demos/            # Demo applications and examples
└── docs/             # Documentation and development guides
```

### Core Components Deep Dive

#### 1. Bootstrap Layer (`bootstrap/`)

The bootstrap layer handles initial cluster setup and GitOps installation. Key features include:

- **Cluster Installation**: OpenShift cluster deployment with GPU machinesets
- **GitOps Bootstrap**: OpenShift GitOps operator installation and initial configuration
- **Authentication Setup**: Google OAuth integration for internal access
- **Certificate Management**: Let's Encrypt certificates with automatic renewal

📖 **Reference**: [Bootstrap Documentation](https://github.com/rh-aiservices-bu/rhoaibu-cluster/blob/main/bootstrap/README.md)

#### 2. Operators Management (`components/operators/`)

**Core Red Hat Operators** managed through GitOps:

- Red Hat OpenShift AI (RHOAI)
- NVIDIA GPU Operator
- OpenShift Data Foundation
- OpenShift Cert Manager
- OpenShift Service Mesh
- OpenShift Serverless

📖 **Reference**: [Core Operators Documentation](https://github.com/rh-aiservices-bu/rhoaibu-cluster/blob/main/components/operators)

**Community & Third-Party Operators** (`components/operators-extra/`):

📖 **Reference**: [Operators Documentation](https://github.com/rh-aiservices-bu/rhoaibu-cluster/tree/main/docs#community-and-third-party-operators)

#### 3. Instance Configurations (`components/instances/`)

Configurations for each operator:

- **RHOAI Instance**: Complete DataScienceCluster configuration with custom accelerator profiles, workbenches, and dashboard settings
- **GPU Management**: NVIDIA operator policies optimized for AI workloads
- **Storage**: OpenShift Data Foundation instance for proving storage capabilities (including RWX) for OpenShift AI workloads.
- **Certificates**: Automated TLS certificate management for model serving endpoints

📖 **Reference**: [Operator Instances Documentation](https://github.com/rh-aiservices-bu/rhoaibu-cluster/blob/main/components/instances/rhoai-instance)

#### 4. Cluster Configurations (`components/configs/`)

Cluster-wide settings:

- **Authentication**: OAuth providers and RBAC configurations
- **Autoscaling**: GPU-optimized cluster autoscaler with support for several GPUs
- **Console Customization**: OpenShift and RHOAI console with AI-focused navigation
- **Namespace Management**: Project request templates and resource quotas

📖 **Reference**: [Cluster Configurations Documentation](https://github.com/rh-aiservices-bu/rhoaibu-cluster/blob/main/components/configs)

!!! note "Comprehensive Configuration Details"
    For detailed Kubernetes object definitions and advanced configuration options for model serving, data connections, accelerator profiles, and InferenceService objects, see our comprehensive **[Managing RHOAI with GitOps](gitops.md)** guide.

## Key GitOps Workflows

### Development and Testing Workflow

The repository supports a complete development lifecycle:

1. **Fork and Branch**: Developers create feature branches for infrastructure changes
2. **Local Testing**: Kustomize allows local validation before deployment
3. **Dev Environment**: Changes are tested in the development cluster first
4. **Production Promotion**: Validated changes are promoted to production via GitOps

📖 **Reference**: [Development Workflow](https://github.com/rh-aiservices-bu/rhoaibu-cluster/blob/main/docs/develop-and-test-changes.md)

### GPU Autoscaling for AI Workloads

GPU autoscaling is critical for AI workloads since GPUs are expensive resources that need to scale based on demand. The cluster automatically provisions GPU nodes from separate pools for different use cases (Tesla T4 for cost-effective training/inference, NVIDIA A10G for high-memory workloads and model serving), with configurations for both shared and private access patterns.

📖 **Reference**: [Autoscaling Configuration](https://github.com/rh-aiservices-bu/rhoaibu-cluster/blob/main/components/configs/autoscaling/README.md)

### Accelerator Profiles

OpenShift AI Accelerator Profiles configurations for optimal GPU utilization:

```yaml
# Example accelerator profiles
- NVIDIATesla T4: 16GB VRAM, ideal for inference and small model training
- NVIDIA A10G: 24GB VRAM, optimized for large model training
- NVIDIA L40: 24GB VRAM, optimized for large model training
- NVIDIA L40s: 48GB VRAM, designed for multi-modal AI workloads
- NVIDIA H100: 80GB VRAM, cutting-edge performance for large-scale training
```

📖 **Reference**: [Accelerator Profiles Configuration](https://github.com/rh-aiservices-bu/rhoaibu-cluster/tree/main/components/instances/rhoai-instance/components/accelerator-profiles)

## AI-Specific Components

### Models as a Service (MaaS)

The RHOAI BU Cluster serves as internal infrastructure hosting a complete **Models as a Service (MaaS)** platform. MaaS runs entirely on top of this GitOps-managed cluster infrastructure, providing AI model serving capabilities for development and testing purposes internally to all Red Hat Employees.

The MaaS implementation leverages the **[Models as a Service repository](https://github.com/rh-aiservices-bu/models-aas)**, which demonstrates how to set up 3Scale and Red Hat SSO in front of models served by OpenShift AI.

!!! warning "Internal Service Notice"
    This is an internal development and testing service. No Service Level Agreements (SLAs) or Service Level Objectives (SLOs) are provided or guaranteed. This service is not intended for production use cases or mission-critical applications.

#### Declarative Model Serving as Code

```yaml
# MaaS Configuration Structure
components/configs/maas/
├── model-serving/base/              # Model deployment configurations
│   ├── granite-3.3-8b-instruct.yaml
│   ├── llama-4-scout.yaml
│   ├── llama-3.2-3b.yaml
│   ├── mistral-small-24b.yaml
│   ├── phi-4.yaml
│   ├── nomic-embed-text-v1-5.yaml
│   ├── docling.yaml
│   ├── sdxl-custom-runtime.yaml
│   └── serving-runtimes/
└── 3scale-config/base/              # API management configurations
    ├── granite-3-3-8b-instruct/
    ├── llama-4-scout/
    ├── llama-3-2-3b/
    ├── mistral-small-24b/
    ├── phi-4/
    └── [per-model configs for authentication, rate limiting, documentation]
```

The MaaS platform organizes configurations for 15+ models with dedicated GitOps configurations for both OpenShift AI model serving and 3Scale API management.

**Available Model Types**:

- **Large Language Models**: Llama-4-Scout, Granite 3.3 8B, Llama 3.2 3B, Mistral Small 24B, Phi-4
- **Embedding Models**: Nomic Embed Text v1.5 for semantic search and RAG applications
- **Vision Models**: Granite Vision 3.2 2B, Qwen2.5 VL 7B for multimodal AI
- **Specialized Models**: Document processing (Docling), safety checking, image generation (SDXL)
- **Lightweight Models**: TinyLlama 1.1B (running on CPU)

📖 **Reference**: [MaaS Configurations](https://github.com/rh-aiservices-bu/rhoaibu-cluster/blob/main/components/configs/maas/)

This GitOps approach ensures:

- **Consistent Deployments**: Identical model configurations across dev/prod environments
- **Version Control**: Full audit trail of model deployment changes  
- **Easy Rollbacks**: Quick reversion to previous model versions
- **Automated Scaling**: GPU autoscaling based on model demand

### Custom AI Workbenches

Pre-configured Jupyter environments optimized for specific AI tasks:

- **PyTorch 2.1.4** with CUDA support for deep learning
- **Elyra with R** for data science workflows  
- **AnythingLLM** for conversational AI development
- **Docling** for document processing pipelines
- **Custom tools** for specialized AI workflows

📖 **Reference**: [Custom Workbenches Configuration](https://github.com/rh-aiservices-bu/rhoaibu-cluster/blob/main/components/instances/rhoai-instance)

While this implementation serves as a practical reference architecture, organizations can extend it with additional features based on their specific requirements.

!!! tip "Next Steps"
    - Review our foundational **[GitOps guide](gitops.md)** for object-level details
    - Explore the **[RHOAI BU Cluster Repository](https://github.com/rh-aiservices-bu/rhoaibu-cluster)** for complete implementation examples
    - Check out the **[AI on OpenShift examples](https://github.com/rh-aiservices-bu/llm-on-openshift)** for application-level patterns

This repository demonstrates that GitOps isn't just a deployment strategy—it's a comprehensive approach to managing the complex, rapidly-evolving infrastructure requirements of modern AI platforms.
