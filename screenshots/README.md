# Screenshots Directory

This directory contains all screenshots for the platform documentation.

## 📁 File Organization

Screenshots are organized by category and referenced in `../Screenshots.md`:

### Database Screenshots
- `database-connection.png` - RDS connection verification
- `database-tenant-schemas.png` - Multi-tenant schema structure
- `database-tables.png` - Database tables
- `database-sample-data.png` - Sample data
- `database-tenants-table.png` - Tenants table query results
- `database-platform-users.png` - Platform tenant users
- `database-analytics-users.png` - Analytics tenant users

### Kubernetes Screenshots
- `k8s-pods.png` - Pod status
- `k8s-namespaces.png` - Namespaces and resources
- `k8s-services.png` - Services
- `k8s-deployments.png` - Deployments
- `k8s-verification-output.png` - Complete verification output

### Application Screenshots
- `app-frontend-homepage.png` - Frontend application
- `app-backend-api.png` - Backend API
- `app-loadbalancer.png` - LoadBalancer service
- `app-platform-dashboard.png` - Platform tenant dashboard
- `app-analytics-dashboard.png` - Analytics tenant dashboard

### Grafana Screenshots
- `grafana-sample-app-dashboard.png` - Sample App Dashboard
- `grafana-node-exporter.png` - Node Exporter Dashboard

### ArgoCD Screenshots
- `argocd-applications.png` - ArgoCD applications list
- `argocd-application-details.png` - Application details
- `argocd-ui.png` - ArgoCD UI

### AWS Screenshots
- `aws-eks-cluster.png` - EKS cluster
- `aws-rds-instance.png` - RDS instance
- `aws-ecr-repositories.png` - ECR repositories
- `aws-secrets-manager.png` - Secrets Manager
- `terraform-outputs.png` - Terraform infrastructure outputs

### Monitoring Screenshots
- `prometheus-targets.png` - Prometheus targets
- `prometheus-queries.png` - Prometheus queries
- `cloudwatch-logs.png` - CloudWatch logs

### CI/CD Screenshots
- `github-actions-workflow.png` - GitHub Actions workflow
- `docker-build.png` - Docker build
- `deployment-success.png` - Deployment success message

## 📝 Adding New Screenshots

1. **Save the screenshot** in this directory with a descriptive name
2. **Update `../Screenshots.md`** to reference the new screenshot
3. **Follow naming convention**: `category-description.png` (lowercase, hyphens)
4. **Keep file sizes reasonable**: Compress large images if needed
5. **Remove sensitive data**: Blur or redact credentials, IPs, tokens before committing

## 🔒 Security Note

**Before committing screenshots:**
- ✅ Remove or blur sensitive information (credentials, tokens, IPs)
- ✅ Redact AWS account IDs
- ✅ Remove personal data
- ✅ Check for exposed secrets in logs or UI

## 📏 Image Guidelines

- **Format**: PNG preferred (supports transparency)
- **Resolution**: 1920x1080 or higher for clarity
- **File Size**: Keep under 2MB per image when possible
- **Naming**: Use lowercase, hyphens, descriptive names

---

**Last Updated**: 2025-12-12

