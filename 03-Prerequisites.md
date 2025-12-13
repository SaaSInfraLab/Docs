# Prerequisites

Complete setup guide for all required tools and AWS configuration.

## Required Tools

### 1. AWS CLI

**Installation:**

**macOS:**
```bash
brew install awscli
```

**Linux:**
```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

**Windows:**
Download from: https://aws.amazon.com/cli/

**Verify:**
```bash
aws --version
# Should show: aws-cli/2.x.x
```

**Configuration:**
```bash
aws configure
# AWS Access Key ID: [Your Access Key]
# AWS Secret Access Key: [Your Secret Key]
# Default region name: us-east-1
# Default output format: json
```

### 2. Terraform

**Installation:**

**macOS:**
```bash
brew install terraform
```

**Linux:**
```bash
wget https://releases.hashicorp.com/terraform/1.6.0/terraform_1.6.0_linux_amd64.zip
unzip terraform_1.6.0_linux_amd64.zip
sudo mv terraform /usr/local/bin/
```

**Windows:**
Download from: https://www.terraform.io/downloads

**Verify:**
```bash
terraform version
# Should show: Terraform v1.6.0
```

### 3. kubectl

**Installation:**

**macOS:**
```bash
brew install kubectl
```

**Linux:**
```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

**Windows:**
```powershell
choco install kubernetes-cli
```

**Verify:**
```bash
kubectl version --client
```

### 4. Git

**Installation:**

**macOS:**
```bash
brew install git
```

**Linux:**
```bash
sudo apt-get update
sudo apt-get install git
```

**Windows:**
Download from: https://git-scm.com/download/win

**Verify:**
```bash
git --version
```

### 5. Docker (Optional - for local development)

**Installation:**

**macOS:**
```bash
brew install --cask docker
```

**Linux:**
```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

**Windows:**
Download Docker Desktop from: https://www.docker.com/products/docker-desktop

**Verify:**
```bash
docker --version
```

## AWS Account Setup

### 1. Create AWS Account

If you don't have an AWS account:
1. Go to https://aws.amazon.com/
2. Click "Create an AWS Account"
3. Follow the registration process

### 2. Create IAM User

**For Terraform operations:**

1. Go to IAM Console → Users → Add users
2. Username: `terraform-user` (or your choice)
3. Select: "Programmatic access"
4. Attach policy: `AdministratorAccess` (for development)
   - **Note**: For production, use least-privilege policies
5. Save Access Key ID and Secret Access Key

### 3. Configure AWS CLI

```bash
aws configure
# AWS Access Key ID: [Your Access Key]
# AWS Secret Access Key: [Your Secret Key]
# Default region name: us-east-1
# Default output format: json
```

**Verify:**
```bash
aws sts get-caller-identity
# Should show your account ID and user ARN
```

### 4. Required AWS Permissions

The IAM user needs permissions for:
- EC2 (VPC, subnets, security groups)
- EKS (cluster creation and management)
- RDS (database creation)
- IAM (role and policy creation)
- Secrets Manager (secret creation)
- ECR (container registry)
- CloudWatch (logging and monitoring)
- S3 (Terraform state storage)

**Minimum Required Policies:**
- `AmazonEC2FullAccess`
- `AmazonEKSClusterPolicy`
- `AmazonRDSFullAccess`
- `IAMFullAccess`
- `SecretsManagerReadWrite`
- `AmazonEC2ContainerRegistryFullAccess`
- `CloudWatchFullAccess`
- `AmazonS3FullAccess`

**For Development:** `AdministratorAccess` simplifies setup

### 5. Set Up S3 Backend for Terraform State

**Create S3 bucket:**
```bash
# Replace with your unique bucket name
BUCKET_NAME="my-saas-platform-terraform-state-$(date +%s)"

aws s3 mb s3://$BUCKET_NAME --region us-east-1

# Enable versioning
aws s3api put-bucket-versioning \
  --bucket $BUCKET_NAME \
  --versioning-configuration Status=Enabled

# Enable encryption
aws s3api put-bucket-encryption \
  --bucket $BUCKET_NAME \
  --server-side-encryption-configuration '{
    "Rules": [{
      "ApplyServerSideEncryptionByDefault": {
        "SSEAlgorithm": "AES256"
      }
    }]
  }'
```

**Note the bucket name** - you'll use it in backend configuration.

## GitHub Setup (for CI/CD)

### 1. Create GitHub Account

If you don't have one:
1. Go to https://github.com/
2. Sign up for a free account

### 2. Fork/Clone Repositories

```bash
# Fork these repositories to your GitHub account:
# - cloudnative-saas-eks
# - Gitops-pipeline
# - Sample-saas-app
```

### 3. Set Up GitHub Secrets

For CI/CD to work, add these secrets to your repositories:

**In `Sample-saas-app` repository:**

Go to Settings → Secrets and variables → Actions → New repository secret

Add:
- `AWS_ACCESS_KEY_ID` - Your AWS access key
- `AWS_SECRET_ACCESS_KEY` - Your AWS secret key
- `AWS_REGION` - `us-east-1`
- `ECR_REPOSITORY` - Your ECR repository URL
- `GITOPS_REPO_TOKEN` - GitHub token with repo access

**In `Gitops-pipeline` repository:**

Add:
- `AWS_ACCESS_KEY_ID` - Your AWS access key
- `AWS_SECRET_ACCESS_KEY` - Your AWS secret key
- `AWS_REGION` - `us-east-1`

### 4. Create GitHub Personal Access Token

1. Go to GitHub → Settings → Developer settings → Personal access tokens → Tokens (classic)
2. Generate new token
3. Scopes needed:
   - `repo` (full control)
   - `workflow` (update workflows)
4. Save the token securely

## Environment Variables

Create a `.env` file or export these variables:

```bash
# AWS Configuration
export AWS_REGION="us-east-1"
export AWS_ACCESS_KEY_ID="your-access-key"
export AWS_SECRET_ACCESS_KEY="your-secret-key"

# Terraform Backend
export TF_STATE_BUCKET="your-terraform-state-bucket"
export TF_STATE_KEY="dev/terraform.tfstate"
export TF_STATE_REGION="us-east-1"

# Kubernetes
export KUBECONFIG="$HOME/.kube/config"

# GitHub (for CI/CD)
export GITHUB_TOKEN="your-github-token"
```

## Verification Checklist

Before proceeding, verify:

- [ ] AWS CLI installed and configured
- [ ] Terraform installed (v1.5+)
- [ ] kubectl installed
- [ ] Git installed
- [ ] AWS credentials configured
- [ ] AWS account has required permissions
- [ ] S3 bucket created for Terraform state
- [ ] GitHub account set up
- [ ] GitHub repositories forked/cloned
- [ ] GitHub secrets configured (for CI/CD)

## Testing Your Setup

### Test AWS Access

```bash
aws sts get-caller-identity
# Should return your account ID and user ARN
```

### Test Terraform

```bash
terraform version
# Should show version 1.5+
```

### Test kubectl (after EKS cluster is created)

```bash
kubectl version --client
# Should show client version
```

### Test Git

```bash
git --version
# Should show git version
```

## Troubleshooting

### AWS CLI Issues

**Problem**: `aws: command not found`
- **Solution**: Ensure AWS CLI is in your PATH

**Problem**: `Unable to locate credentials`
- **Solution**: Run `aws configure` again

### Terraform Issues

**Problem**: `terraform: command not found`
- **Solution**: Ensure Terraform is in your PATH

**Problem**: Backend configuration errors
- **Solution**: Verify S3 bucket exists and you have permissions

### kubectl Issues

**Problem**: `kubectl: command not found`
- **Solution**: Install kubectl and add to PATH

**Problem**: Connection refused
- **Solution**: Configure kubeconfig after EKS cluster is created

## Next Steps

Once prerequisites are met:

1. **Follow [02-Quick-Start.md](02-Quick-Start.md)** - Deploy the platform
2. **Read [04-Architecture.md](04-Architecture.md)** - Understand the design
3. **Review [07-Deployment-Guide.md](07-Deployment-Guide.md)** - Detailed deployment

---

**Ready?** → [02-Quick-Start.md](02-Quick-Start.md)

