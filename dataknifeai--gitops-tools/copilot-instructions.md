## gitops-tools

> This repository contains GitOps configurations for deploying managed Kubernetes tools (Harbor, etc.) to the `nprd-apps` cluster in a dedicated `managed-tools` namespace.

# Cursor Rules for GitOps Tools Repository

## Project Context
This repository contains GitOps configurations for deploying managed Kubernetes tools (Harbor, etc.) to the `nprd-apps` cluster in a dedicated `managed-tools` namespace.

## Code Style and Standards
- Use YAML for all Kubernetes manifests
- Follow Kustomize best practices for organizing base and overlay configurations
- Maintain clear separation between base configurations and cluster-specific overlays
- Use meaningful commit messages following conventional commit format

## Kubernetes Best Practices
- Always specify resource limits and requests for containers
- Use namespaces to isolate resources
- Label all resources appropriately with `app`, `managed-by`, and other relevant labels
- Avoid hardcoding values; use ConfigMaps and Secrets where appropriate
- Document any cluster-specific configurations in overlay directories

## GitOps Principles
- All changes should be made through Git commits
- Manifests should be declarative and idempotent
- Keep base configurations generic and reusable
- Use overlays for environment/cluster-specific customizations

## File Organization
- Base manifests go in `harbor/base/`
- Cluster-specific overlays go in `harbor/overlays/{cluster-name}/`
- Each tool should have its own directory following the same pattern

## Security
- Never commit secrets or sensitive data
- Use Kubernetes Secrets or external secret management
- Review all manifests for security best practices before committing

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DataKnifeAI) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
