# Multi-Tenancy Architecture

Complete guide to the multi-tenant architecture and isolation mechanisms.

## Overview

The platform uses **namespace-based multi-tenancy** where each tenant gets:
- Isolated Kubernetes namespace
- Resource quotas (CPU, memory, storage, pods)
- Network policies for traffic isolation
- RBAC for access control
- Database schema isolation

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│  Kubernetes Cluster                                      │
│                                                           │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │
│  │  Platform     │  │  Analytics   │  │  Monitoring  │   │
│  │  Namespace    │  │  Namespace    │  │  Namespace    │   │
│  │               │  │               │  │               │   │
│  │  ┌─────────┐  │  │  ┌─────────┐  │  │  ┌─────────┐  │   │
│  │  │ Pods    │  │  │  │ Pods    │  │  │  │ Pods    │  │   │
│  │  │ Services│  │  │  │ Services│  │  │  │ Services│  │   │
│  │  │ Secrets │  │  │  │ Secrets │  │  │  │ Secrets │  │   │
│  │  └─────────┘  │  │  └─────────┘  │  │  └─────────┘  │   │
│  │               │  │               │  │               │   │
│  │  Resource     │  │  Resource     │  │  Resource     │   │
│  │  Quota        │  │  Quota        │  │  Quota        │   │
│  │  Network      │  │  Network      │  │  Network      │   │
│  │  Policy       │  │  Policy       │  │  Policy       │   │
│  └──────────────┘  └──────────────┘  └──────────────┘   │
└─────────────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────────────┐
│  RDS PostgreSQL (Multi-tenant Database)                 │
│                                                           │
│  ┌──────────────┐  ┌──────────────┐                      │
│  │ tenant_      │  │ tenant_      │                      │
│  │ platform     │  │ analytics   │                      │
│  │ schema       │  │ schema      │                      │
│  └──────────────┘  └──────────────┘                      │
└─────────────────────────────────────────────────────────┘
```

## Isolation Mechanisms

### 1. Namespace Isolation

Each tenant has a dedicated Kubernetes namespace:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: platform
  labels:
    tenant: platform
```

**Benefits**:
- Logical separation of resources
- Easy resource management
- Simplified access control

### 2. Resource Quotas

Prevent resource exhaustion:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: platform-quota
  namespace: platform
spec:
  hard:
    requests.cpu: "4"
    requests.memory: 8Gi
    limits.cpu: "8"
    limits.memory: 16Gi
    persistentvolumeclaims: "10"
    pods: "50"
```

### 3. Network Policies

Control pod-to-pod communication:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
  namespace: platform
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

### 4. Database Schema Isolation

Each tenant has a separate database schema:

```sql
-- Platform tenant schema
CREATE SCHEMA tenant_platform;

-- Analytics tenant schema
CREATE SCHEMA tenant_analytics;
```

## Tenant Configuration

### Default Tenants

| Tenant | Namespace | CPU Limit | Memory Limit | Purpose |
|--------|-----------|-----------|-------------|---------|
| Platform | `platform` | 4 cores | 8Gi | Core services |
| Analytics | `analytics` | 2 cores | 4Gi | Analytics workloads |

### Adding New Tenants

To add new tenants, edit `cloudnative-saas-eks/examples/dev-environment/config/tenants.tfvars` and add a new tenant configuration, then run `terraform apply` in the tenants directory.

## Isolation Levels

### Level 1: Namespace Isolation
- Resources isolated by namespace
- RBAC at namespace level
- Resource quotas per namespace

### Level 2: Network Isolation
- Network policies prevent cross-tenant traffic
- Default deny policies
- Explicit allow rules

### Level 3: Database Isolation
- Separate schemas per tenant
- Schema-level access control
- Data isolation at database level

## Best Practices

1. **Always use namespaces** for tenant separation
2. **Set resource quotas** to prevent resource exhaustion
3. **Enforce network policies** for security
4. **Use schema isolation** in databases
5. **Monitor resource usage** per tenant

## Next Steps

- **Read [07-Deployment-Guide.md](07-Deployment-Guide.md)** - Complete deployment process
- **Review [04-Architecture.md](04-Architecture.md)** - System architecture
- **Check [09-Troubleshooting.md](09-Troubleshooting.md)** - Common issues

---

**Want to deploy?** → [02-Quick-Start.md](02-Quick-Start.md)

