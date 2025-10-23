# Phase 4 Setup Guide

This guide provides step-by-step instructions for setting up CI/CD pipelines using GitHub Actions and ArgoCD.

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [GitHub Actions Setup](#github-actions-setup)
3. [ArgoCD Setup](#argocd-setup)
4. [Application Deployment](#application-deployment)
5. [Testing and Verification](#testing-and-verification)
6. [Troubleshooting](#troubleshooting)

## Prerequisites

### Required Tools
- **Kubernetes Cluster**: A running Kubernetes cluster (minikube, kind, GKE, EKS, AKS, etc.)
- **kubectl**: Kubernetes CLI tool
- **Docker**: For building and pushing images
- **Git**: Version control
- **ArgoCD CLI** (optional): For managing ArgoCD from command line

### Required Accounts
- **GitHub Account**: With repository access
- **Docker Hub Account**: For storing Docker images
- **Nexus Repository**: For library artifacts (optional)

## GitHub Actions Setup

### Step 1: Configure GitHub Secrets

Navigate to your GitHub repository settings and add the following secrets:

#### For Spring Boot Service
```
Settings → Secrets and variables → Actions → New repository secret
```

Add these secrets:
- **DOCKER_USERNAME**: Your Docker Hub username
- **DOCKER_PASSWORD**: Your Docker Hub password or access token
- **ARGOCD_TOKEN**: ArgoCD API token (optional, for automated sync)

#### For Library Publishing
Add these additional secrets:
- **NEXUS_USERNAME**: Nexus repository username
- **NEXUS_PASSWORD**: Nexus repository password

### Step 2: Update Workflow Configuration

#### For Spring Boot Service
Edit `.github/workflows/spring-boot-service.yml`:

1. Update the `IMAGE_NAME` in the env section:
```yaml
env:
  IMAGE_NAME: your-dockerhub-username/your-service-name
```

2. Update the paths to match your Spring Boot application location:
```yaml
paths:
  - 'your-spring-boot-app-path/**'
```

3. Update the ArgoCD URL in the sync job:
```yaml
https://your-argocd-instance.com/api/v1/applications/spring-boot-service/sync
```

#### For Library Publishing
Edit `.github/workflows/library-publish.yml`:

1. Update the `NEXUS_URL`:
```yaml
env:
  NEXUS_URL: 'https://your-nexus-instance.com'
```

2. Update paths to match your library location:
```yaml
cd your-library-path
```

### Step 3: Test GitHub Actions

1. **Push to main branch** to trigger the Spring Boot service workflow
2. **Create a version tag** to trigger the library publishing workflow:
```bash
git tag -a v1.0.0 -m "Release version 1.0.0"
git push origin v1.0.0
```

## ArgoCD Setup

### Step 1: Install ArgoCD

```bash
# Create ArgoCD namespace
kubectl create namespace argocd

# Install ArgoCD
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Wait for ArgoCD to be ready
kubectl wait --for=condition=available --timeout=300s deployment/argocd-server -n argocd
```

### Step 2: Access ArgoCD UI

#### Option 1: Port Forwarding
```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```
Access at: https://localhost:8080

#### Option 2: Expose via LoadBalancer
```bash
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```

#### Option 3: Expose via Ingress
Create an Ingress resource for ArgoCD (recommended for production).

### Step 3: Get Initial Admin Password

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d && echo
```

Login with:
- Username: `admin`
- Password: (output from above command)

### Step 4: Install ArgoCD CLI (Optional)

#### Linux/macOS
```bash
curl -sSL -o argocd https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
chmod +x argocd
sudo mv argocd /usr/local/bin/
```

#### Login via CLI
```bash
argocd login localhost:8080 --username admin --password <password> --insecure
```

### Step 5: Apply ArgoCD Project

```bash
kubectl apply -f phase\ 4/argocd/projects/devops-project.yaml
```

Verify:
```bash
kubectl get appproject -n argocd
```

### Step 6: Apply ArgoCD Application

```bash
kubectl apply -f phase\ 4/argocd/applications/spring-boot-service.yaml
```

Verify:
```bash
kubectl get applications -n argocd
```

Or via ArgoCD CLI:
```bash
argocd app list
```

## Application Deployment

### Step 1: Create Application Namespace

```bash
kubectl create namespace spring-boot-service
```

### Step 2: Create Secrets (if needed)

```bash
kubectl create secret generic spring-boot-service-secrets \
  --from-literal=database.password=yourpassword \
  --from-literal=api.key=yourapikey \
  -n spring-boot-service
```

### Step 3: Sync ArgoCD Application

#### Via UI
1. Open ArgoCD UI
2. Find `spring-boot-service` application
3. Click "Sync" button
4. Select sync options and confirm

#### Via CLI
```bash
argocd app sync spring-boot-service
```

### Step 4: Monitor Deployment

#### Via ArgoCD UI
- Watch the application sync status
- View resource tree
- Check pod logs

#### Via kubectl
```bash
# Watch deployment
kubectl get deployments -n spring-boot-service -w

# Check pods
kubectl get pods -n spring-boot-service

# View logs
kubectl logs -f deployment/spring-boot-service -n spring-boot-service
```

## Testing and Verification

### Verify Application Health

```bash
# Port forward to service
kubectl port-forward svc/spring-boot-service -n spring-boot-service 8080:80

# Check health endpoint
curl http://localhost:8080/actuator/health
```

### Verify ArgoCD Sync

```bash
# Check application status
argocd app get spring-boot-service

# View sync history
argocd app history spring-boot-service
```

### Verify GitHub Actions

1. Go to GitHub repository → Actions tab
2. Check workflow runs
3. Review logs for any failures

### Verify Library in Nexus

After publishing a library:
```bash
curl -u username:password \
  https://your-nexus-instance.com/repository/maven-releases/com/example/library/1.0.0/library-1.0.0.jar
```

## Troubleshooting

### GitHub Actions Issues

#### Workflow not triggering
- Check if paths are correct in workflow triggers
- Verify branch names match workflow configuration
- Ensure workflows are enabled in repository settings

#### Docker push fails
- Verify DOCKER_USERNAME and DOCKER_PASSWORD secrets are set correctly
- Check Docker Hub account permissions
- Ensure image name format is correct

#### Nexus publish fails
- Verify NEXUS_USERNAME and NEXUS_PASSWORD are correct
- Check Nexus repository permissions
- Ensure NEXUS_URL is accessible from GitHub Actions runners

### ArgoCD Issues

#### Application not syncing
- Check if ArgoCD has access to Git repository
- Verify path in Application manifest is correct
- Check ArgoCD server logs: `kubectl logs -n argocd deployment/argocd-server`

#### Sync fails with permission errors
- Check AppProject permissions
- Verify service account has necessary RBAC permissions
- Review ArgoCD application controller logs

#### Resources not appearing
- Verify Kubernetes manifests are valid: `kubectl apply --dry-run=client -f manifest.yaml`
- Check namespace exists
- Review resource events: `kubectl describe -n spring-boot-service deployment/spring-boot-service`

### Application Issues

#### Pods not starting
- Check pod logs: `kubectl logs -n spring-boot-service pod-name`
- Describe pod: `kubectl describe pod -n spring-boot-service pod-name`
- Verify image exists in Docker registry
- Check resource requests and limits

#### Service not accessible
- Verify service exists: `kubectl get svc -n spring-boot-service`
- Check endpoints: `kubectl get endpoints -n spring-boot-service`
- Test service connectivity: `kubectl run -it --rm debug --image=busybox --restart=Never -- wget -O- http://spring-boot-service.spring-boot-service.svc.cluster.local`

#### Health checks failing
- Verify health check endpoints are correct
- Check initialDelaySeconds is sufficient for app startup
- Review application logs for startup issues

## Advanced Configuration

### Enable Auto-Sync in ArgoCD

Edit the Application manifest or use CLI:
```bash
argocd app set spring-boot-service --sync-policy automated
```

### Configure Sync Waves

Add annotations to Kubernetes resources for ordered deployment:
```yaml
metadata:
  annotations:
    argocd.argoproj.io/sync-wave: "1"
```

### Setup Notifications

Configure ArgoCD to send notifications on sync events:
1. Install ArgoCD Notifications
2. Configure notification services (Slack, email, etc.)
3. Define triggers and templates

### Implement Blue-Green Deployment

Modify deployment strategy in manifest:
```yaml
spec:
  strategy:
    blueGreen:
      activeService: spring-boot-service
      previewService: spring-boot-service-preview
```

## Best Practices

1. **Use semantic versioning** for library releases
2. **Keep secrets encrypted** - never commit secrets to Git
3. **Implement proper health checks** for reliable deployments
4. **Use resource limits** to prevent resource exhaustion
5. **Enable monitoring and logging** for observability
6. **Implement RBAC** for access control
7. **Use NetworkPolicies** for network segmentation
8. **Regular updates** of base images and dependencies
9. **Test in staging** before production deployment
10. **Implement rollback procedures** for quick recovery

## Additional Resources

- [ArgoCD Documentation](https://argo-cd.readthedocs.io/)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Spring Boot Documentation](https://spring.io/projects/spring-boot)
- [Nexus Repository Documentation](https://help.sonatype.com/repomanager3)
