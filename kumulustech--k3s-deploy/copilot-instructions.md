## k3s-deploy

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

k3s-deploy is a production-ready, reusable deployment framework for K3s (lightweight Kubernetes) clusters. It provides automated K3s installation, Traefik ingress controller with IngressRoute CRD support, automatic SSL/TLS certificate management via cert-manager and Let's Encrypt, and a comprehensive Helm chart template supporting multiple workload types including Deployments, StatefulSets, and CronJobs.

## Key Commands

### Setup K3s Cluster
```bash
# Required: Set Let's Encrypt email
export LETSENCRYPT_EMAIL="user@example.com"

# Install K3s with Traefik and cert-manager (development)
./scripts/setup-k3s.sh

# Install K3s with Longhorn storage (production)
export INSTALL_LONGHORN=true
./scripts/setup-k3s.sh
```

### Deploy Applications
```bash
# Deploy with default values
./scripts/deploy-app.sh

# Deploy with custom configuration
NAMESPACE=my-app VALUES_FILE=examples/production-app-values.yaml ./scripts/deploy-app.sh

# Deploy with custom release name
RELEASE_NAME=api-service NAMESPACE=backend ./scripts/deploy-app.sh
```

### Fix Registry Issues
```bash
# If Docker registry issues occur
./scripts/fix-registry.sh
```

### Check Deployment Status
```bash
# View all resources in a namespace
kubectl get all -n <namespace>

# Check IngressRoute status
kubectl get ingressroute -n <namespace>
kubectl describe ingressroute <name> -n <namespace>

# Check certificate status
kubectl get certificates -n <namespace>
kubectl describe certificate <name> -n <namespace>

# View Traefik logs
kubectl logs -n traefik deployment/traefik

# Check Longhorn status (if installed)
kubectl get pods -n longhorn-system
kubectl get storageclass
kubectl get volumes.longhorn.io -n longhorn-system
```

### Working with Helm
```bash
# Test Helm chart locally
helm template my-app ./helm-chart -f examples/simple-app-values.yaml

# Debug Helm deployment
helm get values <release-name> -n <namespace>
helm get manifest <release-name> -n <namespace>
```

## Architecture

### Directory Structure
- `scripts/`: Automation scripts for K3s setup and app deployment
- `helm-chart/`: Generic Helm chart with templates for all common Kubernetes resources
- `config/`: cert-manager ClusterIssuer configurations for Let's Encrypt
- `examples/`: Sample values files demonstrating different deployment scenarios

### Key Components

1. **K3s Setup Script** (`scripts/setup-k3s.sh`):
   - Installs prerequisites (Docker, kubectl, helm, etc.) if not present
   - Installs K3s with Traefik disabled (to allow custom configuration)
   - Configures kubectl for user access (copies k3s.yaml to ~/.kube/config)
   - Sets up local Docker registry for development
   - Installs Traefik with IngressRoute CRD support
   - Installs cert-manager and creates Let's Encrypt ClusterIssuers
   - Optionally installs Longhorn distributed storage for production

2. **Helm Chart** (`helm-chart/`):
   - Supports both standard Kubernetes Ingress and Traefik IngressRoute
   - Includes templates for: Deployment, StatefulSet, CronJob, Service, IngressRoute, Certificate, ConfigMap, Secret, HPA, PVC, RBAC (Role, RoleBinding)
   - Configurable via `values.yaml` with sensible defaults
   - Example values files in `examples/` directory

3. **Deployment Flow**:
   - K3s provides the Kubernetes cluster
   - Traefik handles ingress routing (using IngressRoute CRD)
   - cert-manager manages SSL certificates automatically
   - Applications are deployed via the generic Helm chart

### Important Configuration Points

1. **IngressRoute vs Ingress**: The chart supports both. Set `ingress.useIngressRoute: true` for Traefik's native CRD (recommended).

2. **SSL/TLS**: Automatic certificates are enabled by default via cert-manager. Configure with:
   ```yaml
   ingress:
     tls:
       enabled: true
       certManager:
         enabled: true
         clusterIssuer: "letsencrypt-prod"  # or "letsencrypt-staging" for testing
   ```

3. **Environment Variables**: The setup script accepts:
   - `LETSENCRYPT_EMAIL` (required) - validated email format
   - `REGISTRY_PORT` (default: 5000)
   - `CERT_MANAGER_VERSION` (default: v1.13.3)
   - `INSTALL_LONGHORN` (default: false)
   - `LONGHORN_VERSION` (default: v1.5.3)

4. **Persistence**: 
   - Development: Uses K3s's default local-path provisioner
   - Production: Uses Longhorn distributed storage (when installed)
   
   Configure in values.yaml:
   ```yaml
   persistence:
     enabled: true
     storageClass: "local-path"  # or "longhorn" for production
     size: 10Gi
   ```
   
   Note: When Longhorn is installed, it becomes the default storage class automatically.

5. **Workload Types**:
   - **Deployment** (default): For stateless applications
   - **StatefulSet**: Enable with `statefulset.enabled: true` for databases, etc.
   - **CronJob**: Enable with `cronjob.enabled: true` for scheduled tasks

6. **RBAC**: Enable with `rbac.create: true` and define rules for service account permissions

7. **Registry**: The local Docker registry at `localhost:5000` is configured automatically. Use `fix-registry.sh` if issues occur.

## Development Workflow

1. Modify application values in `helm-chart/values.yaml` or create a custom values file (see `examples/` directory)
2. Test the Helm chart locally with `helm template`
3. Deploy using `./scripts/deploy-app.sh` with appropriate environment variables
4. Monitor deployment with `kubectl` commands
5. Check Traefik logs if ingress issues occur
6. Verify certificates with `kubectl get certificates`

## New Features in Latest Update

- **Input Validation**: Scripts now validate required environment variables and file paths
- **RBAC Support**: Added Role and RoleBinding templates for fine-grained permissions
- **StatefulSet Template**: Support for stateful workloads like databases
- **CronJob Template**: Support for scheduled tasks and batch jobs
- **Example Values Files**: Pre-configured examples for common deployment scenarios
- **Registry Fix Script**: Troubleshooting script for Docker registry issues
- **Improved Documentation**: Comprehensive README with examples and best practices

## Notes

- No traditional build/test commands - this is a pure Kubernetes/Helm project
- The setup script automatically installs Docker and other prerequisites
- The local Docker registry (port 5000) is configured automatically for development use
- Always test with Let's Encrypt staging issuer before switching to production
- The Helm chart is designed to be generic and reusable across different applications
- For production workloads with persistent data, use Longhorn storage for replication and data protection
- Longhorn requires open-iscsi and nfs-common packages (installed automatically by the script)
- Scripts include error handling and validation for better reliability

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kumulustech)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kumulustech)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
