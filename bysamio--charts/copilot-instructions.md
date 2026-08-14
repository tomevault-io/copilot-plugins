## charts

> BySamio Helm charts for Kubernetes. Ships hardened, non-root container images via `ghcr.io/bysamio/`.

# AGENTS.md

## Repository Overview

BySamio Helm charts for Kubernetes. Ships hardened, non-root container images via `ghcr.io/bysamio/`.

**Charts:** `wordpress/`, `keycloak/`, `postgresql/`, `mariadb/`, `memcached/`, `minio/`
**Registry:** `oci://ghcr.io/bysamio/charts`
**CI:** `.github/workflows/release-charts.yml` — packages and pushes charts on merge to `main`

## Project Structure

```
<chart>/
  Chart.yaml          # Chart metadata, version, appVersion, dependencies
  values.yaml         # Default values (source of truth for all params)
  README.md           # User-facing docs with param tables and upgrade notes
  templates/
    _helpers.tpl      # All template helpers/functions
    statefulset.yaml  # or deployment.yaml — main workload
    *.yaml            # Services, ingress, secrets, configmaps, etc.
    NOTES.txt         # Post-install message
scripts/              # Pre-commit hook scripts (lint, template, deps, test)
```

## Key Conventions

- **Images:** All charts use `ghcr.io/bysamio/<app>` images. Keycloak has two variants: flexible (default, auto-builds at startup) and optimized (tag contains `-optimized`, pre-built).
- **Security:** Non-root (UID 1001), `readOnlyRootFilesystem` where possible, network policies default on. Keycloak auto-adjusts `readOnlyRootFilesystem` based on image variant.
- **Passwords:** Always use `secretKeyRef` env vars. Never use `_FILE` suffix env vars (that's a Bitnami convention, not supported by upstream images).
- **Helpers:** All reusable logic goes in `_helpers.tpl`. Use `include` not `template`. Helpers handle image resolution, secret names, security contexts, and feature detection.
- **Version bumps:** Patch for fixes, minor for new features/non-breaking behavior changes, major for breaking changes. Update `Chart.yaml` version, add upgrade notes to `README.md` under `## Upgrading`.

## Development Workflow

### Prerequisites
- `helm` (3.8+), `kubectl`, `pre-commit`, a Kubernetes cluster (minikube)

### Pre-commit Hooks
All commits run 10 hooks automatically (configured in `.pre-commit-config.yaml`):
- File checks: trailing whitespace, end-of-file, YAML syntax, large files, merge/case conflicts
- Helm: lint, template render, dependency check, changed-chart tests

Never skip hooks (`--no-verify`). If a hook fails, fix the issue.

### Testing Changes
1. Edit chart files
2. Verify template rendering: `helm template test <chart>/`
3. Deploy to minikube: `helm upgrade --install <name> <chart>/ --set <values> --wait --timeout 5m`
4. Check logs: `kubectl logs <pod> -c <container> --tail=20`
5. Clean up: `helm uninstall <name> && kubectl delete pvc --all`

### Committing
```bash
git add <chart>/ && git commit -m "<type>(<chart>): <description>"
```
Commit types: `fix`, `feat`, `docs`, `refactor`, `chore`

## Chart-Specific Notes

### Keycloak
- `keycloak.isOptimizedImage` helper detects optimized vs flexible by checking for `-optimized` in the image tag
- `keycloak.hostname` helper resolves hostname: `values.hostname` > `ingress.hostname` > empty (Keycloak fails if `hostnameStrict=true` without a hostname — this is intentional)
- `keycloak.containerSecurityContext` forces `readOnlyRootFilesystem=false` for flexible images
- Init container `prepare-write-dirs` copies `/opt/keycloak/lib` to emptyDir for writable overlay (flexible only)
- `keycloakConfigCli` uses `adorsys/keycloak-config-cli` (upstream), not Bitnami's fork

### PostgreSQL
- `auth.username`/`auth.password` create an **additional** user, not a replacement for the `postgres` superuser
- Init script in `create-user-configmap.yaml` runs `CREATE USER` + `GRANT` via psql
- Subchart dependency for Keycloak is pinned in `keycloak/Chart.yaml`

### MinIO
- Supports `mode: standalone` (1 replica, 1 PVC) and `mode: distributed` (4+ replicas, erasure coding)
- `minio.distributedArgs` helper builds the erasure-coding URL pattern `http://<name>-{0...N-1}.<headless>/data{0...D-1}`
- Headless service (`<name>-headless`) provides stable DNS for StatefulSet pods
- `initBuckets` Job creates buckets via `mc`; set `useHelmHooks: false` + custom `annotations` for ArgoCD compatibility
- Image is upstream `quay.io/minio/minio` — will be replaced with `ghcr.io/bysamio/minio` once the hardened image is built

### WordPress
- Depends on `mariadb` and `memcached` subcharts
- PHP config via `php-configmap.yaml`

## Skills

This repo ships agent skills in [`.agents/skills/`](.agents/skills/). Read the relevant `SKILL.md` before and during matching work — each carries the project's standards for its domain. Start with [`using-agent-skills`](.agents/skills/using-agent-skills/SKILL.md) if you are unsure which applies.

### Core — expect to use these on most chart work

| Skill | Use it whenever you are... |
|---|---|
| [`source-driven-development`](.agents/skills/source-driven-development/SKILL.md) | Setting any chart value, env var, or image tag. Chart params mirror upstream contracts (Keycloak `KC_*`, PostgreSQL, MinIO) and the [`bysamio/images`](https://github.com/bysamio/images) READMEs — verify against those sources, never from memory. Cite what you checked. |
| [`documentation-and-adrs`](.agents/skills/documentation-and-adrs/SKILL.md) | Changing anything user-visible. Every value change lands in the chart `README.md` param table, and every version bump gets an entry under `## Upgrading`. Docs are part of the change, not a follow-up. |
| [`git-workflow-and-versioning`](.agents/skills/git-workflow-and-versioning/SKILL.md) | Bumping `Chart.yaml` `version`/`appVersion` or committing. Semver drives what consumers get: patch = fix/image bump, minor = new non-breaking values, major = breaking values. Commits are `<type>(<chart>): <description>`. |
| [`api-and-interface-design`](.agents/skills/api-and-interface-design/SKILL.md) | Adding, renaming, or removing anything in `values.yaml`. Values are this repo's public API — renames and default changes are breaking changes for every consumer. |
| [`security-and-hardening`](.agents/skills/security-and-hardening/SKILL.md) | Touching security contexts, secrets, RBAC, network policies, TLS, or image provenance. This repo's whole reason to exist is hardened defaults. |
| [`code-review-and-quality`](.agents/skills/code-review-and-quality/SKILL.md) | Reviewing any change before it merges, including your own. |

### Situational — use when the task matches

| Skill | Use it whenever you are... |
|---|---|
| [`incremental-implementation`](.agents/skills/incremental-implementation/SKILL.md) | Making a change that spans more than one file or chart. Land thin slices, each rendering cleanly with `helm template`. |
| [`deprecation-and-migration`](.agents/skills/deprecation-and-migration/SKILL.md) | Removing or renaming a value, changing a default, or writing an upgrade path for existing releases. |
| [`ci-cd-and-automation`](.agents/skills/ci-cd-and-automation/SKILL.md) | Touching `.pre-commit-config.yaml`, `scripts/`, `renovate.json`, or `.github/workflows/release-charts.yml`. |
| [`debugging-and-error-recovery`](.agents/skills/debugging-and-error-recovery/SKILL.md) | A template fails to render, a hook fails, or a pod crash-loops on a test cluster. Find the root cause; don't paper over it with a value override. |
| [`observability-and-instrumentation`](.agents/skills/observability-and-instrumentation/SKILL.md) | Working on `metrics`, `ServiceMonitor`, `PrometheusRule`, probes, or anything that determines whether an operator can see the workload. |
| [`planning-and-task-breakdown`](.agents/skills/planning-and-task-breakdown/SKILL.md) | Coordinating a change across several charts (e.g. an image bump that ripples into the `casepack` umbrella). |
| [`code-simplification`](.agents/skills/code-simplification/SKILL.md) | Refactoring `_helpers.tpl` or templates that have grown hard to follow, without changing rendered output. |

### Not applicable here

`frontend-ui-engineering`, `browser-testing-with-devtools`, `test-driven-development`, and `performance-optimization` target application repos. This repo has no UI and no unit-test suite — verification is `helm lint`, `helm template`, and a real deploy to minikube (see Development Workflow above).

Rules:
- Default to using a skill when a task plausibly matches its trigger.
- More than one skill can apply to a task.
- Where a skill is silent, follow this file and the local repo patterns.

---
> Source: [bysamio/charts](https://github.com/bysamio/charts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
