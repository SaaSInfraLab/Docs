# Repository Structure

Understanding the codebase organization and how repositories work together.

## Repository Overview

The platform consists of four main repositories, each serving a specific purpose:

```
┌─────────────────────────────────────────────────────────┐
│  cloudnative-saas-eks                                   │
│  Single Source of Truth for Infrastructure Config        │
└─────────────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────────────┐
│  Terraform-modules                                       │
│  Reusable Infrastructure Modules                         │
└─────────────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────────────┐
│  Gitops-pipeline                                         │
│  GitOps Automation & Application Manifests              │
└─────────────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────────────┐
│  Sample-saas-app                                         │
│  Application Code & CI/CD                                │
└─────────────────────────────────────────────────────────┘
```

## Repository Details

### 1. cloudnative-saas-eks

**Purpose**: Single source of truth for all infrastructure configuration

**Structure**:
```
cloudnative-saas-eks/
├── examples/
│   └── dev-environment/
│       ├── config/                    # ⭐ ALL CONFIGURATION FILES
│       │   ├── common.tfvars          # Shared values
│       │   ├── infrastructure.tfvars  # Infrastructure config
│       │   ├── tenants.tfvars         # Tenant config
│       │   └── */backend-dev.tfbackend # Backend configs
│       ├── infrastructure/            # Infrastructure Terraform
│       └── tenants/                   # Tenant Terraform
└── docs/                              # Documentation
```

**Key Files**:
- `config/*.tfvars` - Configuration files (edit these)
- `infrastructure/main.tf` - Infrastructure Terraform code
- `tenants/main.tf` - Tenant provisioning code

### 2. Gitops-pipeline

**Purpose**: GitOps automation layer and Kubernetes manifests

**Structure**:
```
Gitops-pipeline/
├── argocd/
│   ├── applications/                   # ArgoCD app definitions
│   └── app-of-apps.yaml               # Bootstrap pattern
├── apps/
│   └── sample-saas-app/               # Application manifests
│       ├── base/                      # Base resources
│       └── overlays/                  # Tenant-specific configs
├── scripts/                           # Deployment scripts
└── .github/workflows/                 # CI/CD workflows
```

**Key Files**:
- `argocd/applications/*.yaml` - ArgoCD application definitions
- `apps/*/base/*.yaml` - Base Kubernetes manifests
- `apps/*/overlays/*/kustomization.yaml` - Tenant overlays

### 3. Sample-saas-app

**Purpose**: Sample multi-tenant application code

**Structure**:
```
Sample-saas-app/
├── backend/                           # Node.js backend
│   ├── src/                           # Source code
│   ├── Dockerfile                     # Container image
│   └── package.json                  # Dependencies
├── frontend/                          # React frontend
│   ├── src/                           # Source code
│   ├── Dockerfile                     # Container image
│   └── package.json                  # Dependencies
├── database/
│   └── migrations/                    # SQL migrations
└── .github/workflows/                 # CI/CD workflows
```

**Key Files**:
- `backend/src/server.js` - Backend entry point
- `frontend/src/App.jsx` - Frontend entry point
- `database/migrations/*.sql` - Database migrations

### 4. Terraform-modules

**Purpose**: Reusable Terraform modules

**Structure**:
```
Terraform-modules/
└── modules/
    ├── vpc/                           # VPC module
    ├── eks/                           # EKS module
    ├── rds/                           # RDS module
    ├── iam/                           # IAM module
    ├── multi-tenancy/                 # Multi-tenancy module
    └── ...                            # Other modules
```

## File Organization Patterns

### Configuration Files

All configuration is centralized in `cloudnative-saas-eks/examples/dev-environment/config/`:

- **common.tfvars** - Shared values (AWS region, project name, tags)
- **infrastructure.tfvars** - Infrastructure-specific config
- **tenants.tfvars** - Tenant-specific config

### Kubernetes Manifests

Organized using Kustomize:

- **base/** - Base resources (deployments, services, configmaps)
- **overlays/** - Environment/tenant-specific patches

### Application Code

Standard application structure:

- **src/** - Source code
- **Dockerfile** - Container image definition
- **package.json** - Dependencies

## Workflow

1. **Edit configs** in `cloudnative-saas-eks/config/`
2. **Terraform applies** using modules from `Terraform-modules`
3. **GitOps watches** `Gitops-pipeline` for changes
4. **Applications deploy** from `Sample-saas-app` via CI/CD

## Next Steps

- **Read [04-Architecture.md](04-Architecture.md)** - Understand system design
- **Review [07-Deployment-Guide.md](07-Deployment-Guide.md)** - Deployment process
- **Check [08-CI-CD-Pipeline.md](08-CI-CD-Pipeline.md)** - CI/CD workflow

---

**Want to understand the architecture?** → [04-Architecture.md](04-Architecture.md)

