# gh-newops

A GitOps infrastructure repository for managing Kubernetes deployments across multiple environments using ArgoCD and Helm.

## 📋 Overview

**gh-newops** is a modern GitOps repository that automates the deployment and management of Spring Boot microservices across multiple Kubernetes environments (dev, int, preprod, prod). It leverages **ArgoCD** for continuous deployment and **Helm** for package management.

## 🏗️ Architecture

### Services

The repository manages the following microservices:

- **gh-config-server** - Centralized configuration server
- **gh-eureka-server** - Service registry and discovery server
- **gh-gateway** - API Gateway service
- **gh-simple-spring** - Simple Spring Boot application
- **gh-another-spring** - Additional Spring Boot application

### Environments

Deployments are organized across four environments:

- **dev** - Development environment for testing new features
- **int** - Integration environment for integration testing
- **preprod** - Pre-production environment for staging
- **prod** - Production environment

### Cluster

- **homelab994** - Target Kubernetes cluster with environment-specific configurations

## 🚀 Getting Started

### Prerequisites

- Kubernetes cluster (v1.18+)
- ArgoCD installed on the cluster
- Helm 3.x
- kubectl configured with access to the cluster

### Setup

1. **Clone the repository:**
   ```bash
   git clone https://github.com/homelab994/gh-newops.git
   cd gh-newops
   ```

2. **Create the target namespace (if not exists):**
   ```bash
   kubectl create namespace dev
   kubectl create namespace int
   kubectl create namespace preprod
   kubectl create namespace prod
   ```


## 🔄 GitOps Workflow

This repository uses **ArgoCD** with the following setup:

### Root Application
Each environment has a root application (`root-app.yaml`) that serves as the entry point for ArgoCD synchronization.

### ApplicationSet
Each environment uses an `ApplicationSet` (`applicationset.yaml`) that:
- Discovers Helm charts in the `charts/` directory
- Dynamically creates Argo Applications for each chart
- Applies environment-specific values from `environments/<env>/<chart>-values.yaml`
- Automatically syncs and heals deployments

**Example flow:**
```
ApplicationSet (dev) 
  └─> Discovers: gh-config-server, gh-eureka-server, gh-gateway, gh-simple-spring, gh-another-spring
  └─> Creates Application for each service
      └─> Uses: charts/gh-config-server/ + environments/dev/gh-config-server-values.yaml
```

### Sync Policy

All applications are configured with:
- **Automated sync** - Changes are automatically synchronized
- **Prune** - Removes resources that are no longer in the repository
- **Self-healing** - Automatically corrects drift from the desired state

## 📝 Configuration

### Adding a New Service

1. **Create a Helm chart:**
   ```bash
   helm create charts/my-new-service
   ```

2. **Create environment-specific values:**
   ```bash
   touch environments/dev/my-new-service-values.yaml
   touch environments/int/my-new-service-values.yaml
   touch environments/preprod/my-new-service-values.yaml
   touch environments/prod/my-new-service-values.yaml
   ```

3. **Update platform-config/services.yaml:**
   ```yaml
   services:
     my-new-service:
       type: k8s
   ```

4. **Commit and push** - ArgoCD will automatically detect and deploy the new service

### Customizing Environment Values

Edit the appropriate values file in `environments/<env>/` to customize service deployments:

```bash
nano environments/dev/gh-gateway-values.yaml
```

## 🔐 Access Control

ArgoCD project: `homelab994`

Ensure your cluster is configured with appropriate RBAC policies for the ArgoCD service account.

## 📊 Monitoring Deployments

### Check Application Status

```bash
# View all applications in an environment
kubectl get applications -n argocd -l environment=dev

# Get detailed status of a specific application
kubectl get application gh-gateway-dev -n argocd

# Watch ArgoCD sync status
argocd app list
argocd app get gh-gateway-dev
```

### View Logs

```bash
# Check ArgoCD controller logs
kubectl logs -n argocd -l app.kubernetes.io/name=argocd-application-controller -f
```

## 🔗 Repository Information

- **Repository:** https://github.com/homelab994/gh-newops
- **Cluster Target:** https://kubernetes.default.svc (in-cluster)
- **Namespaces:** dev, int, preprod, prod

## 📚 Additional Resources

- [ArgoCD Documentation](https://argo-cd.readthedocs.io/)
- [Helm Documentation](https://helm.sh/docs/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)

## 🤝 Contributing

1. Create a feature branch
2. Make your changes
3. Commit with clear messages
4. Push to the repository
5. ArgoCD will automatically sync the changes

## 📄 License

Project maintained by homelab994
