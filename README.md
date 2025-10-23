# devops_basic
Basically convert docker-compose file to k8s and automatically CI/CD

## Repository Structure

This repository is organized into phases representing the DevOps workflow progression:

### Phase 1: Core Infrastructure and Common Tools
Contains the foundational infrastructure components and common utilities needed for the DevOps environment.

### Phase 2: Kong Gateway Implementation
Includes Kong Gateway setup with OpenAPI specifications and routing configurations (HTTP and TCP).

### Phase 3: Domain and Infrastructure Setup
Covers domain purchase, Cloudflare configuration, and server rental documentation.

### Phase 4: CI/CD Pipeline
Implements automated build, test, and deployment pipelines:
- **GitHub Actions**: CI/CD workflows for Spring Boot services and library publishing
- **ArgoCD**: GitOps-based continuous deployment configurations
- **Kubernetes Manifests**: Production-ready K8s resources with security best practices

## Getting Started

### Prerequisites
- Kubernetes cluster
- GitHub account with repository access
- Docker Hub account
- ArgoCD (for CD)
- Nexus repository (for library publishing)

### Quick Start
1. Clone this repository
2. Follow the setup guide in each phase directory
3. For CI/CD setup, see [Phase 4 Setup Guide](phase%204/SETUP.md)

## CI/CD Overview

This repository uses:
- **GitHub Actions** for Continuous Integration
  - Automated building and testing
  - Docker image creation and publishing
  - Library publishing to Nexus
- **ArgoCD** for Continuous Deployment
  - GitOps-based deployment
  - Automated sync from Git repository
  - Production-ready Kubernetes deployments

## Documentation

Each phase contains detailed documentation:
- Phase-specific README files
- Setup guides
- Configuration examples

For detailed CI/CD setup instructions, see [Phase 4/SETUP.md](phase%204/SETUP.md)
