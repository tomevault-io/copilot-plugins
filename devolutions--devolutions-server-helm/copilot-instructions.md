## devolutions-server-helm

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Helm chart for deploying [Devolutions Server (DVLS)](https://devolutions.net/server/) on Kubernetes. The chart lives under `chart/` and produces Kubernetes Deployment, Service, Certificate, and optional networking resources (HTTPRoute, BackendTLSPolicy, DestinationRule).

## Common Commands

```bash
# Lint the chart
helm lint chart/

# Render templates locally (all four required values must be set)
helm template test chart/ \
  --set dvls.hostname=test.example.com \
  --set database.host=db.example.com \
  --set database.name=testdb \
  --set certificate.issuerName=letsencrypt
```

There are no unit tests or automated linting CI yet — validation is done manually with `helm lint` and `helm template`.

## Architecture

### Chart Structure

- `chart/Chart.yaml` — chart metadata; `version` is the chart version, `appVersion` is the DVLS image version
- `chart/values.yaml` — all configurable values with defaults
- `chart/templates/_helpers.tpl` — shared template helpers (naming, labels, image tags, required-value validation)
- `chart/templates/` — core Kubernetes manifests (deployment, service, certificate)
- `chart/templates/hooks/` — pre-upgrade Helm hooks for database migrations
- `chart/environments/` — per-environment value overrides (gitignored)

### Required Values

Four values have no defaults and are enforced with `required` in helpers:
- `dvls.hostname` — external FQDN for the DVLS instance
- `database.host` — SQL Server hostname
- `database.name` — database name
- `certificate.issuerName` — cert-manager ClusterIssuer name (when `certificate.enabled: true`)

### Database Migration Hooks

Pre-upgrade hooks run in weight order to handle migrations safely:
1. **weight -15** `scale-down-rbac.yaml` — creates ServiceAccount/Role/RoleBinding
2. **weight -10** `scale-down-job.yaml` — scales deployment to 0 and waits for rollout
3. **weight -5** `db-migration-job.yaml` — runs DVLS with `DVLS_UPDATE_MODE=true`

Migrations are enabled by default (`migration.enabled: true`) and can be skipped with `--set migration.enabled=false`.

### Database Environment Variable Prefix

The chart supports two env-var prefixes controlled by `database.envPrefix`: `DATABASE` (default) or `AZURE_SQL`. This determines whether database credentials are exposed as `DATABASE_USERNAME`/`DATABASE_PASSWORD` or `AZURE_SQL_USERNAME`/`AZURE_SQL_PASSWORD`.

### Selector Label Override

`selectorLabels` in values.yaml overrides the default selector labels on Deployment, Service, and hooks. This is used for migrating from an existing release that used different labels (e.g., `app: dvls` instead of `app: devolutions-server`).

### TLS Configuration

When `certificate.enabled: true`, the chart creates a cert-manager Certificate resource and mounts the resulting TLS secret into the pod at `/etc/certs/`. When disabled, an existing secret can be referenced via `certificate.secretName`.

### Release Workflow

`.github/workflows/release.yml` is triggered manually (`workflow_dispatch`). It creates a GitHub release tagged from `chart/Chart.yaml` version, packages the chart, and publishes it to the [Devolutions Helm repository](https://devolutions.github.io/helm-charts).

---
> Source: [Devolutions/devolutions-server-helm](https://github.com/Devolutions/devolutions-server-helm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
