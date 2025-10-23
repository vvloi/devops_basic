# Phase 4 - CI/CD Pipeline

This phase implements Continuous Integration and Continuous Deployment pipelines using GitHub Actions and ArgoCD.

## Overview

Phase 4 provides automated build, test, and deployment workflows for:
- **Spring Boot Services**: Automated CI/CD pipeline for microservices
- **Libraries**: Build and publish to Nexus repository

## Folders

### github-actions
Contains reference copies of GitHub Actions workflow definitions for CI/CD pipelines:
- `spring-boot-service.yml`: CI/CD workflow for Spring Boot applications
- `library-publish.yml`: CI pipeline for building and publishing libraries to Nexus

**Note**: The actual workflows are located in `.github/workflows/` directory at the repository root, which is where GitHub Actions automatically discovers and runs them.

### argocd
ArgoCD configurations for GitOps-based continuous deployment:
- `applications/`: ArgoCD Application manifests
- `projects/`: ArgoCD Project definitions

### k8s-manifests
Kubernetes manifests for deployed services:
- `spring-boot-service/`: Kubernetes resources for Spring Boot applications

## Prerequisites

### For GitHub Actions
1. **GitHub Secrets** (configure in repository settings):
   - `DOCKER_USERNAME`: Docker Hub username
   - `DOCKER_PASSWORD`: Docker Hub password/token
   - `NEXUS_USERNAME`: Nexus repository username
   - `NEXUS_PASSWORD`: Nexus repository password
   - `ARGOCD_TOKEN`: ArgoCD API token (for automated sync)

### For ArgoCD
1. **ArgoCD Installation**: ArgoCD must be installed in your Kubernetes cluster
2. **Repository Access**: ArgoCD needs access to your Git repository
3. **Cluster Configuration**: Target Kubernetes cluster must be configured in ArgoCD

## Usage

### Spring Boot Service CI/CD
1. Push code changes to trigger the workflow
2. GitHub Actions will:
   - Build and test the application
   - Build Docker image
   - Push to Docker registry
   - Update image tag in Kubernetes manifests
3. ArgoCD will automatically detect changes and deploy

### Library Publishing
1. Tag your library release (e.g., `v1.0.0`)
2. GitHub Actions will:
   - Build the library
   - Run tests
   - Publish to Nexus repository

## Workflow Triggers

- **Spring Boot Service**: Triggers on push to `main` branch and pull requests
- **Library Publishing**: Triggers on version tags (e.g., `v*.*.*`)

## ArgoCD Setup

### Installing ArgoCD
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### Accessing ArgoCD UI
```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

### Get Initial Admin Password
```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

## Directory Structure
```
phase 4/
├── README.md
├── github-actions/
│   ├── spring-boot-service.yml
│   └── library-publish.yml
├── argocd/
│   ├── applications/
│   │   └── spring-boot-service.yaml
│   └── projects/
│       └── devops-project.yaml
└── k8s-manifests/
    └── spring-boot-service/
        ├── deployment.yaml
        ├── service.yaml
        └── configmap.yaml
```

## Notes

- The workflows are designed to be template examples that can be customized for specific applications
- Ensure all required secrets are configured before running workflows
- ArgoCD sync policies can be adjusted based on your deployment strategy
- Consider implementing blue-green or canary deployment strategies for production
