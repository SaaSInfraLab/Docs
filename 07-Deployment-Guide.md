# Complete Deployment Guide

Step-by-step guide to deploying the entire CloudNative SaaS Platform.

## Deployment Overview

The deployment process consists of 5 main phases:

```
Phase 1: Infrastructure (Terraform)
    ↓
Phase 2: Tenants (Terraform)
    ↓
Phase 3: GitOps Setup (ArgoCD)
    ↓
Phase 4: Database Initialization
    ↓
Phase 5: Application Deployment
```

## Phase 1: Infrastructure Deployment

### Prerequisites

- AWS CLI configured
- Terraform installed
- S3 bucket for Terraform state

### Step 1: Configure Infrastructure

Edit `cloudnative-saas-eks/examples/dev-environment/config/infrastructure.tfvars`:

```hcl
vpc_cidr = "10.0.0.0/16"
eks_cluster_version = "1.28"
node_instance_type = "t3.medium"
node_desired_size = 2
```

### Step 2: Deploy Infrastructure

```bash
cd cloudnative-saas-eks/examples/dev-environment/infrastructure

# Initialize
terraform init -backend-config=../config/infrastructure/backend-dev.tfbackend

# Plan
terraform plan -var-file=../config/infrastructure.tfvars -var-file=../config/common.tfvars

# Apply
terraform apply -var-file=../config/infrastructure.tfvars -var-file=../config/common.tfvars
```

**Duration**: 15-20 minutes

**Creates**:
- VPC and networking
- EKS cluster
- RDS PostgreSQL
- ECR repositories
- IAM roles
- Security groups
- AWS Secrets Manager secrets

## Phase 2: Tenant Deployment

### Step 1: Configure Tenants

Edit `cloudnative-saas-eks/examples/dev-environment/config/tenants.tfvars`:

```hcl
tenants = {
  platform = {
    namespace = "platform"
    cpu_limit = "4"
    memory_limit = "8Gi"
  }
  analytics = {
    namespace = "analytics"
    cpu_limit = "2"
    memory_limit = "4Gi"
  }
}
```

### Step 2: Deploy Tenants

```bash
cd ../tenants

# Initialize
terraform init -backend-config=../config/tenants/backend-dev.tfbackend

# Apply
terraform apply -var-file=../config/tenants.tfvars -var-file=../config/common.tfvars
```

**Duration**: 2-5 minutes

**Creates**:
- Tenant namespaces
- Resource quotas
- Network policies
- Kubernetes secrets (from AWS Secrets Manager)
- ConfigMaps

## Phase 3: GitOps Setup

### Step 1: Configure kubectl

```bash
CLUSTER_NAME=$(cd ../infrastructure && terraform output -raw cluster_name)
AWS_REGION=$(cd ../infrastructure && terraform output -raw aws_region)

aws eks update-kubeconfig --name $CLUSTER_NAME --region $AWS_REGION
```

### Step 2: Install ArgoCD

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Wait for ArgoCD
kubectl wait --for=condition=available --timeout=300s deployment/argocd-server -n argocd
```

### Step 3: Access ArgoCD

```bash
# Get admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d && echo

# Port forward
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Access: `https://localhost:8080` (username: `admin`)

### Step 4: Bootstrap Applications

```bash
cd ~/saas-platform/Gitops-pipeline
kubectl apply -f argocd/app-of-apps.yaml
```

## Phase 4: Database Initialization

Database initialization happens automatically when ArgoCD syncs applications.

### Verify Database Init

```bash
# Check init jobs
kubectl get jobs -n platform
kubectl get jobs -n analytics

# Check logs
kubectl logs job/init-rds-database -n platform
```

## Phase 5: Application Deployment

### Option 1: Automatic (CI/CD)

Push code to trigger CI/CD:

```bash
cd ~/saas-platform/Sample-saas-app
git add .
git commit -m "Deploy application"
git push origin main
```

### Option 2: Manual

Build and push images manually, then update GitOps repository.

## Verification

### Check All Resources

```bash
# Pods
kubectl get pods -A

# Services
kubectl get svc -A

# ArgoCD applications
kubectl get applications -n argocd
```

### Access Applications

```bash
# Get LoadBalancer URLs
kubectl get svc -n platform frontend-service -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'
kubectl get svc -n platform backend-service -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'
```

## Next Steps

- **Read [08-CI-CD-Pipeline.md](08-CI-CD-Pipeline.md)** - CI/CD workflow and database initialization
- **Review [04-Architecture.md](04-Architecture.md)** - System architecture
- **Check [09-Troubleshooting.md](09-Troubleshooting.md)** - Common issues

---

**Need help?** Check [09-Troubleshooting.md](09-Troubleshooting.md)

