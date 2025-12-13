# Quick Start Guide

Get the CloudNative SaaS Platform up and running in under 30 minutes.

## Prerequisites

Before you begin, ensure you have:

- ✅ AWS Account with appropriate permissions
- ✅ AWS CLI configured (`aws configure`)
- ✅ Terraform installed (v1.5+)
- ✅ kubectl installed
- ✅ Git installed
- ✅ GitHub account (for CI/CD)

See [03-Prerequisites.md](03-Prerequisites.md) for detailed setup instructions.

## Step 1: Clone Repositories

```bash
# Create a workspace directory
mkdir -p ~/saas-platform
cd ~/saas-platform

# Clone all repositories
git clone https://github.com/SaaSInfraLab/cloudnative-saas-eks.git
git clone https://github.com/SaaSInfraLab/Gitops-pipeline.git 
git clone https://github.com/SaaSInfraLab/Sample-saas-app.git
```

## Step 2: Configure Infrastructure

Edit the configuration files in `cloudnative-saas-eks/examples/dev-environment/config/`:

```bash
cd cloudnative-saas-eks/examples/dev-environment/config
```

**Edit `common.tfvars`:**
```hcl
aws_region = "us-east-1"
environment = "dev"
project_name = "my-saas-platform"
```

**Edit `infrastructure.tfvars`:**
```hcl
vpc_cidr = "10.0.0.0/16"
eks_cluster_version = "1.28"
node_instance_type = "t3.medium"
```

**Edit `tenants.tfvars`:**
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

## Step 3: Deploy Infrastructure

### 3.1 Deploy Core Infrastructure

```bash
cd cloudnative-saas-eks/examples/dev-environment/infrastructure

# Initialize Terraform
terraform init -backend-config=../config/infrastructure/backend-dev.tfbackend

# Review the plan
terraform plan -var-file=../config/infrastructure.tfvars -var-file=../config/common.tfvars

# Apply infrastructure
terraform apply -var-file=../config/infrastructure.tfvars -var-file=../config/common.tfvars
```

This creates:
- VPC and networking
- EKS cluster
- RDS PostgreSQL database
- IAM roles and policies
- Security groups

**Wait for completion** (~15-20 minutes)

### 3.2 Configure kubectl

```bash
# Get cluster name from Terraform output
CLUSTER_NAME=$(terraform output -raw cluster_name)
AWS_REGION=$(terraform output -raw aws_region)

# Configure kubectl
aws eks update-kubeconfig --name $CLUSTER_NAME --region $AWS_REGION

# Verify connection
kubectl get nodes
```

### 3.3 Deploy Tenants

```bash
cd ../tenants

# Initialize Terraform
terraform init -backend-config=../config/tenants/backend-dev.tfbackend

# Apply tenant configuration
terraform apply -var-file=../config/tenants.tfvars -var-file=../config/common.tfvars
```

This creates:
- Tenant namespaces
- Resource quotas
- Network policies
- RBAC configurations

## Step 4: Set Up GitOps (ArgoCD)

### 4.1 Install ArgoCD

```bash
cd ~/saas-platform/Gitops-pipeline

# Install ArgoCD
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Wait for ArgoCD to be ready
kubectl wait --for=condition=available --timeout=300s deployment/argocd-server -n argocd
```

### 4.2 Get ArgoCD Admin Password

```bash
# Get initial admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d && echo
```

### 4.3 Access ArgoCD UI

```bash
# Port forward to ArgoCD
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Open browser: `https://localhost:8080`
- Username: `admin`
- Password: (from step 4.2)

### 4.4 Bootstrap Applications

```bash
# Apply app-of-apps pattern
kubectl apply -f argocd/app-of-apps.yaml
```

This automatically creates:
- Sample SaaS application (platform tenant)
- Sample SaaS application (analytics tenant)
- Monitoring stack (Prometheus/Grafana)

## Step 5: Deploy Applications

### 5.1 Build and Push Container Images

The CI/CD pipeline will automatically build and push images when you push code. For manual build:

```bash
cd ~/saas-platform/Sample-saas-app

# Build backend
cd backend
docker build -t <your-ecr-repo>/backend:latest .
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <your-ecr-repo>
docker push <your-ecr-repo>/backend:latest

# Build frontend
cd ../frontend
docker build -t <your-ecr-repo>/frontend:latest .
docker push <your-ecr-repo>/frontend:latest
```

### 5.2 Update Image Tags in GitOps

Edit `Gitops-pipeline/apps/sample-saas-app/base/deployment.yaml` to use your image tags.

### 5.3 Sync Applications in ArgoCD

ArgoCD will automatically sync, or manually sync:
1. Go to ArgoCD UI
2. Click on each application
3. Click "Sync" button

## Step 6: Access Your Applications

### Get Application URLs

```bash
# Get frontend LoadBalancer URL
kubectl get svc -n platform frontend-service -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'

# Get backend LoadBalancer URL
kubectl get svc -n platform backend-service -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'
```

### Access Grafana

```bash
# Get Grafana LoadBalancer URL
kubectl get svc -n monitoring monitoring-stack-grafana -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'

# Get admin password
kubectl get secret -n monitoring monitoring-stack-grafana -o jsonpath='{.data.admin-password}' | base64 -d && echo
```

Access Grafana at: `http://<loadbalancer-url>`
- Username: `admin`
- Password: (from command above)

## Step 7: Verify Deployment

### Check Pod Status

```bash
# Check all pods
kubectl get pods -A

# Check specific namespace
kubectl get pods -n platform
kubectl get pods -n analytics
kubectl get pods -n monitoring
```

### Check Services

```bash
# List all services
kubectl get svc -A

# Check LoadBalancers
kubectl get svc -A | grep LoadBalancer
```

### Test Application

```bash
# Get frontend URL
FRONTEND_URL=$(kubectl get svc -n platform frontend-service -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')

# Test frontend
curl http://$FRONTEND_URL

# Test backend API
BACKEND_URL=$(kubectl get svc -n platform backend-service -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
curl http://$BACKEND_URL/api/health
```

## Troubleshooting

### Common Issues

**Issue**: Terraform apply fails
- **Solution**: Check AWS credentials and permissions
- **See**: [09-Troubleshooting.md](09-Troubleshooting.md)

**Issue**: Pods not starting
- **Solution**: Check pod logs and events
- **See**: [09-Troubleshooting.md](09-Troubleshooting.md)

**Issue**: ArgoCD sync failing
- **Solution**: Check ArgoCD application status and logs
- **See**: [09-Troubleshooting.md](09-Troubleshooting.md)

## Next Steps

1. **Read [04-Architecture.md](04-Architecture.md)** - Understand the architecture
2. **Review [06-Multi-Tenancy.md](06-Multi-Tenancy.md)** - Learn about multi-tenancy
3. **Explore [11-Monitoring.md](11-Monitoring.md)** - Set up monitoring dashboards
4. **Check [18-Tenant-Provisioning.md](18-Tenant-Provisioning.md)** - Add more tenants

## Quick Reference Commands

```bash
# Get cluster info
kubectl cluster-info

# Get all resources
kubectl get all -A

# Check ArgoCD applications
kubectl get applications -n argocd

# View pod logs
kubectl logs -n <namespace> <pod-name>

# Describe resource
kubectl describe <resource-type> -n <namespace> <resource-name>
```

---

**Need help?** Check [09-Troubleshooting.md](09-Troubleshooting.md) or review [04-Architecture.md](04-Architecture.md)

