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

### Node Exporter Dashboard

**Location**: Grafana → Dashboards → Node Exporter Full

**What to capture:**
- Node metrics overview
- CPU usage graphs
- Memory usage graphs
- Disk I/O metrics
- Network metrics


---

## 🗄️ Database (RDS PostgreSQL)

### Database Connection

**What to capture:**
- Successful connection to RDS
- Database version information
- Available schemas

**Screenshot:**
<!-- Add screenshot: screenshots/database-connection.png -->
![Database Connection](screenshots/database-connection.png)

**Description**: Successful connection to RDS PostgreSQL instance.

---

### Tenant Schemas

**What to capture:**
- List of tenant schemas (tenant_platform, tenant_analytics)
- Schema structure
- Tables in each schema

**Screenshot:**
<!-- Add screenshot: screenshots/database-tenant-schemas.png -->
![Tenant Schemas](screenshots/database-tenant-schemas.png)

**Description**: Multi-tenant database structure with isolated schemas.

---

### Database Tables

**What to capture:**
- Tables in tenant_platform schema
- Tables in tenant_analytics schema
- Table structures (columns, constraints)

**Screenshot:**
<!-- Add screenshot: screenshots/database-tables.png -->
![Database Tables](screenshots/database-tables.png)

**Description**: Database tables created by migration scripts.

---

### Sample Data

**What to capture:**
- Sample records from users table
- Sample records from tasks table
- Data isolation between tenants

**Screenshot:**
<!-- Add screenshot: screenshots/database-sample-data.png -->
![Sample Database Data](screenshots/database-sample-data.png)

**Description**: Sample data showing tenant isolation.

---

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

### Frontend Application

**What to capture:**
- Application homepage
- User registration/login
- Task management interface
- Application running in browser

**Screenshot:**
<!-- Add screenshot: screenshots/app-frontend-homepage.png -->
![Frontend Application - Homepage](screenshots/app-frontend-homepage.png)

**Description**: Frontend application accessible via LoadBalancer URL.

---

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

---

### Backend API

**What to capture:**
- API health check endpoint
- API documentation (if available)
- Successful API responses

**Screenshot:**
<!-- Add screenshot: screenshots/app-backend-api.png -->
![Backend API Health Check](screenshots/app-backend-api.png)

**Description**: Backend API responding successfully.

---

### Application LoadBalancer

**What to capture:**
- LoadBalancer service in Kubernetes
- External IP/DNS name
- Service endpoints

**Screenshot:**
<!-- Add screenshot: screenshots/app-loadbalancer.png -->
![Application LoadBalancer](screenshots/app-loadbalancer.png)

**Description**: LoadBalancer service providing external access to the application.

---

## ☸️ Kubernetes Resources

### Pod Status

**What to capture:**
- All pods running successfully
- Pod status across namespaces
- Resource usage

**Screenshot:**
<!-- Add screenshot: screenshots/k8s-pods.png -->
![Kubernetes Pods](screenshots/k8s-pods.png)

**Description**: All application pods running successfully in their respective namespaces.

---

### Namespaces and Resources

**What to capture:**
- All namespaces (platform, analytics, monitoring)
- Resource quotas
- Network policies

**Screenshot:**
<!-- Add screenshot: screenshots/k8s-namespaces.png -->
![Kubernetes Namespaces](screenshots/k8s-namespaces.png)

**Description**: Multi-tenant namespaces with resource quotas and isolation.

---

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

---

### Services

**What to capture:**
- All services across namespaces
- LoadBalancer services with external IPs
- Service endpoints

**Screenshot:**
<!-- Add screenshot: screenshots/k8s-services.png -->
![Kubernetes Services](screenshots/k8s-services.png)

**Description**: Services exposing applications and monitoring stack.

---

### Deployments

**What to capture:**
- Deployment status
- Replica sets
- Rolling update status

**Screenshot:**
<!-- Add screenshot: screenshots/k8s-deployments.png -->
![Kubernetes Deployments](screenshots/k8s-deployments.png)

**Description**: Application deployments with desired replicas running.

---

## 🔄 ArgoCD (GitOps)

### ArgoCD Applications

**What to capture:**
- List of all ArgoCD applications
- Sync status (Synced/OutOfSync)
- Health status (Healthy/Degraded)

**Screenshot:**
<!-- Add screenshot: screenshots/argocd-applications.png -->
![ArgoCD Applications](screenshots/argocd-applications.png)

**Description**: All ArgoCD applications showing sync and health status.

---

### Application Details

**What to capture:**
- Application resource tree
- Sync history
- Application events

**Screenshot:**
<!-- Add screenshot: screenshots/argocd-application-details.png -->
![ArgoCD Application Details](screenshots/argocd-application-details.png)

**Description**: Detailed view of an ArgoCD application showing resources and sync status.

---

### ArgoCD UI

**What to capture:**
- ArgoCD dashboard
- Application topology
- Resource visualization

**Screenshot:**
<!-- Add screenshot: screenshots/argocd-ui.png -->
![ArgoCD UI Dashboard](screenshots/argocd-ui.png)

**Description**: ArgoCD web interface showing application management.

---

## 🔐 AWS Resources

### EKS Cluster

**What to capture:**
- EKS cluster details in AWS Console
- Cluster status
- Node groups

**Screenshot:**
<!-- Add screenshot: screenshots/aws-eks-cluster.png -->
![AWS EKS Cluster](screenshots/aws-eks-cluster.png)

**Description**: EKS cluster running successfully with managed node groups.

---

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

---

### RDS Database

**What to capture:**
- RDS instance details
- Database status
- Connection information

**Screenshot:**
<!-- Add screenshot: screenshots/aws-rds-instance.png -->
![AWS RDS Instance](screenshots/aws-rds-instance.png)

**Description**: RDS PostgreSQL instance running and accessible.

---

### ECR Repositories

**What to capture:**
- ECR repositories list
- Container images
- Image tags

**Screenshot:**
<!-- Add screenshot: screenshots/aws-ecr-repositories.png -->
![AWS ECR Repositories](screenshots/aws-ecr-repositories.png)

**Description**: ECR repositories containing application container images.

---

### Secrets Manager

**What to capture:**
- Secrets Manager secrets
- Database credentials secret
- Secret rotation status

**Screenshot:**
<!-- Add screenshot: screenshots/aws-secrets-manager.png -->
![AWS Secrets Manager](screenshots/aws-secrets-manager.png)

**Description**: Database credentials stored securely in AWS Secrets Manager.

---

## 📈 Monitoring & Observability

### Prometheus Targets

**What to capture:**
- Prometheus targets page
- Scrape targets status
- ServiceMonitor discoveries

**Screenshot:**
<!-- Add screenshot: screenshots/prometheus-targets.png -->
![Prometheus Targets](screenshots/prometheus-targets.png)

**Description**: Prometheus successfully scraping metrics from all targets.

---

### Prometheus Queries

**What to capture:**
- Prometheus query interface
- Sample queries and results
- Metric exploration

**Screenshot:**
<!-- Add screenshot: screenshots/prometheus-queries.png -->
![Prometheus Query Interface](screenshots/prometheus-queries.png)

**Description**: Prometheus query interface showing metric exploration.

---

### CloudWatch Logs

**What to capture:**
- CloudWatch log groups
- Application logs
- EKS control plane logs

**Screenshot:**
<!-- Add screenshot: screenshots/cloudwatch-logs.png -->
![CloudWatch Logs](screenshots/cloudwatch-logs.png)

**Description**: CloudWatch log groups for application and infrastructure logging.

---

## 🔧 CI/CD Pipeline

### GitHub Actions Workflow

**What to capture:**
- Successful CI/CD workflow run
- Build and push steps
- GitOps update step

**Screenshot:**
<!-- Add screenshot: screenshots/github-actions-workflow.png -->
![GitHub Actions Workflow](screenshots/github-actions-workflow.png)

**Description**: Successful CI/CD pipeline execution building and deploying application.

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

---

### Docker Image Build

**What to capture:**
- Docker image build logs
- Image push to ECR
- Image tags

**Screenshot:**
<!-- Add screenshot: screenshots/docker-build.png -->
![Docker Image Build](screenshots/docker-build.png)

**Description**: Docker images successfully built and pushed to ECR.

---

## 📝 Screenshot Checklist

Use this checklist to ensure you capture all important aspects:

### Infrastructure
- [ ] EKS cluster in AWS Console
- [ ] RDS instance details
- [ ] VPC and networking
- [ ] Security groups
- [ ] IAM roles

### Kubernetes
- [ ] All pods running
- [ ] Services and LoadBalancers
- [ ] Namespaces and resource quotas
- [ ] Deployments and ReplicaSets
- [ ] ConfigMaps and Secrets

### Applications
- [ ] Frontend application UI
- [ ] Backend API responses
- [ ] Application logs
- [ ] Health checks

### Monitoring
- [ ] Grafana dashboards
- [ ] Prometheus targets
- [ ] Prometheus queries
- [ ] CloudWatch metrics

### GitOps
- [ ] ArgoCD applications
- [ ] ArgoCD sync status
- [ ] Application resource tree

### Database
- [ ] Database connection
- [ ] Tenant schemas
- [ ] Database tables
- [ ] Sample data

### CI/CD
- [ ] GitHub Actions workflows
- [ ] Docker builds
- [ ] ECR repositories

## 📁 Screenshot Directory Structure

```
Docs/
└── screenshots/
    ├── grafana/
    │   ├── sample-app-dashboard.png
    │   ├── node-exporter.png
    │   └── kubernetes-cluster.png
    ├── database/
    │   ├── connection.png
    │   ├── tenant-schemas.png
    │   └── tables.png
    ├── application/
    │   ├── frontend-homepage.png
    │   └── backend-api.png
    ├── kubernetes/
    │   ├── pods.png
    │   ├── services.png
    │   └── namespaces.png
    ├── argocd/
    │   ├── applications.png
    │   └── ui.png
    ├── aws/
    │   ├── eks-cluster.png
    │   ├── rds-instance.png
    │   └── ecr-repositories.png
    └── cicd/
        ├── github-actions.png
        └── docker-build.png
```

## 💡 Tips for Good Screenshots

1. **Use high resolution**: Capture at 1920x1080 or higher
2. **Remove sensitive data**: Blur or redact credentials, tokens, IPs
3. **Add annotations**: Use arrows or text to highlight important areas
4. **Consistent naming**: Use descriptive, consistent file names
5. **Update regularly**: Keep screenshots current with latest versions
6. **Show context**: Include enough UI context to understand the view

## 🔒 Security Note

**Important**: Before sharing screenshots publicly:
- ✅ Remove or blur sensitive information (credentials, tokens, IPs)
- ✅ Redact AWS account IDs
- ✅ Remove personal data
- ✅ Check for exposed secrets in logs or UI

---

**Ready to add screenshots?** Save them in `Docs/screenshots/` and update this file with the image references!

