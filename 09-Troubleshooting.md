# Troubleshooting Guide

Common issues and their solutions for the CloudNative SaaS Platform.

## Table of Contents

- [Infrastructure Issues](#infrastructure-issues)
- [Kubernetes Issues](#kubernetes-issues)
- [Application Issues](#application-issues)
- [GitOps/ArgoCD Issues](#gitopsargocd-issues)
- [Database Issues](#database-issues)
- [Monitoring Issues](#monitoring-issues)
- [Network Issues](#network-issues)

## Infrastructure Issues

### Terraform Apply Fails

**Symptoms:**
- `terraform apply` fails with errors
- Resources partially created

**Solutions:**

1. **Check AWS Credentials:**
```bash
aws sts get-caller-identity
# Should return your account ID
```

2. **Check Permissions:**
```bash
# Verify you have required permissions
aws iam get-user
```

3. **Check State File:**
```bash
# Verify backend configuration
terraform init -backend-config=../config/infrastructure/backend-dev.tfbackend
```

4. **Common Errors:**
   - **"Access Denied"**: Check IAM permissions
   - **"Resource already exists"**: Import existing resource or use different name
   - **"InvalidParameter"**: Check variable values in `.tfvars` files

### EKS Cluster Not Accessible

**Symptoms:**
- `kubectl get nodes` fails
- Connection timeout errors

**Solutions:**

1. **Update kubeconfig:**
```bash
CLUSTER_NAME=$(terraform output -raw cluster_name)
AWS_REGION=$(terraform output -raw aws_region)
aws eks update-kubeconfig --name $CLUSTER_NAME --region $AWS_REGION
```

2. **Check Cluster Status:**
```bash
aws eks describe-cluster --name $CLUSTER_NAME --region $AWS_REGION
```

3. **Verify Security Groups:**
```bash
# Check if your IP is allowed
aws eks describe-cluster --name $CLUSTER_NAME --query 'cluster.resourcesVpcConfig'
```

### RDS Connection Issues

**Symptoms:**
- Applications can't connect to database
- Connection timeout errors

**Solutions:**

1. **Check Security Group Rules:**
```bash
# Get RDS security group
RDS_SG=$(aws rds describe-db-instances --query 'DBInstances[0].VpcSecurityGroups[0].VpcSecurityGroupId' --output text)

# Check ingress rules
aws ec2 describe-security-groups --group-ids $RDS_SG
```

2. **Verify EKS Security Group is Allowed:**
   - RDS security group must allow traffic from:
     - EKS cluster security group
     - EKS node security group

3. **Check Database Endpoint:**
```bash
# Get RDS endpoint
aws rds describe-db-instances --query 'DBInstances[0].Endpoint.Address' --output text
```

4. **Test Connection:**
```bash
# From a pod in the cluster
kubectl run -it --rm --image=postgres:15-alpine test-connection -- sh
# Then: psql -h <rds-endpoint> -U <username> -d <database>
```

## Kubernetes Issues

### Pods Not Starting

**Symptoms:**
- Pods stuck in `Pending` or `CrashLoopBackOff`
- Pods not ready

**Solutions:**

1. **Check Pod Status:**
```bash
kubectl get pods -n <namespace>
kubectl describe pod <pod-name> -n <namespace>
```

2. **Check Events:**
```bash
kubectl get events -n <namespace> --sort-by='.lastTimestamp'
```

3. **Common Issues:**

   **ImagePullBackOff:**
   ```bash
   # Check image name and tag
   kubectl describe pod <pod-name> -n <namespace> | grep Image
   
   # Verify ECR access
   aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <ecr-repo>
   ```

   **Pending (Resource Quota):**
   ```bash
   # Check resource quotas
   kubectl get resourcequota -n <namespace>
   
   # Check available resources
   kubectl describe namespace <namespace>
   ```

   **CrashLoopBackOff:**
   ```bash
   # Check pod logs
   kubectl logs <pod-name> -n <namespace>
   
   # Check previous container logs
   kubectl logs <pod-name> -n <namespace> --previous
   ```

### Services Not Accessible

**Symptoms:**
- Can't access application via LoadBalancer URL
- Service shows `<pending>` external IP

**Solutions:**

1. **Check Service:**
```bash
kubectl get svc -n <namespace>
kubectl describe svc <service-name> -n <namespace>
```

2. **Check LoadBalancer:**
```bash
# Get LoadBalancer ARN
kubectl get svc <service-name> -n <namespace> -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'

# Check AWS LoadBalancer
aws elbv2 describe-load-balancers --region us-east-1
```

3. **Wait for Provisioning:**
   - AWS LoadBalancers take 2-5 minutes to provision
   - Check again after waiting

4. **Check Security Groups:**
   - LoadBalancer security group must allow traffic on service port

### Resource Quota Exceeded

**Symptoms:**
- New pods can't be created
- Error: "exceeded quota"

**Solutions:**

1. **Check Quotas:**
```bash
kubectl get resourcequota -n <namespace>
kubectl describe resourcequota -n <namespace>
```

2. **Check Usage:**
```bash
kubectl top pods -n <namespace>
kubectl top nodes
```

3. **Increase Quota:**
   - Edit tenant configuration in `tenants.tfvars`
   - Re-run `terraform apply` in tenants directory

## Application Issues

### Application Not Responding

**Symptoms:**
- HTTP 502/503 errors
- Timeout errors

**Solutions:**

1. **Check Pod Status:**
```bash
kubectl get pods -n <namespace>
kubectl logs <pod-name> -n <namespace>
```

2. **Check Service:**
```bash
kubectl get svc -n <namespace>
kubectl describe svc <service-name> -n <namespace>
```

3. **Test from Inside Cluster:**
```bash
# Port forward to pod
kubectl port-forward <pod-name> -n <namespace> 8080:3000

# Test locally
curl http://localhost:8080/health
```

4. **Check Application Logs:**
```bash
kubectl logs -f <pod-name> -n <namespace>
```

### Database Connection Errors

**Symptoms:**
- "Connection refused" errors
- "Authentication failed" errors

**Solutions:**

1. **Check Secret:**
```bash
kubectl get secret -n <namespace>
kubectl describe secret <secret-name> -n <namespace>
```

2. **Verify Secret Values:**
```bash
kubectl get secret <secret-name> -n <namespace> -o jsonpath='{.data.db-password}' | base64 -d
```

3. **Check Environment Variables:**
```bash
kubectl exec <pod-name> -n <namespace> -- env | grep DB_
```

4. **Test Database Connection:**
```bash
# Create a test pod
kubectl run -it --rm --image=postgres:15-alpine test-db -- sh

# Inside pod:
psql -h <db-host> -U <db-user> -d <db-name>
```

## GitOps/ArgoCD Issues

### ArgoCD Sync Failing

**Symptoms:**
- Application shows `OutOfSync` or `Degraded`
- Sync errors in ArgoCD UI

**Solutions:**

1. **Check Application Status:**
```bash
kubectl get application -n argocd
kubectl describe application <app-name> -n argocd
```

2. **Check Sync Status:**
```bash
# View sync operation
kubectl get application <app-name> -n argocd -o yaml | grep -A 20 operationState
```

3. **Common Issues:**

   **CRD Annotation Size Limit:**
   ```bash
   # Manually apply CRDs with server-side apply
   kubectl apply --server-side -f <crd-url>
   ```

   **Image Pull Errors:**
   - Check ECR repository access
   - Verify image tags in manifests
   - Check image pull secrets

   **Resource Conflicts:**
   - Check for existing resources with same name
   - Use `kubectl get all -A` to find conflicts

4. **Manual Sync:**
   - Use ArgoCD UI: Click "Sync" button
   - Or CLI: `argocd app sync <app-name>`

### ArgoCD Not Accessible

**Symptoms:**
- Can't access ArgoCD UI
- Connection refused

**Solutions:**

1. **Check ArgoCD Pods:**
```bash
kubectl get pods -n argocd
kubectl logs -n argocd deployment/argocd-server
```

2. **Port Forward:**
```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
# Access: https://localhost:8080
```

3. **Get Admin Password:**
```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d && echo
```

## Database Issues

### Database Initialization Fails

**Symptoms:**
- Init job fails
- Tables not created

**Solutions:**

1. **Check Init Job:**
```bash
kubectl get jobs -n <namespace>
kubectl logs job/<init-job-name> -n <namespace>
```

2. **Check Database Connection:**
```bash
# Verify database credentials
kubectl get secret <db-secret> -n <namespace> -o yaml
```

3. **Manual Database Init:**
```bash
# Connect to database
kubectl run -it --rm --image=postgres:15-alpine db-client -- sh

# Run migrations manually
psql -h <db-host> -U <db-user> -d <db-name> -f /path/to/migration.sql
```

### Database Performance Issues

**Symptoms:**
- Slow queries
- Connection timeouts

**Solutions:**

1. **Check Database Metrics:**
   - Use CloudWatch for RDS metrics
   - Check CPU, memory, connection count

2. **Check Connections:**
```sql
-- Connect to database and check
SELECT count(*) FROM pg_stat_activity;
```

3. **Optimize:**
   - Increase instance size
   - Add read replicas
   - Optimize queries
   - Add connection pooling

## Monitoring Issues

### Grafana Not Accessible

**Symptoms:**
- Can't access Grafana UI
- Service not found

**Solutions:**

1. **Check Service:**
```bash
kubectl get svc -n monitoring monitoring-stack-grafana
```

2. **Check Pods:**
```bash
kubectl get pods -n monitoring -l app.kubernetes.io/name=grafana
```

3. **Get LoadBalancer URL:**
```bash
kubectl get svc -n monitoring monitoring-stack-grafana -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'
```

4. **Port Forward (temporary):**
```bash
kubectl port-forward -n monitoring svc/monitoring-stack-grafana 3000:80
```

### Prometheus Not Scraping

**Symptoms:**
- No metrics in Grafana
- Targets showing as down

**Solutions:**

1. **Check ServiceMonitors:**
```bash
kubectl get servicemonitor -A
kubectl describe servicemonitor <name> -n <namespace>
```

2. **Check Prometheus Targets:**
```bash
# Port forward to Prometheus
kubectl port-forward -n monitoring svc/prometheus-operated 9090:9090
# Visit: http://localhost:9090/targets
```

3. **Verify Service Labels:**
   - ServiceMonitor selector must match service labels
   - Check `release: prometheus` label

## Network Issues

### Pods Can't Communicate

**Symptoms:**
- Pods can't reach each other
- Network policy blocking traffic

**Solutions:**

1. **Check Network Policies:**
```bash
kubectl get networkpolicies -A
kubectl describe networkpolicy <name> -n <namespace>
```

2. **Test Connectivity:**
```bash
# From source pod
kubectl exec <pod-name> -n <namespace> -- curl <target-service>.<namespace>.svc.cluster.local
```

3. **Check Security Groups:**
   - Verify security group rules allow traffic
   - Check both source and destination security groups

### LoadBalancer Not Working

**Symptoms:**
- External IP pending
- Can't access via LoadBalancer URL

**Solutions:**

1. **Check LoadBalancer Status:**
```bash
kubectl get svc <service-name> -n <namespace>
```

2. **Check AWS LoadBalancer:**
```bash
aws elbv2 describe-load-balancers --region us-east-1
```

3. **Wait for Provisioning:**
   - LoadBalancers take 2-5 minutes
   - Check DNS propagation

4. **Check Annotations:**
   - Verify service annotations for LoadBalancer type
   - Check for AWS-specific annotations

## Getting Help

If you're still experiencing issues:

1. **Check Logs:**
   - Pod logs: `kubectl logs <pod-name> -n <namespace>`
   - Events: `kubectl get events -n <namespace>`
   - ArgoCD logs: `kubectl logs -n argocd deployment/argocd-server`

2. **Review Documentation:**
   - [04-Architecture.md](04-Architecture.md) - Understand the architecture
   - [07-Deployment-Guide.md](07-Deployment-Guide.md) - Deployment process
   - [11-Monitoring.md](11-Monitoring.md) - Monitoring setup

3. **Common Commands:**
```bash
# Get all resources
kubectl get all -A

# Describe resource
kubectl describe <resource-type> <name> -n <namespace>

# Check events
kubectl get events -A --sort-by='.lastTimestamp'

# Check logs
kubectl logs -f <pod-name> -n <namespace>
```

---

**Still stuck?** Review the architecture in [04-Architecture.md](04-Architecture.md) or check the deployment guide in [07-Deployment-Guide.md](07-Deployment-Guide.md)

