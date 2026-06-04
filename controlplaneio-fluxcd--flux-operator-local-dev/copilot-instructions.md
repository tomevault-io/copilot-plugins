## flux-operator-local-dev

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a local development environment for the [Flux Operator](https://github.com/controlplaneio-fluxcd/flux-operator) using Docker and Kubernetes KIND. It demonstrates GitOps-style continuous delivery using OCI artifacts stored in a local registry instead of Git repositories.

## Common Commands

### Environment Management
- `make up` - Create KIND cluster, local registry, install Flux Operator, and deploy infrastructure and apps
- `make down` - Tear down the entire environment (deletes cluster and registry)
- `make sync` - Push local manifest changes to registry and reconcile with cluster
- `make tools` - Install all required tools via Homebrew (macOS only)

### Flux Operations
- `flux-operator -n flux-system tree ks flux-system` - List all deployed resources
- `flux tree kustomization apps-sync` - View application resources tree
- `flux reconcile kustomization infra-sync --with-source` - Force reconcile infrastructure
- `flux reconcile kustomization apps-sync --with-source` - Force reconcile applications

### Registry
- Local registry runs on `localhost:5050` (external) and `flux-registry:5000` (internal to cluster)
- Manifests are packaged as OCI artifacts and pushed to registry before deployment

## Architecture

### Three-Tier Resource Organization

1. **Cluster Sync Layer** (`kubernetes/clusters/local/`)
   - Entry point for the cluster configuration
   - Defines `ResourceSetInputProvider` for registry configuration
   - Creates two main `ResourceSet` objects: `infra` and `apps`
   - Sets up namespaces and RBAC (platform-team, dev-team service accounts)

2. **Infrastructure Layer** (`kubernetes/infra/`)
   - Platform-level components managed by `platform-team`
   - Installed in order with dependencies: metrics-server → cert-manager → cluster-issuers
   - Uses `dependsOn` with `readyExpr` for proper ordering

3. **Application Layer** (`kubernetes/apps/`)
   - User workloads managed by `dev-team`
   - Demo apps: podinfo (web app with Redis cache backend)
   - Apps depend on infrastructure being ready first

### ResourceSet Pattern

This repo uses the Flux Operator's [ResourceSet](https://raw.githubusercontent.com/controlplaneio-fluxcd/flux-operator/refs/heads/main/docs/api/v1/resourceset.md) and [ResourceSetInputProvider](https://raw.githubusercontent.com/controlplaneio-fluxcd/flux-operator/refs/heads/main/docs/api/v1/resourcesetinputprovider.md) APIs extensively:

**ResourceSetInputProvider**: Provides dynamic inputs to ResourceSets
- `OCIArtifactTag` type - fetches container image tags (e.g., latest Redis Alpine version)
- `Static` type - provides static configuration values (e.g., registry URL)
- Can define schedules (cron) for checking updates

**ResourceSet**: Groups related Kubernetes resources with templating
- Uses Go template syntax with `<< >>` delimiters (e.g., `<< inputs.tag >>`)
- `commonMetadata` - applies labels to all resources
- `dependsOn` - declares dependencies on other resources with readiness checks
- `wait` - waits for all resources to become ready before reconciliation completes
- `serviceAccountName` - specifies which service account reconciles the ResourceSet

### OCI Artifact Workflow

1. Local manifests organized in `kubernetes/` directory
2. `flux push artifact` packages directories as OCI artifacts
3. Artifacts pushed to local registry with `local` tag
4. Flux Operator pulls artifacts and applies them to cluster
5. Changes detected via `flux diff artifact` before pushing

Three OCI artifacts:
- `flux-cluster-sync:local` - from `kubernetes/clusters/local/`
- `flux-infra-sync:local` - from `kubernetes/infra/`
- `flux-apps-sync:local` - from `kubernetes/apps/`

## Security Patterns

All workloads in this repo follow security best practices:

**Pod Security Context:**
- `runAsNonRoot: true`
- `runAsUser: <appropriate-uid>` (999 for Redis, 65534 for podinfo)
- `fsGroup: <appropriate-gid>` (1000 for Redis Alpine, 65534 for podinfo)
- `seccompProfile.type: RuntimeDefault`

**Container Security Context:**
- `allowPrivilegeEscalation: false`
- `readOnlyRootFilesystem: true`
- `runAsNonRoot: true`
- `capabilities.drop: [ALL]`

**Service Accounts:**
- Every deployment has its own ServiceAccount
- ResourceSets specify `serviceAccountName` for RBAC isolation

## Development Workflow

### Making Changes to Manifests

1. Edit files in `kubernetes/apps/` or `kubernetes/infra/`
2. Run `make sync` to push to registry and reconcile
3. Flux automatically detects changes and applies them
4. Use `flux tree kustomization <name>` to verify deployment

### Adding New Applications

Follow the pattern in `kubernetes/apps/podinfo.yaml`:

1. Create `ResourceSetInputProvider` for dynamic inputs (e.g., latest container tag)
2. Create `ResourceSet` with:
   - `inputsFrom` referencing the provider
   - `commonMetadata.labels` for consistent labeling
   - `dependsOn` if app depends on other resources
   - Resources array with ServiceAccount, Deployment, Service, etc.
3. Add security contexts at pod and container level
4. Use templating for dynamic values: `<< inputs.tag >>`, `<< inputs.provider.namespace >>`

### Label Conventions

- Use `app.kubernetes.io/name` for workload labels (not `app`)
- ResourceSets are labeled with `toolkit.fluxcd.io/tenant` (platform-team or dev-team)

## Key Files

- `Makefile` - Primary interface for all operations
- `scripts/kind-up.sh` - Creates KIND cluster with registry
- `scripts/flux-up.sh` - Installs Flux Operator and instance
- `scripts/flux-push.sh` - Packages and pushes manifests as OCI artifacts
- `scripts/flux-sync.sh` - Forces reconciliation of all Kustomizations
- `kubernetes/clusters/local/registry.yaml` - Registry configuration provider
- `kubernetes/clusters/local/infra.yaml` - Infrastructure sync setup
- `kubernetes/clusters/local/apps.yaml` - Applications sync setup

## Testing

The repository includes GitHub Actions workflow (`.github/workflows/test.yaml`) that validates the setup.

---
> Source: [controlplaneio-fluxcd/flux-operator-local-dev](https://github.com/controlplaneio-fluxcd/flux-operator-local-dev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-04 -->
