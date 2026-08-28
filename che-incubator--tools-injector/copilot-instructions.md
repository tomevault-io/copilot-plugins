## tools-injector

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

Container images + `inject-tool` CLI for dynamically injecting AI CLI tools into Eclipse Che DevWorkspaces. Two components:

1. **Dockerfiles** (`dockerfiles/<tool>/`) — minimal multi-arch images for 7 tools
2. **inject-tool** (`inject-tool/inject-tool.py`) — Python3 CLI that patches DevWorkspace CRs via Kubernetes API using RFC 6902 JSON Patch. Delivered via ConfigMap as a shell shim + `.py` file.

## Build Commands

```bash
make docker-build-local-<tool>   # Quick local build (current platform only)
make docker-build-<tool>         # Multi-arch build (amd64+arm64), no push
make docker-push-<tool>          # Push multi-arch + create manifest
make docker-<tool>               # Build + push shorthand
make docker-build-all            # Build all tools
make help                        # Show all targets
```

Override registry/tag: `IMAGE_REGISTRY=quay.io/myorg TAG=dev make docker-build-local-opencode`

## Architecture

### Two Injection Patterns

**Init pattern** (claude-code, opencode, goose, tmux, python3): Single binary → Alpine builder stage → UBI10 minimal runtime. Binary copied to shared `/injected-tools` volume via preStart init container.

**Bundle pattern** (kilocode, gemini-cli): Node.js runtime + tool → `node:22-slim` builder → UBI10 minimal runtime. Entire bundle at `/opt/<tool>/`, copied to `/injected-tools/<tool>/` via init container. Editor container gets +512Mi memory bump.

Alpine is used for builder stages because UBI10 requires x86-64-v3 which QEMU can't emulate during cross-arch builds.

### inject-tool Internals

The tool registry is `inject-tool/registry.json`:
- `registry`, `tag` — default image registry and tag (overridable via `INJECT_TOOL_REGISTRY`/`INJECT_TOOL_TAG` env vars)
- `infrastructure.patch` — RFC 6902 ops for the shared `injected-tools` volume
- `tools.<name>` — per-tool: `description`, `pattern`, `src`, `binary`, `patch` (append-only init container ops), `editor` (volumeMounts, env, postStart)

`inject-tool.py` loads `registry.json` at startup from the same directory (override path with `INJECT_TOOL_REGISTRY_FILE` env var for testing).

**Patching flow**: validate tools → extract auth token from KUBECONFIG (falls back to service account token) → fetch DevWorkspace CR from Kubernetes API → build inject ops per tool (first with infra, rest skip_infra) → merge into single JSON Patch array → PATCH via API → workspace restarts.

**Multi-tool**: `inject-tool opencode goose tmux` builds patches for each tool, merges into one API call, one restart. `--hot` stays single-tool only.

**Hot-inject mode** (`inject-tool <tool> --hot`): uses `oc image extract` to pull binary directly without restart. Init-pattern tools only. Not persistent across restarts.

### Deployment

`inject-tool/setup.sh <namespace>` creates a ConfigMap with DWO automount labels so every workspace in that namespace gets `/usr/local/bin/inject-tool`.

## CI/CD

- `.github/workflows/pr.yml` — builds all 7 tools multi-arch on every PR, tags with `pr-<number>`
- `.github/workflows/release.yml` — pushes `next` tag on merge to main

Both push to `quay.io/che-incubator/tools-injector/<tool>`.

## Adding a New Tool

1. Create `dockerfiles/<tool>/Dockerfile` following init or bundle pattern
2. Add tool name to `TOOLS` list in `Makefile`
3. Add registry entry in `inject-tool/registry.json` (`tools.<name>` with `description`, `pattern`, `src`, `binary`, `patch`, `editor`)
4. PR triggers CI multi-arch build; merge pushes `:next`

## No Automated Tests

CI validates via successful Docker builds only. inject-tool logic is tested manually in DevWorkspaces.

---
> Source: [che-incubator/tools-injector](https://github.com/che-incubator/tools-injector) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
