## openclaw-kubernetes

> This file provides guidance to AI coding agents when working with code in this repository.

# AGENTS.md

This file provides guidance to AI coding agents when working with code in this repository.

## Overview

This is a Helm chart repository for deploying OpenClaw (a personal AI assistant gateway) to Kubernetes. The chart deploys two components: an OpenClaw StatefulSet (single-instance, persistent storage) and an optional LiteLLM proxy Deployment for model provider decoupling. It also includes a Dockerfile for the OpenClaw container image.

## Links

- [OpenClaw](https://openclaw.ai/) (formerly Moltbot/Clawdbot)
- [Source Code](https://github.com/openclaw/openclaw)

## Development Commands

```bash
# Lint the chart against all values files
./scripts/helm-lint.sh

# Render templates to verify correctness (output discarded)
./scripts/helm-test.sh

# Render templates to stdout for inspection
helm template openclaw . -f values.yaml

# Render with a specific values file
helm template openclaw . -f values-production.yaml --set secrets.openclawGatewayToken=test

# Lint with chart-testing tool (used in CI)
ct lint --config ct.yaml

# Package and publish chart to GHCR (requires authentication)
helm registry login ghcr.io -u <username> -p <token>
./scripts/publish-chart.sh
```

All lint/test scripts pass `--set secrets.openclawGatewayToken=lint-token` automatically to satisfy the required secret validation.

## Version Bump

Follow these steps to release a new chart version:

1. **Update chart version** — Bump `version` and `appVersion` in `Chart.yaml` (e.g., `0.1.11` → `0.1.12`).
2. **Update changelog** — Run `git log v<previous>..HEAD --oneline` to list all commits since the last version tag. For any merge commits (e.g., `Merge pull request #N`), also inspect the individual commits they brought in (`git log v<previous>..HEAD --oneline` will include them). Ensure every non-version-bump commit — including those introduced via merged PRs — is reflected in `CHANGELOG.md`. Add a summary following the existing format (date, bullet points describing changes).
3. **Commit** — Stage `Chart.yaml` and `CHANGELOG.md`, commit with message `chore: bump chart version to <new-version>`.
4. **Tag** — Create an annotated tag: `git tag v<new-version>`.
5. **Push** — Ask the user whether to push the main branch and new tag (`git push origin main && git push origin v<new-version>`).

## Bump Dependencies

These dependencies are not covered by Dependabot and must be bumped manually.

### 1. LiteLLM image (`values.yaml`)

```bash
# Check latest stable tags from GHCR
TOKEN=$(curl -s "https://ghcr.io/token?scope=repository:berriai/litellm:pull" | jq -r .token)
curl -s -H "Authorization: Bearer $TOKEN" "https://ghcr.io/v2/berriai/litellm/tags/list" \
  | jq -r '.tags[]' | grep '^main-v.*-stable' | sort -V | tail -5
```

Update `litellm.image.tag` in `values.yaml`.

### 2. openclaw npm package (`Dockerfile`)

```bash
npm view openclaw version
```

Update `ARG OPENCLAW_VERSION=` in `Dockerfile`.

### 3. clawhub npm package (`Dockerfile`)

```bash
npm view clawhub version
```

Update `ARG CLAWHUB_VERSION=` in `Dockerfile`.

### After updating

Run lint and template tests to verify correctness:

```bash
./scripts/helm-lint.sh
./scripts/helm-test.sh
```

## Architecture

### Two-Component Design

The chart deploys two workloads:

1. **OpenClaw StatefulSet** (`templates/statefulset.yaml`) — The gateway itself. Single-instance only (`replicaCount: 1`). Uses persistent storage for the entire `/home/vibe` directory (plugins, configs, tools).
2. **LiteLLM Deployment** (`templates/litellm-deployment.yaml`) — Optional proxy (enabled by default) that decouples OpenClaw from specific AI providers. Supports GitHub Copilot, Anthropic, and OpenAI providers. Runs as a separate Deployment with its own service, config, and secrets.

OpenClaw connects to LiteLLM via its internal service URL, configured automatically in the generated `openclaw.json`.

### Init Container Data Seeding

The `init-home-data` container in the StatefulSet:

1. Seeds `/home/vibe` from image to PVC on first run (if PVC is empty)
2. Seeds `openclaw.json`, `codex-config.toml`, and `claude-settings.json` from ConfigMap

### ConfigMap Generation (`templates/configmap.yaml`)

The ConfigMap renders three configs from values:

- **`openclaw.json`** — Merges base config with LiteLLM provider settings. Auto-detects API format based on model name prefix (`anthropic-messages` for claude*, `openai-responses` for gpt*, `openai-completions` otherwise).
- **`codex-config.toml`** — Codex CLI config pointing at the LiteLLM proxy service.
- **`claude-settings.json`** — Claude Code settings with LiteLLM as base URL, model selections, and permission rules.

Source templates for these configs live in `configs/`.

### Persistence

Three modes, controlled by `persistence.*` values:

- **StatefulSet volumeClaimTemplates** (default, `persistence.useStatefulSetVolumeClaim: true`) — automatic PVC provisioning
- **Existing PVC** (`persistence.existingClaim`) — for pre-provisioned storage
- **emptyDir** (`persistence.enabled: false`) — ephemeral, for development/testing

### Secrets Management

Two modes:

- **Chart-created**: Set values under `secrets.*`. The `openclaw.validateSecrets` helper enforces `openclawGatewayToken` is set.
- **External**: Reference via `secrets.existingSecret` (expects specific key names: `OPENCLAW_GATEWAY_TOKEN`, bot tokens, etc.)

Both secrets and LiteLLM secrets use `lookup` + `helm.sh/resource-policy: keep` to preserve existing values on upgrades.

### Values Presets

- `values.yaml` — Full defaults with security hardening, resource limits, persistence enabled
- `values-development.yaml` — NodePort service, relaxed security, no persistence, debug logging
- `values-production.yaml` — Ingress with TLS, fast-ssd storage, backup annotations, pod anti-affinity
- `values-minimal.yaml` — No security context, no resources, no persistence (CI/testing)

### Template Helpers (`_helpers.tpl`)

Key named templates:

- `openclaw.fullname` / `openclaw.labels` / `openclaw.selectorLabels` — Standard naming and labeling
- `openclaw.secretName` / `openclaw.configMapName` / `openclaw.pvcName` — Resource name resolution (existing or generated)
- `openclaw.validateSecrets` — Fails render if required secrets missing
- `openclaw.validateAutoscaling` — Enforces single-instance constraint
- `openclaw.litellm.fullname` / `openclaw.litellm.configMapName` / `openclaw.litellm.secretName` — LiteLLM resource naming
- `openclaw.ingress.apiVersion` / `openclaw.ingress.supportsPathType` — Kubernetes version compatibility

### Dockerfile

`Dockerfile` builds the OpenClaw container image (`ghcr.io/feiskyer/openclaw-gateway`):

- Base: `node:24-slim` with development tools (git, vim, zsh, ripgrep, gh, chromium)
- User: `vibe` (UID/GID 1024), non-root with sudo
- Installs: `openclaw` (npm), `@openai/codex` (npm), Claude Code, `uv` (Python)
- Copies `configs/codex-config.toml` and `configs/claude-settings.json` into the image
- Entrypoint: `openclaw gateway --allow-unconfigured`

### CI/CD

- **helm-lint-test.yml** — PR and main pushes: runs `ct lint`, `helm-lint.sh`, `helm-test.sh`
- **publish-chart.yml** — Main pushes: publishes chart to `ghcr.io/feiskyer/openclaw-kubernetes` (OCI)
- **docker-build.yml** — Version tags (`v*.*.*`) and main pushes: multi-arch build (amd64/arm64), pushes to GHCR

---
> Source: [feiskyer/openclaw-kubernetes](https://github.com/feiskyer/openclaw-kubernetes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
