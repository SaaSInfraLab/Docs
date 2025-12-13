# System Architecture

Complete architecture documentation for the CloudNative SaaS Platform.

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    AWS Account                          │
│                                                         │
│  ┌────────────────────────────────────────────────┐     │
│  │              VPC (10.0.0.0/16)                 │     │
│  │                                                │     │
│  │  ┌──────────────┐  ┌──────────────┐            │     │
│  │  │ Public Subnet│  │ Public Subnet│            │     │
│  │  │  (10.0.101.x)│  │  (10.0.102.x)│            │     │
│  │  │              │  │              │            │     │
│  │  │  NAT Gateway │  │  NAT Gateway │            │     │
│  │  └──────────────┘  └──────────────┘            │     │
│  │                                                │     │
│  │  ┌──────────────────────────────────────────┐  │     │
│  │  │         EKS Control Plane                │  │     │
│  │  │    (Managed by AWS)                      │  │     │
│  │  └──────────────────────────────────────────┘  │     │
│  │                                                │     │
│  │  ┌──────────────┐  ┌──────────────┐            │     │
│  │  │Private Subnet│  │Private Subnet│            │     │
│  │  │  (10.0.1.x)  │  │  (10.0.2.x)  │            │     │
│  │  │              │  │              │            │     │
│  │  │  ┌────────┐  │  │  ┌────────┐  │            │     │
│  │  │  │Worker  │  │  │  │Worker  │  │            │     │
│  │  │  │Nodes   │  │  │  │Nodes   │  │            │     │
│  │  │  │        │  │  │  │        │  │            │     │
│  │  │  │ Pods:  │  │  │  │ Pods:  │  │            │     │
│  │  │  │ - App  │  │  │  │ - App  │  │            │     │
│  │  │  │ - Argo │  │  │  │ - Prom │  │            │     │
│  │  │  │ - Graf │  │  │  │ - Node │  │            │     │
│  │  │  └────────┘  │  │  └────────┘  │            │     │
│  │  └──────────────┘  └──────────────┘            │     │
│  └────────────────────────────────────────────────┘     │
│                                                         │
│  ┌────────────────────────────────────────────────┐     │
│  │         RDS PostgreSQL (Multi-tenant)          │     │
│  │    - Database per tenant or shared             │     │
│  │    - Automated backups                         │     │
│  │    - High availability                         │     │
│  └────────────────────────────────────────────────┘     │
│                                                         │
│  ┌────────────────────────────────────────────────┐     │
│  │         AWS Secrets Manager                    │     │
│  │    - Database credentials                      │     │
│  │    - API keys                                  │     │
│  │    - TLS certificates                          │     │
│  └────────────────────────────────────────────────┘     │
│                                                         │
│  ┌────────────────────────────────────────────────┐     │
│  │         ECR (Container Registry)               │     │
│  │    - Application images                        │     │
│  │    - Versioned releases                        │     │
│  └────────────────────────────────────────────────┘     │
│                                                         │
│  ┌────────────────────────────────────────────────┐     │
│  │         S3 (Terraform State)                   │     │
│  │    - Infrastructure state                      │     │
│  │    - Versioned and encrypted                   │     │
│  └────────────────────────────────────────────────┘     │
└─────────────────────────────────────────────────────────┘
```

## Repository Architecture

```
┌─────────────────────────────────────────────────────────┐
│  cloudnative-saas-eks (Configuration Repository)        │
│  ├── examples/dev-environment/                          │
│  │   ├── config/                                        │
│  │   │   ├── common.tfvars                              │
│  │   │   ├── infrastructure.tfvars                      │
│  │   │   └── tenants.tfvars                             │
│  │   ├── infrastructure/ (Terraform root module)        │
│  │   └── tenants/ (Terraform root module)               │
│  └── docs/                                              │
└─────────────────────────────────────────────────────────┘
                    ↓ calls
┌──────────────────────────────────────────────────────────┐
│  Terraform-modules (Module Library - GitHub)             │
│  ├── modules/vpc/                                        │
│  ├── modules/eks/                                        │
│  ├── modules/rds/                                        │
│  ├── modules/iam/                                        │
│  ├── modules/multi-tenancy/                              │
│  └── ...                                                 │
└──────────────────────────────────────────────────────────┘
                    ↓ deploys
┌─────────────────────────────────────────────────────────┐
│  Gitops-pipeline (GitOps Automation)               │
│  ├── argocd/applications/                               │
│  ├── apps/ (Kubernetes manifests)                       │
│  ├── .github/workflows/ (CI/CD)                         │
│  └── scripts/ (Deployment automation)                   │
└─────────────────────────────────────────────────────────┘
                    ↓ manages
┌─────────────────────────────────────────────────────────┐
│  AWS EKS Cluster                                        │
│  ├── Namespaces (platform, analytics, monitoring)       │
│  ├── Applications (deployed via ArgoCD)                │
│  └── Monitoring (Prometheus, Grafana)                  │
└─────────────────────────────────────────────────────────┘
```

## Component Architecture

### 1. Networking Layer (VPC)

**Components:**
- **VPC**: Custom VPC with configurable CIDR (default: 10.0.0.0/16)
- **Public Subnets**: 2+ subnets across availability zones
- **Private Subnets**: 2+ subnets across availability zones
- **Internet Gateway**: For public subnet internet access
- **NAT Gateways**: For private subnet outbound internet access
- **Route Tables**: Separate routes for public and private subnets
- **VPC Flow Logs**: Network traffic monitoring

**Design Decisions:**
- Multi-AZ deployment for high availability
- Private subnets for worker nodes (security)
- Public subnets for NAT gateways and load balancers
- VPC Flow Logs enabled for security monitoring

### 2. Compute Layer (EKS)

**Components:**
- **EKS Control Plane**: Managed Kubernetes control plane
- **Worker Node Groups**: Managed node groups (on-demand or spot)
- **Auto Scaling**: Cluster autoscaler for dynamic scaling
- **Container Runtime**: Docker/containerd

**Design Decisions:**
- Managed node groups for simplified management
- Auto-scaling based on demand
- Multi-AZ node distribution
- Instance types configurable per environment

### 3. Identity & Access (IAM)

**Components:**
- **Cluster IAM Role**: EKS cluster service role
- **Node IAM Role**: Worker node instance role
- **IRSA**: IAM Roles for Service Accounts
- **EKS Access Roles**: Admin, Developer, Viewer roles

**Design Decisions:**
- IRSA for pod-level AWS permissions
- Separate roles for different access levels
- Least-privilege principle
- Role-based access control (RBAC) in Kubernetes

### 4. Security Layer

**Components:**
- **Security Groups**: Network-level firewall rules
- **Network Policies**: Pod-to-pod communication rules
- **RBAC**: Kubernetes role-based access control
- **Encryption**: At rest and in transit
- **Secrets Management**: AWS Secrets Manager integration

**Design Decisions:**
- Defense in depth (multiple security layers)
- Network policies for tenant isolation
- RBAC for Kubernetes resource access
- Secrets Manager for credential management

### 5. Multi-Tenancy

**Components:**
- **Namespaces**: One per tenant
- **Resource Quotas**: CPU, memory, storage, pod limits
- **Network Policies**: Tenant traffic isolation
- **RBAC**: Namespace-level access control
- **Service Accounts**: Per-tenant service accounts with IRSA

**Design Decisions:**
- Namespace-based isolation (simpler than cluster-per-tenant)
- Resource quotas prevent resource exhaustion
- Network policies enforce traffic isolation
- Database isolation per tenant (separate schemas or databases)

### 6. Observability

**Components:**
- **Prometheus**: Metrics collection and storage
- **Grafana**: Visualization and dashboards
- **CloudWatch**: AWS resource monitoring
- **ServiceMonitors**: Automatic metric discovery
- **Node Exporter**: Node-level metrics
- **kube-state-metrics**: Kubernetes object metrics

**Design Decisions:**
- Prometheus for Kubernetes-native monitoring
- Grafana for visualization (popular, extensible)
- CloudWatch for AWS resource metrics
- ServiceMonitors for automatic discovery

## Data Flow

### Application Request Flow

```
User Request
    ↓
Internet
    ↓
AWS Load Balancer (ALB/NLB)
    ↓
Kubernetes Service (LoadBalancer type)
    ↓
Ingress Controller (if using Ingress)
    ↓
Application Pod (Frontend/Backend)
    ↓
Database (RDS PostgreSQL)
```

### Deployment Flow

```
Developer pushes code
    ↓
GitHub Actions triggered
    ↓
Build Docker images
    ↓
Push to ECR
    ↓
Update GitOps repository
    ↓
ArgoCD detects changes
    ↓
Syncs to Kubernetes cluster
    ↓
Application deployed/updated
```

### Monitoring Flow

```
Application Pods
    ↓ (expose /metrics)
Prometheus ServiceMonitor
    ↓ (scrapes metrics)
Prometheus
    ↓ (stores metrics)
Grafana
    ↓ (queries Prometheus)
Dashboards (visualization)
```

## Design Principles

### 1. Infrastructure as Code
- **Terraform** for all infrastructure
- **Version controlled** configurations
- **Reproducible** deployments
- **Environment parity** (dev/staging/prod)

### 2. GitOps
- **Declarative** configuration
- **Version controlled** application manifests
- **Automated** deployments
- **Audit trail** of all changes

### 3. Multi-Tenancy
- **Namespace isolation** per tenant
- **Resource quotas** for fairness
- **Network policies** for security
- **Database isolation** per tenant

### 4. Security
- **Defense in depth** (multiple layers)
- **Least privilege** access
- **Encryption** at rest and in transit
- **Secrets management** via AWS Secrets Manager

### 5. Observability
- **Metrics** collection (Prometheus)
- **Visualization** (Grafana)
- **Logging** (CloudWatch)
- **Alerting** (Alertmanager)

## Scalability Considerations

### Horizontal Scaling
- **Kubernetes HPA**: Pod autoscaling based on metrics
- **Cluster Autoscaler**: Node autoscaling based on demand
- **Database Read Replicas**: For read-heavy workloads

### Vertical Scaling
- **Resource Requests/Limits**: Per-pod resource allocation
- **Node Instance Types**: Larger instances for heavy workloads
- **Database Instance Types**: Scale up database as needed

### Multi-Region
- **Active-Active**: Deploy to multiple regions
- **Active-Passive**: Failover to secondary region
- **Global Load Balancer**: Route traffic to nearest region

## High Availability

### Infrastructure Level
- **Multi-AZ Deployment**: Resources across availability zones
- **Auto Scaling**: Automatic recovery from failures
- **Load Balancing**: Distribute traffic across instances

### Application Level
- **Pod Replicas**: Multiple pod instances
- **Health Checks**: Liveness and readiness probes
- **Rolling Updates**: Zero-downtime deployments

### Database Level
- **Multi-AZ RDS**: Automatic failover
- **Automated Backups**: Point-in-time recovery
- **Read Replicas**: For read scaling

## Next Steps

- **Read [05-Repository-Structure.md](05-Repository-Structure.md)** - Understand codebase organization
- **Review [06-Multi-Tenancy.md](06-Multi-Tenancy.md)** - Learn about multi-tenant design
- **Check [07-Deployment-Guide.md](07-Deployment-Guide.md)** - Detailed deployment process

---

**Want to deploy?** → [02-Quick-Start.md](02-Quick-Start.md)

