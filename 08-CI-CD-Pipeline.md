# CI/CD Pipeline Guide

Complete guide to understanding and setting up the CI/CD pipeline for the CloudNative SaaS Platform.

## Overview

The CI/CD pipeline automates the entire software delivery process from code commit to production deployment:

```
Developer pushes code
    ↓
GitHub Actions (CI) - Tests & Build
    ↓
Build Docker images
    ↓
Push to ECR
    ↓
Update GitOps repository
    ↓
ArgoCD (GitOps) - Deploy to Kubernetes
    ↓
Application running in EKS
```

## Pipeline Architecture

### Two-Stage Pipeline

```
┌─────────────────────────────────────────────────────────┐
│  Stage 1: Continuous Integration (CI)                   │
│  ┌───────────────────────────────────────────────────┐  │
│  │  GitHub Actions Workflow                          │  │
│  │  - Trigger: Push to main/develop                  │  │
│  │  - Run tests                                      │  │
│  │  - Build Docker images                            │  │
│  │  - Push to ECR                                    │  │
│  │  - Update GitOps repo with new image tags         │  │
│  └───────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────────────┐
│  Stage 2: Continuous Deployment (CD) via GitOps         │
│  ┌───────────────────────────────────────────────────┐  │
│  │  ArgoCD                                           │  │
│  │  - Watches GitOps repository                      │  │
│  │  - Detects image tag changes                      │  │
│  │  - Syncs to Kubernetes cluster                    │  │
│  │  - Deploys new application version                │  │
│  └───────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

## Initial Setup

### Step 1: Configure AWS OIDC Provider

**One-time setup** to allow GitHub Actions to access AWS resources:

1. **Go to AWS IAM Console** → Identity providers
2. **Add provider** (if not exists):
   - Provider type: **OpenID Connect**
   - Provider URL: `https://token.actions.githubusercontent.com`
   - Audience: `sts.amazonaws.com`
3. **Click "Add provider"**

### Step 2: Create IAM Role for GitHub Actions

1. **IAM Console** → Roles → Create role
2. **Select**: Web identity
3. **Identity provider**: `token.actions.githubusercontent.com`
4. **Audience**: `sts.amazonaws.com`
5. **Add condition**:
   ```
   Key: token.actions.githubusercontent.com:sub
   Operator: StringLike
   Value: repo:YOUR_ORG/Sample-saas-app:*
   ```
6. **Attach policies**:
   - **ECR Policy** (see below)
   - **EKS Policy** (see below)
7. **Role name**: `github-actions-ecr-eks-role`
8. **Copy Role ARN** (you'll need this)

**ECR Policy:**
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": [
      "ecr:GetAuthorizationToken",
      "ecr:BatchCheckLayerAvailability",
      "ecr:GetDownloadUrlForLayer",
      "ecr:BatchGetImage",
      "ecr:PutImage",
      "ecr:InitiateLayerUpload",
      "ecr:UploadLayerPart",
      "ecr:CompleteLayerUpload"
    ],
    "Resource": "*"
  }]
}
```

**EKS Policy:**
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": [
      "eks:DescribeCluster",
      "eks:ListClusters"
    ],
    "Resource": "*"
  }]
}
```

### Step 3: Get ECR Repository Names

```bash
cd cloudnative-saas-eks/examples/dev-environment/infrastructure
terraform output ecr_backend_repository_name
terraform output ecr_frontend_repository_name
```

**Note the repository names** - you'll add them as GitHub secrets.

### Step 4: Create GitHub Personal Access Token

1. **GitHub** → Settings → Developer settings → Personal access tokens → Tokens (classic)
2. **Generate new token (classic)**
3. **Name**: `GitOps Pipeline Access`
4. **Expiration**: 90 days (or custom)
5. **Scopes**: ✅ `repo` (Full control of private repositories)
6. **Generate token**
7. **Copy token immediately** (you won't see it again!)

### Step 5: Add GitHub Secrets

In your `Sample-saas-app` repository:

**Settings** → **Secrets and variables** → **Actions** → **New repository secret**

Add these secrets:

| Secret Name | Value | How to Get |
|------------|-------|------------|
| `AWS_ROLE_ARN` | `arn:aws:iam::ACCOUNT_ID:role/github-actions-ecr-eks-role` | From Step 2 |
| `AWS_REGION` | `us-east-1` | Your AWS region |
| `ECR_BACKEND_REPO` | `saas-infra-lab-dev-backend` | From Step 3 |
| `ECR_FRONTEND_REPO` | `saas-infra-lab-dev-frontend` | From Step 3 |
| `GITOPS_REPO_TOKEN` | `ghp_xxxxxxxxxxxx` | From Step 4 |
| `GITOPS_REPO_URL` | `https://github.com/YOUR_ORG/Gitops-pipeline.git` | Your GitOps repo URL |

## How the Pipeline Works

### 1. Code Push Triggers CI

**When you push code:**
```bash
git add .
git commit -m "Update application"
git push origin main
```

**GitHub Actions automatically:**
- Detects the push
- Starts CI workflow
- Runs tests (if configured)
- Builds Docker images
- Pushes to ECR

### 2. CI Workflow Steps

**Typical CI workflow:**

```yaml
1. Checkout code
   ↓
2. Set up AWS credentials (OIDC)
   ↓
3. Configure AWS CLI
   ↓
4. Login to ECR
   ↓
5. Build backend Docker image
   ↓
6. Tag image with commit SHA
   ↓
7. Push backend image to ECR
   ↓
8. Build frontend Docker image
   ↓
9. Tag image with commit SHA
   ↓
10. Push frontend image to ECR
    ↓
11. Update GitOps repository
    - Clone GitOps repo
    - Update image tags in deployment manifests
    - Commit and push changes
```

### 3. GitOps Repository Update

**The CI pipeline updates:**
- `Gitops-pipeline/apps/sample-saas-app/base/deployment.yaml`
  - Updates `image:` tags with new ECR image tags

**Example update:**
```yaml
# Before
image: 123456789012.dkr.ecr.us-east-1.amazonaws.com/backend:abc123

# After (new commit)
image: 123456789012.dkr.ecr.us-east-1.amazonaws.com/backend:def456
```

### 4. ArgoCD Detects Changes

**ArgoCD automatically:**
- Polls GitOps repository (every 3 minutes by default)
- Detects image tag changes
- Compares desired state vs. current state
- Shows application as "OutOfSync"

### 5. ArgoCD Syncs Deployment

**ArgoCD syncs:**
- Updates Kubernetes deployment with new image tag
- Kubernetes pulls new image from ECR
- Rolling update strategy replaces old pods
- New version is deployed

## Database Initialization

### Overview

Database initialization happens **before** application deployment. The database must be ready before applications start.

### Initialization Flow

```
Infrastructure Deployment (Terraform)
    ↓
RDS PostgreSQL Created
    ↓
Database Credentials in AWS Secrets Manager
    ↓
Kubernetes Secrets Created (from Secrets Manager)
    ↓
Database Init Job Runs
    ↓
Database Tables Created
    ↓
Application Pods Start
    ↓
Application Connects to Database
```

### Step 1: Infrastructure Creates Database

**When you run:**
```bash
cd cloudnative-saas-eks/examples/dev-environment/infrastructure
terraform apply
```

**Terraform creates:**
- RDS PostgreSQL instance
- Database credentials
- AWS Secrets Manager secret with credentials

### Step 2: Tenants Terraform Creates Kubernetes Secrets

**When you run:**
```bash
cd ../tenants
terraform apply
```

**Terraform:**
- Reads database credentials from AWS Secrets Manager
- Creates Kubernetes secrets in each tenant namespace:
  - `postgresql-secret` (db-user, db-password)
  - `backend-config` ConfigMap (db-host, db-port, db-name)

### Step 3: Database Initialization Job

**The init job is defined in:**
`Gitops-pipeline/apps/sample-saas-app/base/init-db-job.yaml`

**When ArgoCD syncs the application:**
1. Creates the init job
2. Job pod starts
3. Connects to RDS using secrets
4. Runs SQL migrations from ConfigMap
5. Creates tenant schemas and tables
6. Job completes successfully

**Init Job Configuration:**
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: init-rds-database
spec:
  template:
    spec:
      containers:
      - name: init-db
        image: postgres:15-alpine
        command:
        - /bin/sh
        - -c
        - |
          # Run migrations
          psql -h $DB_HOST -U $DB_USER -d $DB_NAME -f /migrations/001_create_tenants.sql
          psql -h $DB_HOST -U $DB_USER -d $DB_NAME -f /migrations/002_create_users.sql
          psql -h $DB_HOST -U $DB_USER -d $DB_NAME -f /migrations/003_create_tasks.sql
        env:
        - name: DB_HOST
          valueFrom:
            configMapKeyRef:
              name: backend-config
              key: db-host
        - name: DB_USER
          valueFrom:
            secretKeyRef:
              name: postgresql-secret
              key: db-user
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: postgresql-secret
              key: db-password
        - name: DB_NAME
          valueFrom:
            configMapKeyRef:
              name: backend-config
              key: db-name
        volumeMounts:
        - name: migrations
          mountPath: /migrations
      volumes:
      - name: migrations
        configMap:
          name: db-init-scripts
      restartPolicy: Never
```

### Step 4: Verify Database Initialization

**Check init job status:**
```bash
# Check job
kubectl get jobs -n platform
kubectl get jobs -n analytics

# Check job logs
kubectl logs job/init-rds-database -n platform
kubectl logs job/init-rds-database -n analytics

# Verify job completed
kubectl describe job init-rds-database -n platform
```

**Expected output:**
```
Status: Complete
Pods Statuses: 1 Succeeded
```

**Verify database tables:**
```bash
# Connect to database
kubectl run -it --rm --image=postgres:15-alpine db-client -- sh

# Inside pod
psql -h <db-host> -U <db-user> -d <db-name>

# Check schemas
\dn

# Check tables
\dt tenant_platform.*
\dt tenant_analytics.*
```

## Complete Deployment Flow

### First-Time Deployment

```
1. Infrastructure Setup
   ├─> Deploy VPC, EKS, RDS (Terraform)
   ├─> Create ECR repositories
   └─> Store credentials in Secrets Manager
   
2. Tenant Setup
   ├─> Create namespaces (platform, analytics)
   ├─> Create Kubernetes secrets from Secrets Manager
   └─> Create ConfigMaps with database config
   
3. GitOps Setup
   ├─> Install ArgoCD
   ├─> Configure ArgoCD applications
   └─> Bootstrap app-of-apps
   
4. Database Initialization
   ├─> ArgoCD creates init job
   ├─> Job runs SQL migrations
   └─> Database tables created
   
5. Application Deployment
   ├─> Push code to GitHub
   ├─> CI builds and pushes images
   ├─> GitOps repo updated
   ├─> ArgoCD syncs deployment
   └─> Application running
```

### Subsequent Deployments

```
1. Developer pushes code
   ↓
2. CI pipeline runs
   ├─> Builds new images
   ├─> Pushes to ECR
   └─> Updates GitOps repo
   ↓
3. ArgoCD detects changes
   ↓
4. ArgoCD syncs deployment
   ↓
5. Kubernetes rolling update
   ↓
6. New version running
```

## Manual Database Initialization

If you need to manually initialize the database:

### Option 1: Run Init Job Manually

```bash
# Apply init job
kubectl apply -f Gitops-pipeline/apps/sample-saas-app/base/init-db-job.yaml -n platform

# Wait for completion
kubectl wait --for=condition=complete job/init-rds-database -n platform --timeout=300s

# Check logs
kubectl logs job/init-rds-database -n platform
```

### Option 2: Use Script

```bash
cd Sample-saas-app/scripts

# Bash
./init-rds-database.sh

# PowerShell
./init-rds-database.ps1
```

### Option 3: Direct SQL Connection

```bash
# Get database credentials
DB_HOST=$(kubectl get configmap backend-config -n platform -o jsonpath='{.data.db-host}')
DB_USER=$(kubectl get secret postgresql-secret -n platform -o jsonpath='{.data.db-user}' | base64 -d)
DB_PASSWORD=$(kubectl get secret postgresql-secret -n platform -o jsonpath='{.data.db-password}' | base64 -d)
DB_NAME=$(kubectl get configmap backend-config -n platform -o jsonpath='{.data.db-name}')

# Connect and run migrations
kubectl run -it --rm --image=postgres:15-alpine db-client -- \
  psql -h $DB_HOST -U $DB_USER -d $DB_NAME -f /path/to/migration.sql
```

## Troubleshooting CI/CD

### CI Pipeline Fails

**Issue**: Build fails
- **Check**: GitHub Actions logs
- **Solution**: Verify Dockerfile syntax, dependencies

**Issue**: ECR push fails
- **Check**: AWS credentials, IAM role permissions
- **Solution**: Verify `AWS_ROLE_ARN` secret, check IAM role policies

**Issue**: GitOps update fails
- **Check**: `GITOPS_REPO_TOKEN` validity
- **Solution**: Regenerate token, verify repo access

### ArgoCD Not Syncing

**Issue**: Application shows "OutOfSync"
- **Check**: `kubectl get application -n argocd`
- **Solution**: Manually sync in ArgoCD UI or `argocd app sync <app-name>`

**Issue**: Image pull errors
- **Check**: Pod events, ECR access
- **Solution**: Verify image tags, check ECR repository permissions

### Database Init Fails

**Issue**: Init job fails
- **Check**: `kubectl logs job/init-rds-database -n <namespace>`
- **Solution**: Verify database credentials, check RDS security group rules

**Issue**: Tables not created
- **Check**: Database connection, SQL syntax
- **Solution**: Review migration files, check database logs

## Best Practices

### 1. Image Tagging
- Use commit SHA for traceability
- Tag with version numbers for releases
- Never use `latest` in production

### 2. Database Migrations
- Make migrations idempotent
- Test migrations in dev first
- Always backup before production migrations

### 3. GitOps Updates
- Commit meaningful messages
- Review changes before pushing
- Use feature branches for testing

### 4. ArgoCD Sync
- Enable auto-sync for dev environments
- Manual sync for production
- Use sync policies appropriately

## Quick Reference

### Check CI/CD Status

```bash
# GitHub Actions
# Go to: https://github.com/YOUR_ORG/Sample-saas-app/actions

# ArgoCD applications
kubectl get applications -n argocd

# Recent deployments
kubectl get deployments -n platform
kubectl get deployments -n analytics
```

### Trigger Manual Build

```bash
# Push a commit
git commit --allow-empty -m "Trigger CI/CD"
git push origin main

# Or use GitHub CLI
gh workflow run "CI - Build and Push Images"
```

### View Pipeline Logs

```bash
# GitHub Actions (via web UI)
# Or use GitHub CLI
gh run list
gh run view <run-id>
```

## Next Steps

- **Read [07-Deployment-Guide.md](07-Deployment-Guide.md)** - Complete deployment process
- **Review [09-Troubleshooting.md](09-Troubleshooting.md)** - Common issues
- **Check [18-Tenant-Provisioning.md](18-Tenant-Provisioning.md)** - Add tenants

---

**Need help?** Check [09-Troubleshooting.md](09-Troubleshooting.md) or review the [CI/CD Setup Guide](../Sample-saas-app/CI_CD_SETUP.md)

