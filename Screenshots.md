# Platform Screenshots

Visual documentation of the CloudNative SaaS Platform components, dashboards, and interfaces.

## 📊 Grafana Dashboards

### Sample App Dashboard

**Location**: Grafana → Dashboards → Sample App Dashboard

**What to capture:**
- Overall dashboard view showing all panels
- Memory Usage by Namespace panel
- CPU Usage by Namespace panel
- Pods per Namespace gauge
- Node CPU and Memory usage

**Screenshot:**
<!-- Add screenshot: screenshots/grafana-sample-app-dashboard.png -->
![Sample App Dashboard - Overview](screenshots/grafana-sample-app-dashboard.png)

**Description**: Complete view of the Sample App Dashboard showing namespace-level and node-level metrics. The dashboard displays:
- **Memory Usage by Namespace**: Bar gauge showing memory consumption per namespace (analytics: 37.8 MB/s, argocd: 623 MB/s, kube-system: 231 MB/s, monitoring: 644 MB/s, platform: 37.0 MB/s)
- **CPU Usage by Namespace**: Time-series graph showing CPU trends with a spike around 02:19-02:20
- **Pods per Namespace**: Individual gauges showing pod counts (analytics: 3, argocd: 7, kube-system: 10, monitoring: 7, platform: 3)
- **Node CPU Usage**: Overall cluster CPU at 3.69% (very low)
- **Node Memory Usage**: Average node memory at 19.8% (low utilization)

---


## 🗄️ Database (RDS PostgreSQL)

### Tenants Table Query

**What to capture:**
- Query results from `tenants` table
- Platform and Analytics tenant records
- Tenant metadata (id, tenant_id, name, namespace, timestamps)

**Screenshot:**
<!-- Add screenshot: screenshots/database-tenants-table.png -->
![Tenants Table Query Results](screenshots/database-tenants-table.png)

**Description**: Query results showing all tenants in the database with their metadata including tenant_id, name, namespace, and creation timestamps.

---

### Platform Tenant Users

**What to capture:**
- Users in `tenant_platform.users` table
- User authentication data
- Tenant isolation verification

**Screenshot:**
<!-- Add screenshot: screenshots/database-platform-users.png -->
![Platform Tenant Users](screenshots/database-platform-users.png)

**Description**: User records from the platform tenant schema, showing email, password hash, name, and tenant_id for data isolation verification.

---

### Analytics Tenant Users

**What to capture:**
- Users in `tenant_analytics.users` table
- User authentication data
- Tenant isolation verification

**Screenshot:**
<!-- Add screenshot: screenshots/database-analytics-users.png -->
![Analytics Tenant Users](screenshots/database-analytics-users.png)

**Description**: User records from the analytics tenant schema, confirming proper schema-level isolation between tenants.

---

## 🚀 Application (Sample SaaS App)

### Platform Tenant Dashboard

**What to capture:**
- Task Management Sample App for Platform tenant
- Task dashboard with statistics
- Resource usage metrics
- User information display

**Screenshot:**
<!-- Add screenshot: screenshots/app-platform-dashboard.png -->
![Platform Tenant Dashboard](screenshots/app-platform-dashboard.png)

**Description**: Task Management Sample App dashboard for the Platform tenant showing:
- User: Platform Team (platform-1@gmail.com)
- Task statistics: Total Tasks, To Do, In Progress, Done (all showing 0)
- Resource Usage: CPU (2.5/20 cores), Memory (8.5/40Gi), Storage (88 kB/200Gi), Pods (5/200)
- Clean, modern UI with resource usage progress bars

---

### Analytics Tenant Dashboard

**What to capture:**
- Task Management Sample App for Analytics tenant
- Task dashboard with statistics
- Resource usage metrics
- User information display

**Screenshot:**
<!-- Add screenshot: screenshots/app-analytics-dashboard.png -->
![Analytics Tenant Dashboard](screenshots/app-analytics-dashboard.png)

**Description**: Task Management Sample App dashboard for the Analytics tenant showing:
- User: Analytics Team (analytics@gmail.com)
- Task statistics: All categories showing 0 tasks
- Resource Usage: CPU (2.5/15 cores), Memory (8.5/30Gi), Storage (88 kB/150Gi), Pods (5/180)
- Demonstrates proper tenant isolation with different resource quotas


## ☸️ Kubernetes Resources

### Kubernetes Verification Output

**What to capture:**
- Namespace listing (analytics, data, platform)
- Resource quotas per namespace
- Network policies configuration
- Service accounts and RBAC roles

**Screenshot:**
<!-- Add screenshot: screenshots/k8s-verification-output.png -->
![Kubernetes Verification](screenshots/k8s-verification-output.png)

**Description**: Complete verification output showing all tenant namespaces, their resource quotas (CPU, memory, storage, pods), network policies with pod selectors, and service accounts. Confirms proper multi-tenant isolation and resource governance.



## 🏗️ Architecture Diagrams

### Complete AWS Cloud-Native SaaS Platform Architecture

**Location**: Architecture documentation

**What to capture:**
- Complete multi-tenant architecture diagram
- VPC structure with 3 Availability Zones
- EKS cluster with all namespaces and components
- ALB with host-based routing
- RDS PostgreSQL with high availability
- All AWS service integrations
- Data flow and connectivity

**Screenshot:**
<!-- Add screenshot: screenshots/complete-aws-saas-architecture.png -->
![Complete AWS Cloud-Native SaaS Platform Architecture](screenshots/complete-aws-saas-architecture.jpeg)

**Description**: Comprehensive architecture diagram showing the complete CloudNative SaaS Platform:

**User Access Layer:**
- **Multi-Tenant Users**: Tenant A (`tenant-a.app.com`), Tenant B (`tenant-b.app.com`), Tenant C (`tenant-c.app.com`)
- **Platform Admin**: `admin.app.com`
- **Internet**: HTTPS requests (Port 443) from external users

**Load Balancing & Routing:**
- **Application Load Balancer (ALB)**: Internet-facing, HTTPS enabled
- **Host-Based Routing**: Routes requests to specific Kubernetes namespaces based on subdomain:
  - `tenant-a.*` → Tenant A Namespace
  - `tenant-b.*` → Tenant B Namespace
  - `tenant-c.*` → Tenant C Namespace
  - `admin.*` → Platform Namespace

**Network Infrastructure:**
- **Main VPC**: Multi-AZ deployment across 3 Availability Zones (A, B, C)
- **Public Subnets**: One per AZ, containing NAT Gateways for outbound internet access
- **Private Subnets - EKS**: EC2 Managed Nodes (worker nodes) with network-isolated pods
- **Private Subnets - Data**: RDS PostgreSQL database instances

**EKS Cluster Components:**
- **Platform Namespace**: Shared services for platform administration
- **Analytics Namespace**: Data processing workloads
- **Monitoring Namespace**: Observability stack
  - **Prometheus**: Metrics collection and storage
  - **Grafana**: Visualization and dashboards
- **Tenant Namespaces**: `tenant-a`, `tenant-b`, `tenant-c`
  - **Isolation Mechanisms**: Network Policies + RBAC + Resource Quotas
  - **Per-tenant isolation**: Separate service accounts, secrets, and resource limits

**GitOps & Deployment:**
- **Argo CD**: GitOps continuous delivery tool
  - Deploys applications to tenant namespaces
  - Manages application lifecycle

**Data Layer:**
- **RDS PostgreSQL**: Multi-tenant database
  - **AZ A**: Primary instance
  - **AZ B**: Standby instance (high availability)
  - **Data Isolation**: Row-level security filtering by `tenant_id`
  - **Connections**: Monitoring and Argo CD components connect to RDS

**External AWS Services:**
- **ECR (Elastic Container Registry)**: Docker image storage
  - EKS cluster pulls images from ECR
- **AWS Secrets Manager**: Secure credential storage
  - **Per-tenant secrets isolation**: DB credentials, API keys
  - Tenant components retrieve secrets dynamically
- **S3 Bucket (Terraform State)**: Infrastructure as Code state storage
  - Tenant components may access Terraform state

**Multi-Tenant Request Flow:**
1. **Request Routing**: User → Internet → ALB → Host-based routing → Kubernetes namespace
2. **Data Isolation**: Application pods use tenant-specific credentials → RDS with row-level security → Filtered by `tenant_id`
3. **Network Isolation**: Network policies prevent cross-tenant traffic within Kubernetes

**Key Architectural Features:**
- ✅ **Multi-AZ Deployment**: High availability across 3 availability zones
- ✅ **EC2 Managed Node Groups**: Simplified EKS worker node management
- ✅ **Network Isolation**: Network policies enforce tenant traffic separation
- ✅ **Resource Governance**: Resource quotas per tenant
- ✅ **GitOps Deployment**: Argo CD for automated application deployment
- ✅ **Complete Observability**: Prometheus + Grafana monitoring stack
- ✅ **Secure Secrets Management**: AWS Secrets Manager with per-tenant isolation
- ✅ **Infrastructure as Code**: Terraform with S3 state backend

**Note**: This diagram shows a generic multi-tenant architecture. The current project implementation uses:
- Tenant names: `platform` and `analytics` (instead of `tenant-a/b/c`)
- Database isolation: Schema-level (`tenant_platform`, `tenant_analytics`) instead of row-level security
- Both approaches are valid multi-tenant patterns

---

## 🔐 AWS Resources

### Terraform Infrastructure Outputs

**What to capture:**
- Terraform output command results
- EKS cluster information
- VPC and networking details
- Security group IDs
- IAM role ARNs

**Screenshot:**
<!-- Add screenshot: screenshots/terraform-outputs.png -->
![Terraform Infrastructure Outputs](screenshots/terraform-outputs.png)

**Description**: Complete Terraform outputs showing:
- **Cluster**: `saas-infra-lab-dev` (v1.32) in `us-east-1`
- **VPC**: `vpc-0747522334f1ea9d7`
- **Subnets**: 3 public and 3 private subnet IDs
- **Security Groups**: Cluster SG (`sg-08a81b2255b144adf`), Nodes SG (`sg-01bcef5f509ddb257`)
- **IAM Roles**: Cluster role and Node role ARNs
- **kubeconfig command**: Ready-to-use command for cluster access



## 🔧 CI/CD Pipeline

### CI Workflow (ci.yml)

**Location**: GitHub Actions → Workflows → ci.yml

**What to capture:**
- CI workflow triggered on push
- All test jobs completed successfully
- Execution times for each job

**Screenshot:**
<!-- Add screenshot: screenshots/ci-workflow.png -->
![CI Workflow](screenshots/ci-workflow.png)

**Description**: Successful CI pipeline execution showing:
- **backend-test**: Completed in 20s
- **frontend-test**: Completed in 11s
- **Security & Code Quality Checks**: Completed in 5s
- All jobs passed with green checkmarks

---

### CD Workflow (cd.yml)

**Location**: GitHub Actions → Workflows → cd.yml

**What to capture:**
- CD workflow triggered by workflow_run
- Build stages (Backend and Frontend in parallel)
- GitOps manifest update step

**Screenshot:**
<!-- Add screenshot: screenshots/cd-workflow.png -->
![CD Workflow](screenshots/cd-workflow.png)

**Description**: Successful CD pipeline execution showing:
- **Check CI Status**: Completed in 2s (validates CI passed)
- **Build Backend**: Completed in 43s (parallel with frontend)
- **Build Frontend**: Completed in 26s (parallel with backend)
- **Update GitOps Manifest**: Completed in 7s (updates image tags in GitOps repo)
- All stages connected sequentially with successful completion

---

### Infrastructure Deployment Workflow (auto-apply-infra.yml)

**Location**: GitHub Actions → Workflows → auto-apply-infra.yml

**What to capture:**
- Infrastructure deployment workflow
- ArgoCD installation step
- Sequential execution flow

**Screenshot:**
<!-- Add screenshot: screenshots/infrastructure-deployment-workflow.png -->
![Infrastructure Deployment Workflow](screenshots/infrastructure-deployment-workflow.png)

**Description**: Successful infrastructure deployment showing:
- **Deploy Infrastructure**: Completed in 50s (Terraform apply for EKS, VPC, RDS, etc.)
- **Install ArgoCD**: Completed in 22s (installs ArgoCD in the cluster)
- Workflow triggered by `repository_dispatch` event
- Both steps completed successfully with green checkmarks

---

### Deployment Success Message

**What to capture:**
- Deployment completion confirmation
- Infrastructure components deployed
- Tenant namespaces created
- Verification output

**Screenshot:**
<!-- Add screenshot: screenshots/deployment-success.png -->
![Deployment Success](screenshots/deployment-success.png)

**Description**: Terminal output showing successful deployment with:
- **Infrastructure (Phase 1)**: EKS Cluster (saas-infra-lab-dev v1.32), VPC, Security Groups, IAM Roles, EKS Access Entries, Managed Node Group
- **Tenants (Phase 2)**: 3 Tenant Namespaces (platform, data, analytics), Resource Quotas, Network Policies, Service Accounts with IRSA, RBAC Roles & Role Bindings
- **Verification**: All three tenant namespaces showing as Active


## 💡 Tips

- **Remove sensitive data**: Blur or redact credentials, tokens, IPs before committing
- **Use descriptive names**: Follow the naming convention used in this document
- **Keep file sizes reasonable**: Compress large images if needed

## 🔒 Security Note

**Important**: Before sharing screenshots publicly:
- ✅ Remove or blur sensitive information (credentials, tokens, IPs)
- ✅ Redact AWS account IDs
- ✅ Remove personal data
- ✅ Check for exposed secrets in logs or UI

---

**Ready to add screenshots?** Save them in `Docs/screenshots/` and update this file with the image references!

